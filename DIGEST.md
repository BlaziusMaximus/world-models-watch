# World Models Watch — Digest

---

## 2026-08-28

### What landed this week

**Wayve launches GAIA-4 (August 3, 2026)**
[Wayve](https://wayve.ai/thinking/gaia-4/)
GAIA-4 puts the Wayve AI Driver *inside* the simulation loop: it reconstructs a recorded drive as video and — new for this version — radar, then lets the AI Driver's decisions change what it perceives next. Change the decision, get a genuinely different counterfactual sensor stream (a cyclist encounter that plays out differently), not just a re-rendered clip. Radar matters because it's the sensor that holds up in fog and low light where cameras degrade — folding it into generation is a step toward a simulator whose outputs are trustworthy enough to score real safety metrics on, not just look plausible. This is the renderer/simulator line from Li's taxonomy made operational: closed-loop means the model's dynamics, not just its pixels, are what's under test.

**NVIDIA open-sources OmniDreams (July 2026)**
[arXiv:2606.03159](https://arxiv.org/abs/2606.03159) | [HuggingFace](https://huggingface.co/papers/2606.03159)
A Cosmos-derived, action-conditioned world model that generates photorealistic driving video in real time, autoregressively conditioned on past frames, current simulator state, and the driving action taken — then closes the loop with NVIDIA's Alpamayo policy model. The headline result: a policy post-trained purely inside OmniDreams' simulated loop beats the VLA-based Alpamayo 1.5 baseline using 1/5 the parameters. Landing within a month of GAIA-4, this is a second major lab independently converging on "the world model's job is to be a reactive loop-partner for a policy," not a video generator judged on FID.

**CrashTwin: a physics-grounded benchmark that catches world models cheating (July 2026)**
[arXiv:2606.28757](https://arxiv.org/abs/2606.28757) | [Project page](https://crash-twin.github.io/)
37K collision sequences (25K synthetic + 12K real, calibration-free reconstruction) built specifically to stress-test whether generated dynamics obey momentum and kinetic-energy conservation — not just look temporally smooth. The finding that matters: high perceptual quality routinely *masks* severe physics violations during multi-agent interactions like collisions. This is the empirical flip side of Li's simulator "contract" covered last digest — a concrete instrument for measuring the gap between plausible video and actual physical simulation, which is exactly the gap CONTEXT.md is tracking.

**VideoWeave: geometric consistency without a geometry crutch (June 12, 2026)**
[arXiv:2606.14162](https://arxiv.org/abs/2606.14162)
Video diffusion models drift out of 3D consistency under viewpoint change because most fixes bolt on explicit depth maps or point clouds as hard constraints — and inherit those upstream reconstruction errors. VideoWeave instead adapts geometry-model features into "geometry latents" and jointly denoises them alongside video latents in a shared space, so geometry shapes the generative distribution implicitly rather than as a rigid condition. Best-in-class on both perceptual video metrics (VBench) and 3D consistency metrics (epipolar error, reconstruction PSNR/SSIM) simultaneously — evidence that implicit and explicit scene representation aren't an either/or, exactly the question CONTEXT.md flags under "what's the right representation."

*Also on the radar but outside this window's freshness cut, worth a look if the neuroscience thread pulls you: ["Structure Abstraction and Generalization in a Hippocampal-Entorhinal Inspired World Model"](https://arxiv.org/abs/2605.15733) (arXiv, submitted May 26 2026) explicitly splits a world model into an HPC-like content pathway and an MEC-like structure pathway built on a continuous attractor network — architecture, not metaphor.*

---

### The LEARNING.md connection

CrashTwin's physics diagnostic checks whether generated collisions conserve momentum and kinetic energy — and that check runs straight through the same math you're studying this week. For a rigid body, the inertia tensor **I** is a symmetric 3×3 matrix, and rotational kinetic energy is ½ωᵀIω. To evaluate that correctly under rotation, you diagonalize: **I = R Λ Rᵀ**, where R (orthogonal) gives the body's principal axes and Λ (diagonal) gives the principal moments of inertia — an eigendecomposition of a symmetric matrix, full stop. It's the identical factorization shape as Gaussian splatting's Σ = RSSᵀRᵀ from two digests ago, just applied to mass distribution instead of a scene primitive's spatial extent. Different physical quantity, same reason for the decomposition: find the basis in which a symmetric matrix's action is just independent stretching along orthogonal directions, so the physically meaningful numbers (moments of inertia, or Gaussian scales) fall out as eigenvalues instead of being buried in an arbitrary coordinate frame. When a benchmark like CrashTwin flags a world model for violating energy conservation, under the hood it's very likely comparing eigenvalues of an inertia-like tensor before and after — the exact operation you're building intuition for.

---

### Threads worth pulling

1. **Two closed-loop driving world models in one month.** GAIA-4 and OmniDreams both shipped policy-in-the-loop simulation within weeks of each other, from different labs, converging on the same idea from different sensor/architecture choices (radar fusion vs. Cosmos-diffusion-plus-real-time-autoregression). Worth a session comparing what "closed loop" actually buys you over open-loop video generation, and whether either genuinely clears Li's simulator bar or just moves the plausibility problem into action-space.

2. **Physics benchmarks are starting to catch what visual metrics can't.** CrashTwin's headline finding — perceptually great videos that silently break conservation laws — suggests FID/VBench-style scores may be actively misleading as a proxy for simulator quality. If this benchmark category grows, it could reshape how "state of the art" gets claimed in this space; worth tracking whether Cosmos 3, GAIA-4, or OmniDreams get run through it.

---

### Papers added to covered list

- Wayve, "GAIA-4: Multimodal World Models Powering Closed-Loop Simulation for Safe and Scalable Autonomy," August 3 2026
- NVIDIA, "OmniDreams: Real-Time Generative World Model for Closed-Loop Autonomous Vehicle Simulation," arXiv:2606.03159, July 2026
- "CrashTwin: A Physics-Grounded Benchmark for Multi-Agent Dynamics in World Models," arXiv:2606.28757, July 2026
- "VideoWeave: Unlocking Geometric Consistency in Video Generation via Joint Geometry-Video Modeling," arXiv:2606.14162, June 12 2026

---

## 2026-06-11

*Inaugural entry. CONTEXT.md and LEARNING.md bootstrapped from session instructions — fill them in or edit to match your actual study state.*

---

### What landed this week

**Fei-Fei Li publishes "A Functional Taxonomy of World Models" (June 3, 2026)**
[World Labs blog](https://www.worldlabs.ai/blog/taxonomy-of-world-models) | [Substack](https://drfeifei.substack.com/p/a-functional-taxonomy-of-world-models)
Li draws a hard line between three things people conflate under "world model": **renderer** (makes pixels), **simulator** (holds geometry + physics + dynamics), and **planner** (picks actions). She anchors the frame in the POMDP loop — the simulator's job is to maintain the belief state about the world, the planner reads from it, the renderer projects it to observation. The key move: the simulator has a *contract* — geometric consistency under novel viewpoints, dynamics that obey physics, no free lunch of just predicting plausible frames. This is exactly the framing that distinguishes meaningful spatial intelligence from video generation. World Labs' Marble product targets the simulator role specifically.

**NVIDIA launches Cosmos 3 (June 1, 2026)**
[NVIDIA newsroom](https://nvidianews.nvidia.com/news/nvidia-launches-cosmos-3-the-open-frontier-foundation-model-for-physical-ai) | [Technical report](https://research.nvidia.com/labs/cosmos-lab/cosmos3/technical-report.pdf)
Omnimodal world foundation model: one mixture-of-transformers architecture that handles text, image, video, ambient audio, and action tokens together. Trained on 20 trillion tokens including 400M videos. First open model claiming native physics accuracy across all modalities. The "omnimodal" bet is interesting — rather than video prediction plus language plus action heads bolted together, it treats all modalities as tokens in a shared latent. Reduces synthetic data generation cycles for robotics and AV from months to days. Open-weights under the NVIDIA Open Model License.

**Waymo World Model ships on DeepMind Genie 3 (February 2026)**
[Waymo blog](https://waymo.com/blog/2026/02/the-waymo-world-model-a-new-frontier-for-autonomous-driving-simulation/) | [Coverage](https://www.marktechpost.com/2026/02/06/waymo-introduces-the-waymo-world-model-a-new-frontier-simulator-model-for-autonomous-driving-and-built-on-top-of-genie-3/)
Waymo x DeepMind collaboration using Genie 3 as the backbone — generates photorealistic, interactive driving environments at 720p and ~20–24 fps. Control via language prompts, action inputs, and scene layouts. The key use case: synthesizing scenarios that don't exist in any real driving log (tornadoes, planes landing on highways) so AV policies can see and learn from edge cases before encountering them. This is the simulator framing made concrete — counterfactual generation from a physics-grounded state space, not just video style transfer.

**Rolls (2025): A Theory and Model of Scene Representations with Hippocampal Spatial View Cells**
[Hippocampus journal](https://onlinelibrary.wiley.com/doi/full/10.1002/hipo.70013)
Edmund Rolls (Oxford/Warwick) proposes a three-layer network model of how the primate ventromedial visual cortex builds whole-scene representations. Layer 1–2: visual feature detectors for fixated patches (V1→retrosplenial). Layer 3: a continuous attractor network in the parahippocampal area / hippocampus stitches fixation patches into a full allocentric scene representation — each neuron is a spatial view cell. Gaze direction provides gain modulation to anchor each patch to the correct part of the scene. Relevant here because: if you want to understand what a biological world model's scene representation looks like, this is a mechanistic account — and the continuous attractor structure is what gives it stability and viewpoint invariance.

**PNAS: Unified Neural Representation for Spatial and Conceptual Computations**
[PNAS](https://www.pnas.org/doi/10.1073/pnas.2413449122)
A computational model trained on navigation and semantic association tasks produces representations that resemble place cells, grid cells, *and* concept cells — spontaneously, in the same architecture. The implication: spatial and semantic representations may not be distinct systems but the same computation applied to different input geometries. For the AI side, this is a hint that a world model that gets spatial structure right might get semantic structure cheaply as a byproduct.

---

### The LEARNING.md connection

You're working on eigendecomposition — here's where it is in the wild.

**Gaussian splatting** (now reaching production-grade tooling in 2026, with glTF KHR_gaussian_splatting hitting release-candidate status) represents every scene primitive as a 3D Gaussian. The shape of each Gaussian is stored not as a raw 3×3 covariance matrix Σ — because gradient descent would immediately break the constraint that Σ must be positive semi-definite — but as Σ = R S S^T R^T, where R is an orthogonal rotation matrix and S is a diagonal scale matrix. This is eigendecomposition: you're factoring a symmetric PSD matrix into its eigenvectors (the columns of R) and eigenvalues (the diagonal of S). The split into an unconstrained rotation and an unconstrained positive scale lets gradient descent roam freely without a projection step. When the densification algorithm decides whether to *split* a Gaussian (make it two smaller ones) vs. *clone* it (add a copy nearby), it checks the magnitude of the largest eigenvalue of Σ — if the Gaussian is too elongated, it splits. That decision is eigenvalue inspection applied at render time. Every time you compute an eigenvalue of a 3×3 symmetric matrix, you're doing what the Gaussian splatting trainer is doing.

---

### Threads worth pulling

1. **Li's taxonomy vs. current systems**: The simulator role she defines — geometry + physics + dynamics as a formal contract — is still mostly aspirational. Worth a deeper session on what current systems (Cosmos 3, Waymo World Model, Marble) actually satisfy of that contract and where they fall short. The gap between "plausible video" and "simulator" is the whole question.

2. **Continuous attractors as scene memory**: Rolls' spatial view cell model uses a continuous attractor network to hold allocentric scene representations stable across saccades. Continuous attractors are also proposed as the mechanism behind grid cells and head-direction cells. This is a thread that connects neuroscience of space to the architecture question — what kind of recurrent structure would let an artificial world model hold 3D scene state stable across time and viewpoints?

---

### Papers added to covered list

- Fei-Fei Li, "A Functional Taxonomy of World Models," World Labs blog, June 3 2026
- NVIDIA, "Cosmos 3: Omnimodal World Models for Physical AI," technical report, June 1 2026
- Waymo, "The Waymo World Model: A New Frontier for Autonomous Driving Simulation," February 2026
- Rolls ET, "A Theory and Model of Scene Representations With Hippocampal Spatial View Cells," *Hippocampus* 2025
- "A Unified Neural Representation Model for Spatial and Conceptual Computations," *PNAS* 2025
