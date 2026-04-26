---
title: "Papers"
permalink: /publications/
layout: single
author_profile: true
---

{% comment %}
## Papers I'm Building Toward

### Target Submission

**Residual Neural Network for Sim-to-Real Adaptation in Quadruped Locomotion**  
*Target: CoRL 2026*  
> Using a lightweight residual network to correct for actuator nonlinearities and friction discrepancies identified via CMA-ES system identification, enabling zero-shot deployment of RL-trained policies to real hardware.

---

## Key References

Papers that directly shaped my research direction:

### Locomotion & Sim-to-Real

**PACE: Human and Camera Motion Estimation from In-the-Wild Videos**  
Kocabas et al., 2024  
*Used CMA-ES output to re-identify actuator parameters on our hardware.*

**Learning to Walk in Minutes Using Massively Parallel Deep Reinforcement Learning**  
Rudin et al., RSS 2022  
[PDF](https://arxiv.org/abs/2109.11978)  
*Foundation for our GPU-parallel PPO training pipeline.*

**Learning Robust Perceptive Locomotion for Quadrupedal Robots in the Wild**  
Miki et al., Science Robotics 2022  
*Asymmetric Actor-Critic with privileged information.*

**RMA: Rapid Motor Adaptation for Legged Robots**  
Kumar et al., RSS 2021  
[PDF](https://arxiv.org/abs/2107.04034)  
*Online adaptation via encoder distillation — related to our residual network approach.*

### Natural Motion

**DeepMimic: Example-Guided Deep Reinforcement Learning of Physics-Based Character Skills**  
Peng et al., SIGGRAPH 2018  
[Project Page](https://xbpeng.github.io/projects/DeepMimic/index.html)  
*Reference state initialization and early termination for motion imitation.*

**AMP: Adversarial Motion Priors for Stylized Physics-Based Character Control**  
Peng et al., SIGGRAPH 2021  
[Project Page](https://xbpeng.github.io/projects/AMP/index.html)  
*Discriminator-based style reward — core method in our natural motion work.*

**ASE: Large-Scale Reusable Adversarial Skill Embeddings for Physically Simulated Characters**  
Peng et al., SIGGRAPH 2022  
*Latent space for multi-skill locomotion — next step after AMP.*

**LCP: Lipschitz-Constrained Policies for Robust Robot Control**  
*Lipschitz continuity guarantees for policy stability.*

### RL Theory

**Proximal Policy Optimization Algorithms**  
Schulman et al., 2017  
[PDF](https://arxiv.org/abs/1707.06347)  
*Base algorithm.*

**Soft Actor-Critic: Off-Policy Maximum Entropy Deep Reinforcement Learning**  
Haarnoja et al., ICML 2018  
[PDF](https://arxiv.org/abs/1801.01290)  
*Off-policy baseline for comparison (FlashSAC variant).*

### Simulation

**MuJoCo: A Physics Engine for Model-Based Control**  
Todorov et al., IROS 2012  
*Core simulator.*

**GPU-Accelerated Robotic Simulation for Distributed Reinforcement Learning**  
Liang et al., 2018  
*Background on GPU parallelism in simulation.*

---

## Personal Papers (Previous Work)

**TechTalk: Dimension Reduction Methods — PCA to Autoencoders**  
Shin, J., 2023  
[PDF](/paper/Dimension/TechTalkCFP_dimension_reduction.pdf)  
*Survey and analysis of dimensionality reduction from linear (PCA) to nonlinear (VAE) methods.*
{% endcomment %}
