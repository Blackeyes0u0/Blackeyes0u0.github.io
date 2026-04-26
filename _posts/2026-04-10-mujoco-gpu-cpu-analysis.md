---
title: "MuJoCo CPU vs GPU: A Deep Dive into Simulation Differences"
date: 2026-04-10
categories:
  - Simulation
  - Engineering
tags:
  - MuJoCo
  - GPU
  - Physics_Simulation
  - Warp
  - Sim_to_Real
author_profile: true
toc: true
toc_label: "Contents"
---

{% comment %}
We use MuJoCo in two very different modes: CPU for prototyping and deployment testing, GPU (via Warp) for large-scale parallel training. They're *mostly* equivalent. The "mostly" is where things get interesting.

## The Equations of Motion (Both Modes)

Both CPU and GPU MuJoCo solve the same equation of motion:

$$
\mathbf{M}(\mathbf{q})\ddot{\mathbf{q}} = \boldsymbol{\tau}_{\text{torque}} - \mathbf{C}(\mathbf{q},\dot{\mathbf{q}})\dot{\mathbf{q}} + \mathbf{J}^T \mathbf{f}_{\text{contact}}
$$

where:
- $$\mathbf{M}(\mathbf{q})$$: mass/inertia matrix (positive definite by construction)
- $$\mathbf{C}(\mathbf{q},\dot{\mathbf{q}})$$: Coriolis and centripetal forces (via RNE)
- $$\mathbf{J}^T \mathbf{f}_{\text{contact}}$$: contact forces projected back into joint space

The solution pipeline (common to both):

```
1. Compute unconstrained acceleration (RNE):
   a_smooth = M^{-1} * (tau - C*v - G)

2. Contact detection → identify active contact pairs

3. Solve contact NCP (LCP in practice):
   a_final = a_smooth + M^{-1} * J^T * F_contact
   s.t. friction cone, normal force ≥ 0

4. Integrate: q_{t+1}, v_{t+1} = integrate(q_t, v_t, a_final, dt)
```

The divergences happen at steps 2, 3, and 4.

## Step 1: Mass Matrix Computation (O(n³) vs CRB)

The **Composite Rigid Body (CRB)** algorithm computes $$\mathbf{M}(\mathbf{q})$$ in $$O(n^2)$$ where $$n$$ is the number of DOF. For a 12-DOF quadruped this is fast, but the structure matters for GPU parallelism.

MuJoCo uses CRB + **Recursive Newton-Euler (RNE)** for the Coriolis term:

$$
\mathbf{C}(\mathbf{q},\dot{\mathbf{q}})\dot{\mathbf{q}} = \text{RNE}(\mathbf{q}, \dot{\mathbf{q}}, \ddot{\mathbf{q}}=0)
$$

**GPU issue**: RNE involves recursive computations over the kinematic chain. Each joint's velocity-dependent term depends on its parent. This sequential dependency limits GPU parallelism within a single simulation. The GPU gains efficiency by parallelizing *across* environments, not within one.

## Step 2: Contact Detection — The Big Difference

### CPU behavior

On CPU, contact pairs are discovered *dynamically* at each timestep by broad-phase collision detection (bounding volume hierarchies). The set of active contacts changes frame to frame.

### GPU behavior (Warp)

On GPU, contact pairs must be **pre-specified** in the XML. The simulator doesn't do dynamic broad-phase — it only checks the pairs you listed. This is a deliberate design choice for GPU efficiency (no dynamic memory allocation during simulation).

**Why this matters for real-world deployment:**

If the robot steps on an edge or an object not represented in the pre-specified contact pairs, the GPU simulator *silently ignores* the contact. The policy never experienced this contact during training, so it may not handle it correctly.

```
# XML example: explicit contact pairs
<contact>
  <pair geom1="foot_fl" geom2="ground"/>
  <pair geom1="foot_fr" geom2="ground"/>
  <!-- You must list every contact you care about -->
</contact>
```

The CPU sim would discover contacts dynamically — including unexpected ones like leg-to-leg collisions during extreme gaits.

## Step 3: The Integrator Difference

### CPU: Multiple choices

| Integrator | Accuracy | Speed | Notes |
|------------|----------|-------|-------|
| `euler` | Low | Fast | First-order, can be unstable at large timesteps |
| `rk4` | High | Slow | 4× function evaluations, but stable |
| `implicit` | Medium-high | Medium | Symplectic, better energy conservation |
| `implicitfast` | Medium | Fast | Simplified implicit, GPU-compatible |

### GPU: Only `implicitfast`

The GPU sim is locked to `implicitfast`. This integrator uses a simplified Jacobian approximation to reduce the per-step cost:

$$
\mathbf{v}_{t+1} = \mathbf{v}_t + \Delta t \cdot \mathbf{M}^{-1}\left(\boldsymbol{\tau} - \partial_\mathbf{v}[\mathbf{C}\mathbf{v}] \cdot \mathbf{v}_t\right)
$$

The approximation drops some higher-order velocity terms. For locomotion at 50 Hz this is usually fine, but we observed discrepancies in high-velocity trajectories.

**Practical test**: after training on GPU, always run the same policy on CPU with `rk4` integrator. If behavior diverges significantly, the policy may be exploiting the integrator approximation.

## The Contact Solver: NCP vs LCP

The contact dynamics satisfy:

$$
\begin{aligned}
\mathbf{f}_n &\geq 0 \quad \text{(normal force non-negative)} \\
\phi(\mathbf{q}) &\geq 0 \quad \text{(no penetration)} \\
\mathbf{f}_n \cdot \phi(\mathbf{q}) &= 0 \quad \text{(complementarity)}
\end{aligned}
$$

Plus the friction cone constraint:
$$\|\mathbf{f}_t\| \leq \mu \mathbf{f}_n$$

This is a **Nonlinear Complementarity Problem (NCP)**. MuJoCo approximates it as a convex problem using a soft constraint formulation.

**Why no static friction by default?** Implementing static friction correctly requires checking the "stick" condition before allowing "slip," which requires solving a more complex system. MuJoCo uses a single unified friction model that approximates both. This is why:

1. Robots in MuJoCo tend to slide slightly more than in reality
2. Locomotion policies sometimes exhibit unexpected foot slippage when deployed

We add a custom static friction term in our real-world contact model analysis.

## The Armature / Actuator Dynamics Question

The actuator model in MuJoCo:

$$
\boldsymbol{\tau}_{\text{effective}} = k_p (q_{\text{target}} - q) + k_d (\dot{q}_{\text{target}} - \dot{q}) - \text{armature} \cdot \ddot{q}
$$

The **armature** parameter models rotor inertia. Its effect: it adds inertia at each joint that resists rapid accelerations. Too high → artificially damps oscillations in sim. Too low → unstable in sim.

Our initial armature values (0.2–0.6) were 5–10× higher than what system identification (PACE/CMA-ES) found (0.04–0.06). The simulator was silently compensating for what would have been instability on real hardware.

**Consequence**: the policy learned to command rapid position changes because the high armature would smooth them out. On real hardware (low armature), these rapid commands caused ~20 Hz oscillations — the infamous action jitter.

## Practical Summary

| Difference | Impact on Sim-to-Real | Fix |
|------------|----------------------|-----|
| Contact detection (pre-specified vs dynamic) | Missing contacts on unusual terrain | Test on CPU, add more contact pairs |
| `implicitfast` integrator | High-velocity trajectory divergence | Validate with CPU `rk4` |
| Static friction not fully modeled | Foot slippage | Tune friction params from real data |
| Armature estimation | Action jitter | Run system identification (PACE) |

The GPU sim is fast enough that we can train policies that generalize well despite these differences — but only if we understand *which* differences matter and test for them explicitly.

## Code Pattern: Validate on CPU Before Deployment

```python
# After GPU training, always validate with CPU sim
cpu_env = make_env(backend="cpu", integrator="rk4")
gpu_env = make_env(backend="gpu", integrator="implicitfast")

# Run same policy on both and compare
rollout_cpu = evaluate(policy, cpu_env, n_episodes=50)
rollout_gpu = evaluate(policy, gpu_env, n_episodes=50)

# If reward gap > threshold, investigate
if abs(rollout_cpu.mean_reward - rollout_gpu.mean_reward) > THRESHOLD:
    print("WARNING: CPU/GPU divergence detected. Check contact pairs and integrator sensitivity.")
```

The next post covers the sim-to-real gap from the dynamics perspective — how Newton-Euler equations help us identify *why* the real robot behaves differently.
{% endcomment %}
