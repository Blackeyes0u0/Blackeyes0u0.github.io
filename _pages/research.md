---
title: "Research"
permalink: /research/
layout: single
author_profile: true
toc: true
toc_label: "Contents"
toc_icon: "robot"
---

{% comment %}
## Overview

My research sits at the intersection of **reinforcement learning**, **robot dynamics**, and **real-world deployment**. The core question is:

> *How do we train a robot in simulation so that it actually works in the real world — and moves like it was born to move?*

---

## Demo — Robustness Tests

> No-mesh simulation (DreamWaQ v5). Robot geometry replaced with collision primitives.

### Push Robustness

Lateral impulse applied for 0.1 s at $t = 4$ s into steady walking.
Policy must recover without falling.

| Terrain | Push $\Delta v = 2.0$ m/s · 815 N | Push $\Delta v = 3.0$ m/s · 1223 N |
|---------|----------------------------------|-------------------------------------|
| **Flat** | ![](/assets/images/robustness/push2_flat_dreamwaq_side.gif) | ![](/assets/images/robustness/push3_flat_dreamwaq_side.gif) |
| **Pyramid Stairs** | ![](/assets/images/robustness/push2_pyramid_stairs_dreamwaq_side.gif) | ![](/assets/images/robustness/push3_pyramid_stairs_dreamwaq_side.gif) |
| **Random Stairs** | ![](/assets/images/robustness/push2_random_stairs_dreamwaq_side.gif) | ![](/assets/images/robustness/push3_random_stairs_dreamwaq_side.gif) |

### Low Friction · $\mu = 0.2$

Ground sliding friction reduced to 0.2 (default 1.0 — 5× more slippery).

<div style="display:flex; gap:12px; flex-wrap:wrap; margin:12px 0 24px;">
  <div style="flex:1; text-align:center;">
    <img src="/assets/images/robustness/friction_flat_dreamwaq_side.gif" style="width:100%;"/>
    <small>Flat</small>
  </div>
  <div style="flex:1; text-align:center;">
    <img src="/assets/images/robustness/friction_pyramid_stairs_dreamwaq_side.gif" style="width:100%;"/>
    <small>Pyramid Stairs</small>
  </div>
  <div style="flex:1; text-align:center;">
    <img src="/assets/images/robustness/friction_random_stairs_dreamwaq_side.gif" style="width:100%;"/>
    <small>Random Stairs</small>
  </div>
</div>

---

## Quadruped RL Locomotion

### Setup

A 12-DOF quadruped robot (4 legs × 3 joints) trained with **PPO** to track velocity commands $(v_x, v_y, \omega_z)$ on flat terrain.

| Component | Detail |
|-----------|--------|
| Simulator | MuJoCo (CPU & GPU via Warp) |
| Algorithm | PPO (asymmetric Actor-Critic, RSL-RL / Brax-JAX) |
| Environments | up to 4096 parallel environments |
| Control rate | 50 Hz (250 Hz sim, 5× decimation) |
| Observation | 69-dim: gyro, gravity, joint angles/velocities, action history, command |
| Action | 12-dim joint position offsets |

### Policy Architecture

**Asymmetric Actor-Critic** — the critic has access to privileged simulation state (ground contact forces, true mass distribution, friction coefficients) that the actor never sees. This dramatically improves sample efficiency without compromising zero-shot sim-to-real transfer.

$$
\pi_\theta(a \mid o) \quad \text{(actor: observation only)}
$$

$$
V_\phi(s_{\text{priv}}) \quad \text{(critic: privileged state)}
$$

### Reward Function

$$
r = r_{\text{vel}} + r_{\text{ang\_vel}} - r_{\text{action\_rate}} - r_{\text{contact}} - r_{\text{torque}} + r_{\text{alive}}
$$

- **Velocity tracking**: $r_{\text{vel}} = \exp\!\left(-\|v_{\text{cmd}} - v_{\text{actual}}\|^2 / \sigma\right)$
- **Action smoothness**: penalizes high-frequency joint commands
- **Gait timing**: rewards alternating foot contacts consistent with a trot gait

### Domain Randomization

| Parameter | Range |
|-----------|-------|
| Base mass offset | ±20% |
| Center of mass offset | ±5 cm |
| Inertia scaling | 0.8 × – 1.2 × |
| PD gains (Kp, Kd) | ±10% |
| Torque saturation | 70% – 100% |
| Ground friction | 0.3 – 1.5 |

### Deployment

Trained policy exported to **C++ inference** for real-time deployment. The controller runs at 50 Hz on embedded hardware, reading joint encoders and IMU, outputting target joint positions to motor drivers.

**Key challenge**: *action jitter* caused by overestimated armature values. PACE / CMA-ES system identification revealed the true armature range was 0.04–0.06, while we were using 0.2–0.6 — leading to high-frequency oscillations.

---

## Sim-to-Real Adaptation

### Problem Statement

Even with domain randomization, there is a fundamental mismatch between simulated and real dynamics. The Newton-Euler equation in simulation assumes:

$$
\mathbf{M}(\mathbf{q})\ddot{\mathbf{q}} + \mathbf{C}(\mathbf{q}, \dot{\mathbf{q}})\dot{\mathbf{q}} + \mathbf{G}(\mathbf{q}) = \boldsymbol{\tau}_{\text{cmd}} - \boldsymbol{\tau}_{\text{friction}}
$$

But in reality $\boldsymbol{\tau}_{\text{true}} = \beta \cdot \boldsymbol{\tau}_{\text{cmd}}$ where $\beta \neq 1$ due to motor driver nonlinearities, backlash, and temperature-dependent resistance.

### Residual Neural Network

Rather than re-training the base policy, we train a residual network $f_\phi$ that learns the correction term:

$$
\boldsymbol{\tau}_{\text{residual}} = f_\phi(\mathbf{o}_t,\, \mathbf{a}_t,\, \mathbf{a}_{t-1},\, \Delta\mathbf{q}_t)
$$

$$
\boldsymbol{\tau}_{\text{deploy}} = \pi_\theta(\mathbf{o}_t) + f_\phi(\cdot)
$$

### System Identification (PACE / CMA-ES)

We use **CMA-ES** to identify physical parameters by minimizing Newton-Euler prediction error on real rollout data:

$$
\hat{\boldsymbol{\theta}} = \arg\min_{\boldsymbol{\theta}} \sum_t \left\| \ddot{\mathbf{q}}^{\text{real}}_t - \hat{\ddot{\mathbf{q}}}_t(\boldsymbol{\theta}) \right\|^2
$$

| Parameter | Sim value | Identified | Effect |
|-----------|-----------|-----------|--------|
| Armature (hip) | 0.30 | 0.05 | action jitter |
| Armature (knee) | 0.40 | 0.06 | action jitter |
| Hip friction | 0 Nm | 0.12 Nm | direction-change lag |
| Base mass offset | 0 kg | +0.3 kg | tracking bias |

### MuJoCo CPU vs GPU Differences

| Aspect | CPU | GPU (Warp) |
|--------|-----|------------|
| Integrator | RK4 / Euler / implicit | `implicitfast` only |
| Contact detection | Dynamic (BVH) | Pre-specified pairs only |
| Constraint solver | Full NCP | Simplified |

The contact detection difference means GPU-trained policies can fail if real-world contact pairs differ from the pre-specified set — a subtle but critical sim-to-real source.

---

## Natural Motion

### Motivation

Standard RL produces *functional* but *mechanical* gaits. We want the robot to move like a dog or wolf — smooth weight transfer, natural swing phase, dynamic balance through the whole body.

### Adversarial Motion Priors (AMP)

Based on [Peng et al., 2021](https://xbpeng.github.io/projects/AMP/index.html), we train a discriminator $D_\psi$ alongside the policy:

$$
r_{\text{style}}(s_t, s_{t+1}) = \max\!\left(0,\; 1 - \tfrac{1}{4}\bigl(D_\psi(s_t, s_{t+1}) - 1\bigr)^2\right)
$$

$$
r_{\text{total}} = w_{\text{task}} \cdot r_{\text{task}} + w_{\text{style}} \cdot r_{\text{style}}
$$

The discriminator is trained with the LSGAN objective to distinguish policy-generated transitions from reference animal motion clips.

### Current Challenge

Balancing $w_{\text{task}}$ and $w_{\text{style}}$ — when task reward dominates, style is ignored; when style dominates, velocity tracking fails. We are testing a curriculum:

$$
w_{\text{style}}(t) = w_{\text{style}}^{\max} \cdot \min\!\left(1,\; \frac{t}{T_{\text{warmup}}}\right)
$$

### Deep Mimic

Following [Peng et al., 2018](https://xbpeng.github.io/projects/DeepMimic/index.html), we also use direct reference tracking reward:

$$
r_{\text{mimic}} = \exp\!\left(-w_q \|\mathbf{q}_t - \hat{\mathbf{q}}_t\|^2 - w_v \|\dot{\mathbf{q}}_t - \hat{\dot{\mathbf{q}}}_t\|^2\right)
$$

Reference State Initialization (RSI) and Early Termination handle phase alignment.

---

## Perception Pipeline (Side Project)

### Autonomous Navigation
- ViNT (Vision-based Navigation Transformer) for goal-conditioned navigation
- BEV-MPPI planner for real-time path optimization
- Depth + RGB + semantic camera fusion (MetaUrban sim)

### Person Tracking Gimbal
- YOLO11s detection + ByteTrack multi-object tracking
- PID controller for gimbal pan/tilt/zoom
- RTSP streaming, MQTT control interface

The long-term goal: the robot *sees* where it can step and plans safe footfall locations in sparse terrain.

---

## Future Work

| Direction | Description |
|-----------|-------------|
| Sparse foothold navigation | Perception + locomotion for mountain-goat-like terrain crossing |
| Humanoid extension | Apply RL + AMP pipeline to bipedal humanoid robots |
| Embodied reasoning | Integrate with VLA models (e.g., Gemini Robotics ER) |
| Online adaptation | Real-time residual network updates during deployment |
{% endcomment %}
