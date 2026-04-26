---
title: "PPO for Quadruped Locomotion: Mathematics, Implementation, and Failure Modes"
date: 2026-04-15
categories:
  - RL
  - Locomotion
tags:
  - PPO
  - Quadruped
  - Reinforcement_Learning
  - MuJoCo
  - Actor_Critic
author_profile: true
toc: true
toc_label: "Contents"
---

{% comment %}
Proximal Policy Optimization (PPO) is the workhorse of legged robot locomotion. But applying it naively to a 12-DOF quadruped reveals failure modes that aren't obvious from the theory. This post covers the math, our asymmetric Actor-Critic setup, and the bugs we chased for weeks.

## Why PPO for Locomotion?

The alternatives are:

- **SAC (off-policy)**: Better sample efficiency on paper, but the replay buffer contains transitions from different policy versions — creating an implicit distribution shift that's hard to correct. In high-dimensional continuous action spaces like quadruped control (12 joints), this becomes significant. More on this below.
- **TD3**: Deterministic policy, which makes exploration in joint space difficult.
- **PPO (on-policy)**: Less sample-efficient, but the on-policy constraint makes training stable under function approximation errors. With GPU-parallelized simulation (4096 envs), sample efficiency matters less than wall-clock time.

## PPO: The Core Equations

The clipped surrogate objective:

$$
L^{\text{CLIP}}(\theta) = \mathbb{E}_t \left[ \min\left( r_t(\theta) \hat{A}_t,\ \text{clip}(r_t(\theta),\ 1-\epsilon,\ 1+\epsilon) \hat{A}_t \right) \right]
$$

where the probability ratio is:

$$
r_t(\theta) = \frac{\pi_\theta(a_t \mid s_t)}{\pi_{\theta_{\text{old}}}(a_t \mid s_t)}
$$

The clip prevents the new policy from moving too far from the old one — a trust region enforced through the objective rather than a KL constraint.

**Advantage estimation** via Generalized Advantage Estimation (GAE):

$$
\hat{A}_t = \sum_{l=0}^{\infty} (\gamma \lambda)^l \delta_{t+l}
\quad \text{where} \quad
\delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)
$$

The $$\lambda$$ parameter trades off bias ($$\lambda \to 0$$, TD(0)) and variance ($$\lambda \to 1$$, Monte Carlo). For locomotion, $$\lambda = 0.95, \gamma = 0.99$$ works well in practice.

## Asymmetric Actor-Critic

Standard Actor-Critic uses the same state for both actor and critic. In simulation, we can do better:

$$
\pi_\theta(a_t \mid o_t) \quad \text{— actor sees sensor observations only}
$$
$$
V_\phi(s_t^{\text{priv}}) \quad \text{— critic sees privileged simulation state}
$$

The **privileged state** $$s_t^{\text{priv}}$$ includes:
- Ground contact forces (not measurable on real robot without force sensors)
- True mass distribution and center-of-mass position
- Ground friction coefficients
- Actuator temperature / torque saturation state

Why does this help? The value function needs to accurately predict long-term returns. If it only has the same noisy sensor data as the actor, it has high variance. Giving it ground truth simulation data dramatically reduces critic variance, which reduces the variance of the advantage estimate, which stabilizes actor updates.

**At deployment**, only the actor runs — the critic is discarded. The policy never sees privileged data, so there's no sim-to-real gap from this asymmetry.

## Observation Space Design

Our 69-dimensional observation:

| Component | Dim | Notes |
|-----------|-----|-------|
| Gyro (angular velocity) | 3 | IMU measurement |
| Gravity projection | 3 | $$R^T \mathbf{g}$$, encodes orientation |
| Joint positions (error) | 12 | $$q - q_{\text{default}}$$ |
| Joint velocities | 12 | |
| Joint error history | 36 | 3 previous timesteps × 12 joints |
| Last action | 12 | Previous command |
| Velocity command | 3 | $$(v_x, v_y, \omega_z)$$ |

The **history** (36 dims) is crucial — it allows the policy to infer velocity and acceleration implicitly, handling the partial observability of the environment without an explicit RNN.

**Why no explicit RNN?** RNNs can represent longer history but are slower to train and harder to deploy. History-in-observation is a common engineering tradeoff in locomotion.

## Action Space and Squashed Gaussian

The policy outputs parameters of a **squashed Gaussian**:

$$
\mu_\theta(o_t),\ \sigma_\theta(o_t) \in \mathbb{R}^{12}
$$

The actual action is:

$$
a_t = \tanh\left(\frac{\mu + \epsilon \sigma}{\text{action\_scale}}\right) \cdot \text{action\_scale}
$$

The `tanh` squashing bounds actions to $$[-1, 1]$$, preventing extreme joint commands that could damage hardware.

**Issue we ran into**: when `action_scale` is too small (e.g., 0.4), the policy lives in the linear region of `tanh` and the gradient with respect to $$\sigma$$ becomes tiny — exploration collapses. With `action_scale = 0.4`, the effective gradient of `tanh` at the boundary is close to 0.

The fix: keep `action_scale` ≥ 1.0, or normalize the output separately from the squashing.

## The Off-Policy Question: Why Not SAC?

SAC optimizes:

$$
\pi^* = \arg\max_\pi \mathbb{E}_{\tau \sim \pi} \left[ \sum_t r_t + \alpha H(\pi(\cdot \mid s_t)) \right]
$$

The Q-function uses bootstrapped TD(0):

$$
Q(s, a) = \mathbb{E}_{s'} \left[ R(s, a, s') + \mathbb{E}_{a' \sim \pi}[Q(s', a')] \right]
$$

Two problems for locomotion:

**Problem 1: TD(0) bias in long-horizon tasks.** Locomotion episodes are ~1000 steps. TD(0) propagates reward signal one step at a time, requiring thousands of updates to correctly attribute a gait-level reward (e.g., "step frequency is wrong") to individual joint actions. GAE with $$\lambda = 0.95$$ propagates credit ~20 steps per update, which is far more efficient.

**Problem 2: Off-policy distribution shift.** The Q-function uses:

$$
\mathbb{E}_{a' \sim \pi_\text{current}}[Q(s', a')]
$$

But in the buffer, transitions were collected under $$\pi_\text{old}$$. The mismatch between $$\pi_\text{current}$$ and $$\pi_\text{old}$$ grows as training progresses. Importance sampling corrects this in theory, but in practice the variance of importance weights is prohibitive in 12-dimensional action spaces.

For locomotion with GPU-parallelized simulation (cheap samples, many envs), **PPO wins on wall-clock time** because on-policy data is inherently in-distribution.

## Failure Modes We Actually Hit

### 1. Feet Not Lifting Off the Ground

Symptom: robot shuffles instead of lifting feet.  
Cause: the contact penalty dominates early training, so the policy learns "never lift, always drag."  
Fix: weight the contact penalty near zero for the first 10M steps, then gradually increase. Curriculum on reward weights matters as much as on terrain difficulty.

### 2. Action Jitter on Real Hardware

Symptom: high-frequency oscillations in joint commands (~20 Hz).  
Cause: overestimated armature value in simulation. We used armature = 0.2–0.6, but PACE/CMA-ES identified the true value as 0.04–0.06.  
Effect: in simulation, artificially high armature damps oscillations. On real hardware, the policy "expects" this damping and the absence causes instability.  
Fix: re-run system identification before training; set armature from measured data.

### 3. Policy Collapses After DR Expansion

Symptom: training was working, we added more domain randomization (wider DR ranges), and performance dropped to near-zero.  
Cause: the value function suddenly faces much higher variance in returns (because different DR samples produce very different reward profiles). The advantage estimates become noisy.  
Fix: widen DR ranges gradually over training. Also increase batch size when widening DR to average out the variance.

## GPU vs CPU Training: Practical Notes

We train on GPU (MuJoCo Warp) for throughput and test policies on CPU (standard MuJoCo) before deployment.

One subtle difference: the GPU simulator uses `implicitfast` integrator which handles contact differently from `euler` or `rk4`. We observed that policies trained exclusively on GPU could behave differently on CPU for the same scenario — specifically in contact-rich situations (feet colliding with rough terrain edges).

**Rule of thumb**: always validate GPU-trained policies on the CPU simulator and on hardware before trusting the GPU sim metrics.

## Next Steps

The current policy handles flat terrain well. The challenge now is:

1. **Unstructured terrain**: rocks, stairs, gaps — requires foothold selection
2. **Sparse foothold navigation**: how do you plan safe steps in terrain like a dry riverbed?
3. **Natural motion**: the current gait is functional but not elegant — AMP or Deep Mimic to inject animal-like motion patterns

The math of natural motion via adversarial training deserves its own post.
{% endcomment %}
