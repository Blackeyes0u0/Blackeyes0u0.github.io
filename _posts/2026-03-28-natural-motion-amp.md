---
title: "Natural Motion via Adversarial Motion Priors: Making Robots Move Like Animals"
date: 2026-03-28
categories:
  - Natural_Motion
  - RL
tags:
  - AMP
  - Deep_Mimic
  - Natural_Motion
  - Adversarial_Training
  - Locomotion
author_profile: true
toc: true
toc_label: "Contents"
---

{% comment %}
Standard RL produces *functional* locomotion — the robot reaches the target speed, doesn't fall, minimizes energy. But watch a dog run. There's something fundamentally different about how it moves: weight transfer through the whole body, anticipatory balance, expressive foot placement. This post explores how we're trying to inject that quality into learned robot locomotion.

## The Problem with "Functional" Gait

A PPO-trained quadruped with a standard reward:

$$
r = r_{\text{vel}} - \lambda_1 r_{\text{torque}} - \lambda_2 r_{\text{action\_rate}}
$$

will learn to track velocity while minimizing energy. The gait that emerges is mechanically optimal under these constraints — but it looks stiff and unnatural. Common artifacts:

- All four legs move in lockstep ("marching" rather than "trotting")
- No weight transfer — the center of mass stays fixed relative to the body
- Foot strike is abrupt, not padded through the leg chain
- Direction changes involve stopping and pivoting rather than flowing turns

These aren't *wrong* per the reward function, but they look nothing like animal locomotion. Why does this matter?

1. **Real-world robustness**: natural animal gaits are biomechanically efficient for handling perturbations. A dog's stride adapts implicitly to uneven ground; a mechanically-optimal stride doesn't.
2. **Usability**: robots that move like animals are more predictable to humans, easier to deploy in human environments.
3. **Foundation for harder tasks**: the flowing, whole-body coordination of natural motion is likely necessary for tasks like jumping between sparse footholds.

## Adversarial Motion Priors (AMP)

Peng et al. (SIGGRAPH 2021) introduced AMP: train a discriminator to distinguish policy-generated trajectories from reference animal motion clips.

### Formulation

Standard RL reward + style reward:

$$
r_{\text{total}} = w_{\text{task}} \cdot r_{\text{task}} + w_{\text{style}} \cdot r_{\text{style}}
$$

The style reward is the discriminator's confidence that the transition $$(s_t, s_{t+1})$$ came from the reference distribution:

$$
r_{\text{style}}(s_t, s_{t+1}) = \max\left(0,\, 1 - \frac{1}{4}\left(D_\psi(s_t, s_{t+1}) - 1\right)^2\right)
$$

This is the negative of the WGAN discriminator loss, bounded to $$[0, 1]$$.

The discriminator $$D_\psi$$ is trained alongside the policy with the GAN objective:

$$
\min_{D_\psi} \mathbb{E}_{(s,s') \sim \mathcal{D}_{\text{motion}}}\left[\left(D_\psi(s,s') - 1\right)^2\right] + \mathbb{E}_{(s,s') \sim \pi_\theta}\left[D_\psi(s,s')^2\right]
$$

The policy tries to fool the discriminator (make $$D_\psi \to 1$$ for its generated transitions), while the discriminator tries to distinguish real motion from policy-generated motion.

### What the Discriminator Sees

The state representation for the discriminator captures features relevant to motion quality:

- Joint positions and velocities (relative to default pose)
- Foot contact states
- Body orientation and angular velocity
- Relative foot positions in the body frame

Notably, the discriminator does *not* see position or heading — only local, body-relative features. This makes the style reward invariant to where the robot is in the world.

### Reference Motion Extraction

We extract motion clips from:
- Dog motion capture datasets (trot, gallop, sit-to-stand)
- Manually annotated video (cats, wolves)
- Retargeted from quadruped motion databases

The retargeting converts motion capture bone transforms to the robot's specific joint configuration — not trivial because animal and robot kinematic structures differ.

## Deep Mimic: Direct Reference Tracking

A simpler but powerful alternative: directly reward the policy for matching a reference trajectory.

$$
r_{\text{mimic}} = \exp\left(-w_q \|\mathbf{q}_t - \hat{\mathbf{q}}_t\|^2 - w_v \|\dot{\mathbf{q}}_t - \hat{\dot{\mathbf{q}}}_t\|^2\right)
$$

where $$\hat{\mathbf{q}}_t$$ is the reference joint angle at time $$t$$.

**Advantage**: provides a very clear learning signal — the policy knows exactly what it should be doing at each timestep.

**Disadvantage**: requires phase matching — the robot must be at the *same point in the gait cycle* as the reference at the same time. If it falls behind by half a step, the reference is maximally wrong (opposite foot should be down) and the reward collapses.

**Reference State Initialization (RSI)**: to handle this, episodes are initialized at random phases of the reference trajectory. The policy learns to continue the motion from any phase, avoiding the phase-matching problem.

**Early Termination (ET)**: if the robot deviates too far from the reference trajectory, the episode terminates early. This prevents the policy from exploring states where the reference provides no useful signal.

## ASE: Multi-Style via Latent Embeddings

Once we have a single-style AMP policy working, the natural extension is multi-style: can one policy produce dog trot, wolf gallop, and cat slink based on a latent command?

ASE (Adversarial Skill Embeddings) trains a latent-conditioned policy:

$$
\pi_\theta(a_t \mid s_t, z) \quad z \in \mathbb{R}^{64}
$$

with a contrastive discriminator that maps each reference clip to a distinct region of the latent space. At deployment, you can interpolate between styles or switch based on terrain context.

This is the direction I want to pursue — a unified locomotion controller that can be *instructed* about gait style while still tracking velocity commands.

## The Balancing Problem

The hardest part in practice: balancing $$w_{\text{task}}$$ and $$w_{\text{style}}$$.

**Too much task weight** ($$w_{\text{task}} \gg w_{\text{style}}$$):
- Style reward is marginal; policy ignores it
- Converges to the same stiff functional gait as plain PPO
- Discriminator trains to distinguish, but policy doesn't bother responding

**Too much style weight** ($$w_{\text{style}} \gg w_{\text{task}}$$):
- Policy learns to perfectly imitate the reference motion
- But the reference motion may not cover all velocity commands
- Robot fails to track commanded velocities
- Discriminator loss collapses to 0 (perfect imitation), gradient vanishes

**The practical fix we're testing**: curriculum on weights. Start with high $$w_{\text{task}}$$ to establish velocity tracking, then gradually increase $$w_{\text{style}}$$.

$$
w_{\text{style}}(t) = w_{\text{style}}^{\max} \cdot \min\left(1, \frac{t}{T_{\text{warmup}}}\right)
$$

Additionally, the discriminator's training can lag behind the policy — if the discriminator updates too slowly, the style gradient is noisy; too fast and it overfits to the early policy's limited distribution.

## Why This Connects to Robustness

There's a deeper connection between natural motion and robustness that I believe is underexplored.

Animal gaits are *biomechanically stable*: they were evolved under the constraint that the animal survives perturbations. A dog's trot is not just aesthetically natural — it's a locally stable limit cycle in the dynamical system defined by the animal's body and gravity.

When we use AMP to imitate natural motion, we're implicitly optimizing for a limit cycle that is stable under perturbations. This *may* be why AMP-trained policies tend to be more robust to external forces than pure RL policies — not because of any explicit robustness reward, but because the reference motion encodes stability through its own trajectory.

This hypothesis connects to work on **Lipschitz-Constrained Policies (LCP)** — if the policy's output is Lipschitz-constrained with respect to the state, small perturbations in state produce small perturbations in action. Natural motion trajectories may implicitly satisfy Lipschitz-like properties because evolution selected for exactly this.

## Where We're Going

The end goal is a robot that can traverse terrain like a mountain goat — jumping between sparse footholds, landing with controlled impact absorption, using the whole body to maintain balance. That requires:

1. **Natural base motion** (AMP + multi-style ASE)
2. **Whole-body coordination** (not just legs — head, spine, tail dynamics)
3. **Foothold selection** from perception (where do I step next?)
4. **Landing control** (predict impact, pre-activate muscles)

Each of these is an open problem. But the first step — making the robot move naturally on flat ground — is the foundation everything else builds on.

This work is ongoing. The discriminator is training. The videos are not pretty yet.
{% endcomment %}
