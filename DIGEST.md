# World Models Watch — Digest

---

## 2026-07-17

### What landed since last time

**WorldRoamBench: An Open-World Benchmark for Long-Horizon Stability of Interactive World Models** (AMAP CV Lab, Alibaba Group, June 2026)
[arXiv](https://arxiv.org/abs/2606.31672)
This is the direct follow-up to last digest's thread about whether current systems actually satisfy Li's simulator contract. WorldRoamBench scores 10+ open and closed interactive world models (Genie-class systems included) across four axes evaluated *simultaneously*: per-frame **action** following, **vision** drift over long WASD-style exploration, **physics** plausibility gated by controllability, and **memory** consistency decoupled from the action stream. Headline result: no model is good at all four at once — strong visual quality doesn't buy you physical plausibility, and neither buys you memory. This is the first real evidence, not manifesto language, of exactly where the "plausible video" vs. "simulator" gap actually lives.

**PH-Dreamer: A Physics-Driven World Model via Port-Hamiltonian Generative Dynamics** (Tongji University / Shanghai Research Institute for Intelligent Autonomous Systems, May 2026)
[arXiv](https://arxiv.org/abs/2605.18303)
Most world models (Dreamer-style recurrent state-space models, Genie's emergent consistency) get physical plausibility as a byproduct of scale and data. PH-Dreamer bets the other way: it bakes a **Port-Hamiltonian structure** directly into the latent transition — every rollout is literally parameterized as energy flowing through a system with explicit conservation and dissipation terms, so violating Newton's laws isn't just discouraged by training data, it's structurally harder to represent. Reduces imagined-vs-real reward mismatch and produces smoother, lower-energy control. A genuinely different answer to "how does a simulator earn its physics" than the scale-and-hope approach.

**A simple model of co-emergence of grid and place fields** (Wang, Morris, Derdikman, Chaudhari, Balasubramanian; May 2026)
[arXiv](https://arxiv.org/abs/2605.21356)
Direct successor to the Rolls spatial-view-cell work from last digest. A single recurrent network — no separate grid-cell or place-cell modules, no supervision of either — trained on one objective (predict the next sensory observation from masked past observations plus egocentric motion) spontaneously grows both grid and place codes, with their balance set by how much sensory noise/masking is applied during training. Without retraining, it reproduces real experimental phenomena: grid fragmentation in hairpin mazes, grid merging after wall removal, lattice alignment across connected rooms, and the developmental order (place cells before grid cells) seen in real animals. Strong evidence that spatial codes are what falls out of *any* system forced to predict its next sensory state from noisy, partial input under motion — which is uncomfortably close to what a video-based world model is also being asked to do.

**NVIDIA launches Cosmos 3 Edge** (July 16, 2026)
[SiliconANGLE](https://siliconangle.com/2026/07/16/nvidia-launches-cosmos-3-edge-model-expands-physical-ai-push-japan/) | [CNBC](https://www.cnbc.com/2026/07/16/nvidia-reveals-new-ai-model-and-expands-japans-physical-ai-ecosystem.html)
A 4B-parameter, Nemotron-based compact version of the Cosmos 3 model covered last digest, built to run vision reasoning and generate robot control policies directly on Jetson/RTX edge hardware, with per-robot adaptation in about a day. Paired with a 20+ company "Cosmos Coalition" expansion into Japanese industrial robotics (FANUC, Yaskawa, Kawasaki, Honda R&D, Toyota-backed Preferred Networks). Less a research advance than a distribution move — it's the point where world-model-as-simulator starts shipping as the control substrate for physical robots at industrial scale, not just a training-data generator.

---

### The LEARNING.md connection

You're about to move from linear algebra into optimization, with Lagrange multipliers and constrained optimization on deck — PH-Dreamer above is a live example of exactly that machinery doing real work. Its "energy-guided actor-critic" doesn't just reward the policy for good returns; it also wants rollouts to stay near a physically valid, low-energy manifold (a *constraint*), and it enforces that constraint the textbook way: fold it into the objective as a Lagrange multiplier term, so the optimizer is pushed toward solutions that are simultaneously high-reward and low-energy, without ever needing a separate projection step back onto the constraint surface. It's the exact same move eigendecomposition made for Gaussian splatting's Σ = RSS^Rᵀ (last digest's connection) — reparameterize so the optimizer can move freely instead of fighting a hard constraint. When you get to Lagrange multipliers in a few weeks, PH-Dreamer is worth revisiting: it's a case where "convert constraint into penalty term" is the entire architectural idea, applied to make a world model's imagined physics actually obey conservation laws.

---

### Threads worth pulling

1. **Which axis does each real system actually fail?** WorldRoamBench doesn't just say "no model passes everything" — it breaks failure down by axis. Worth digging into whether Genie 3 specifically fails on Memory (consistent with its ~1-minute visual memory horizon reported elsewhere) while passing Vision, versus where Cosmos 3 or GAIA-3 land. That would turn last digest's "the gap between plausible video and simulator is the whole question" into a concrete, per-system diagnosis instead of a vibe.

2. **Structural physics vs. emergent physics, as a bet.** PH-Dreamer (bake in conservation laws via Port-Hamiltonian structure) and Genie 3 (physics as an emergent property of scale, no explicit 3D representation at all) are two opposite wagers on how a simulator earns its physics contract. One is falsifiable and interpretable but narrower in what it can represent; the other scales with data but is a black box about *why* it works when it works. Worth a session on which bet is more likely to generalize to genuinely novel physical scenarios neither was trained on.

---

### Papers added to covered list

- "WorldRoamBench: An Open-World Benchmark for Long-Horizon Stability of Interactive World Models," AMAP CV Lab / Alibaba Group, arXiv, June 2026
- "PH-Dreamer: A Physics-Driven World Model via Port-Hamiltonian Generative Dynamics," Tongji University / SRIIAS, arXiv, May 2026
- Wang, Morris, Derdikman, Chaudhari, Balasubramanian, "A Simple Model of Co-Emergence of Grid and Place Fields," arXiv, May 2026
- NVIDIA, "Cosmos 3 Edge," July 16 2026

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
