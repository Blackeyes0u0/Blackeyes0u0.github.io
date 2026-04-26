---
title: "Sim-to-Real Gap: From Newton-Euler Equations to Residual Networks"
date: 2026-04-05
categories:
  - Sim_to_Real
  - Locomotion
tags:
  - Sim_to_Real
  - Residual_Network
  - System_Identification
  - Newton_Euler
  - PACE
  - CMA_ES
author_profile: true
toc: true
toc_label: "Contents"
---

{% comment %}
The robot walks beautifully in simulation. Then you deploy it. The legs shake. One joint develops a consistent torque bias. In certain gaits, the foot slips. What's going wrong, and how do we fix it systematically?

## The Fundamental Assumption

Reinforcement learning for locomotion assumes the simulator correctly models the transition probability:

$$
p(s_{t+1} \mid s_t, a_t)
$$

This probability is defined by the physical equations of motion. If our simulator is wrong about the physics, the policy learns to exploit simulator artifacts that don't exist in reality.

The Newton-Euler equation for a rigid body robot:

$$
\mathbf{M}(\mathbf{q})\ddot{\mathbf{q}} + \mathbf{C}(\mathbf{q},\dot{\mathbf{q}})\dot{\mathbf{q}} + \mathbf{G}(\mathbf{q}) = \boldsymbol{\tau}_{\text{motor}} - \boldsymbol{\tau}_{\text{friction}} + \mathbf{J}^T \mathbf{f}_{\text{contact}}
$$

If this equation held *exactly* with perfect parameters, sim-to-real gap would be zero. The gap comes from:

1. **Incorrect $$\mathbf{M}(\mathbf{q})$$**: wrong mass, inertia, center of mass
2. **Incorrect $$\boldsymbol{\tau}_{\text{motor}}$$**: motor model doesn't match actual actuator response
3. **Incorrect $$\boldsymbol{\tau}_{\text{friction}}$$**: static friction, velocity-dependent friction not modeled
4. **Missing dynamics**: cable stretch, leg flex, thermal effects

## Decomposing the Actuator Model

The commanded torque $$\boldsymbol{\tau}_{\text{cmd}}$$ and the actual delivered torque $$\boldsymbol{\tau}_{\text{true}}$$ differ:

$$
\boldsymbol{\tau}_{\text{true}} = \beta \cdot \boldsymbol{\tau}_{\text{cmd}}
$$

where $$\beta$$ is a gain factor that depends on:
- Motor driver efficiency (never exactly 1.0)
- Back-EMF at high speeds
- Current limiting in the driver
- Temperature state of the motor

In the ideal case $$\beta = 1$$, the acceleration in free space (no contact) would be:

$$
\ddot{\mathbf{q}} = \mathbf{M}^{-1}\left(\boldsymbol{\tau}_{\text{cmd}} - \mathbf{C}\dot{\mathbf{q}} - \mathbf{G}\right)
$$

With $$\beta \neq 1$$, the same commanded torque produces different acceleration. Over a 1-second trajectory, this error accumulates.

**Key insight**: In free flight (no contact), the dynamics are determined entirely by motor torque and rigid-body mechanics. This is where we can isolate the motor model error most cleanly — no contact model uncertainty.

## System Identification via PACE / CMA-ES

System identification means: given real robot data, find parameters that minimize the prediction error of our physics model.

For the i-th trajectory:

$$
\hat{\boldsymbol{\theta}}^* = \arg\min_{\boldsymbol{\theta}} \sum_{t=0}^{T} \left\| \ddot{\mathbf{q}}^{\text{real}}_t - \hat{\ddot{\mathbf{q}}}_t(\boldsymbol{\theta}) \right\|^2
$$

where $$\boldsymbol{\theta}$$ includes:
- Armature values (per joint): $$a_i$$
- Friction coefficients: $$\mu_i, b_i$$ (Coulomb + viscous)
- Mass offsets: $$\Delta m$$
- CoM offsets: $$\Delta r_{\text{CoM}}$$

**Why CMA-ES?** The objective is non-convex (multiple local minima) and the parameter space is ~30 dimensional. Gradient-based methods get stuck. CMA-ES (Covariance Matrix Adaptation Evolution Strategy) is a derivative-free optimizer that works well in this setting.

### What We Found

Running PACE on our robot:

| Parameter | Simulator Value | Identified Value | Effect |
|-----------|----------------|-----------------|--------|
| Armature (hip) | 0.30 | 0.05 | 6× overestimate → action jitter |
| Armature (knee) | 0.40 | 0.06 | 7× overestimate |
| Hip friction | 0.0 (none) | 0.12 Nm | Undershoot on direction changes |
| Knee friction | 0.0 (none) | 0.08 Nm | Velocity-dependent lag |
| Base mass offset | 0.0 kg | +0.3 kg | Policy expects lighter robot |

The armature values were the primary driver of action jitter. When the simulator models high armature, it behaves as if the joints have high rotational inertia — damping rapid oscillations. On real hardware, this damping doesn't exist at the same magnitude, and the aggressive position commands generate 20 Hz ringing.

## Domain Randomization: The Standard Fix

The classic approach is to train the policy under a *distribution* of parameters rather than fixed values:

$$
\boldsymbol{\theta} \sim p(\boldsymbol{\theta}) = \prod_i \mathcal{U}(\theta_i^{\min}, \theta_i^{\max})
$$

The policy must perform well under all parameter combinations, so it learns conservative strategies that work even in the worst case.

**Limitation**: if the real robot parameters fall outside the randomization range, the policy degrades. More importantly, DR doesn't help when the *model structure* is wrong — for example, if the simulator has no static friction but the real robot has significant static friction, no amount of DR over the parameters we've included will fix this.

## Residual Neural Network Approach

Our approach: train a **residual network** to learn the *correction* between simulated and real dynamics.

The base policy $$\pi_\theta$$ is fixed (trained in simulation with DR). We train $$f_\phi$$ to predict an additive correction:

$$
a_{\text{deploy}} = \pi_\theta(o_t) + f_\phi(o_t, a_t^{\text{sim}}, \Delta q_t, \Delta \dot{q}_t)
$$

The correction network observes:
- Current observation $$o_t$$
- The base policy's nominal action $$a_t^{\text{sim}} = \pi_\theta(o_t)$$
- Joint position tracking error $$\Delta q_t = q_t - q_t^{\text{des}}$$
- Joint velocity tracking error $$\Delta \dot{q}_t$$

The residual is trained to minimize real-world tracking error:

$$
\mathcal{L}_\phi = \sum_t \left\| \Delta q_{t+1}^{\text{real}} \right\|^2 + \lambda \left\| f_\phi(o_t, \cdot) \right\|^2
$$

The regularization term $$\lambda \|f_\phi\|^2$$ prevents the residual from dominating the base policy — it should *correct* the sim policy, not replace it.

### Why Residual, Not Adapter?

An alternative is to train an **adapter** that maps real observations to a latent context variable (RMA-style). The residual approach has two advantages:

1. **Interpretability**: the correction signal $$f_\phi$$ tells us *where* the dynamics differ, which helps debugging
2. **Safety**: if $$f_\phi \to 0$$, we fall back to the sim policy — guaranteed to work in sim-like conditions

The adapter approach is more powerful for large distribution shifts, but requires more data and has no fallback.

## The Structure of $$\mathbf{M}^{-1}$$ and Why It Matters

One subtlety: the correction $$f_\phi$$ operates in *joint space* (torque/position corrections). But dynamics errors in mass distribution affect the robot in *Cartesian space*. The coupling between these spaces is $$\mathbf{M}^{-1}$$:

$$
\Delta \ddot{q} = \mathbf{M}^{-1} \Delta \boldsymbol{\tau}
$$

For a well-conditioned $$\mathbf{M}$$, small torque corrections can have large effects on specific joints. The residual network must implicitly learn this mapping.

Since $$\mathbf{M}(\mathbf{q})$$ is symmetric positive definite, we can factor it as $$\mathbf{L}^T \mathbf{D} \mathbf{L}$$ (Cholesky) which is how MuJoCo computes $$\mathbf{M}^{-1}$$ efficiently. The condition number of $$\mathbf{M}$$ varies significantly with configuration — the residual network sees harder optimization near configuration singularities.

## Results and Open Questions

After training the residual network on 2 hours of real-robot data:

- Action jitter: substantially reduced
- Straight-line walking: velocity tracking error reduced by ~40%
- Direction changes: still some lag, likely due to static friction not being fully captured

**Open question 1**: The residual network is trained offline on a fixed dataset. When the robot state distribution shifts (different terrain, added mass), the residual may be out-of-distribution. How do we do *online* residual learning safely?

**Open question 2**: Can we use the residual signal to improve the base policy? If $$f_\phi$$ consistently predicts the same correction, this suggests the base policy is systematically wrong — and we should fix the simulator and retrain.

The interaction between sim-to-real methods and RL training is an open research area. The goal is ultimately to make the sim-to-real gap a *solved* engineering problem — something we handle systematically with identified parameters and learned corrections — rather than an art.
{% endcomment %}
