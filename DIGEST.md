# World Models Watch — Digest

---

## 2026-09-11

*First check-in since June — three months of movement. Leading with the one that matters most for the simulator framing.*

---

### What landed since June

**World Labs ships Atlas (September 1, 2026)**
[World Labs blog](https://www.worldlabs.ai/blog/atlas) | [SiliconANGLE coverage](https://siliconangle.com/2026/09/01/fei-fei-lis-world-labs-debuts-atlas-a-world-model-showcase-for-advanced-spatial-intelligence/) | [Deep dive](https://kingy.ai/blog/world-labs-atlas-world-model-deep-dive/)
This is Marble's big brother, and it's the first system that tries to *be* the simulator role Li defined in June rather than gesture at it. Atlas is an omni transformer — text, image, video, and 3D fused into one shared spatial context via rectified-flow diffusion — that does "Real-to-Sim": film a room on a phone, get back a 3D environment a robot can be trained in, including rigid, articulated, and deformable object interactions. Notably: no weights, code, technical paper, or model card as of this writing — it's a capability announcement, not a reproducible result yet. Worth watching whether the technical report backs up the demo.

**"From Generation to Simulation: How Far Are World Models from Being True Simulators?"** (arXiv 2608.23070, Aug 24 2026)
[arXiv](https://arxiv.org/abs/2608.23070)
This is the paper the June digest's first "thread worth pulling" was asking for — a capability audit that takes Li's simulator contract seriously and scores real systems (Cosmos, GAIA, Genie among them) against it. Verdict: current world models have achieved functional substitution for interaction and controllability in narrow scenarios, but none provide formal physics guarantees, and state feedback — the model actually being told what its own last action did to the world — is the most neglected piece across the board. This is the most important framing update since June: it gives "simulator" a checklist instead of a vibe.

**Wayve GAIA-4** (August 2026)
[Wayve](https://wayve.ai/thinking/gaia-4/)
The GAIA line's fourth iteration puts Wayve's own driving policy *inside* the generated world in closed loop — the policy's decisions change what sensor input the world model generates next, not just narrated-over video. That's the difference between a renderer that shows you a plausible drive and a simulator you can actually evaluate a policy against: counterfactual branching where the agent's actions feed back into world state.

**NVIDIA Alpamayo 2 Super** (~August 2026)
[Coverage](https://www.unite.ai/nvidias-alpamayo-2-super-opens-robotaxi-development-to-commercial-use/) | [explainx.ai](https://www.explainx.ai/blog/nvidia-alpamayo-2-super-autonomous-driving-model-august-2026)
A 34B vision-language-action model for robotaxis, built on top of Cosmos 3's "Super Reasoner" and RL-post-trained for the rare causal-reasoning situations lane-keeping benchmarks don't cover. Useful as a taxonomy check: this is explicitly the **planner** consuming a **simulator's** world representation — a clean real-world instance of Li's three-way split, and a reminder that most of the commercial energy right now is landing on the planner layer because it's easier to make legible progress on than the simulator's physics-guarantee problem.

**Hippocampal place cells map terrain geometry independently of behavior** (*Science Advances*, 2026)
[Science Advances](https://www.science.org/doi/10.1126/sciadv.adz9893)
New evidence that place-cell firing encodes the geometric shape of the terrain surface itself — hills, slopes, drop-offs — rather than the animal's trajectory or behavior over it. That's a vote for a *surface-bound* cognitive map, not a volumetric occupancy grid. Interesting friction with the AI side of this watch: NeRF and Gaussian splatting represent scenes volumetrically (density fields, point clouds of Gaussians), while this result says the brain's spatial code may be organized around 2D surfaces embedded in 3D space. If that's right, mesh- or surfel-based scene representations might be the closer biological analogue than either NeRF or splatting.

---

### The LEARNING.md connection

You're in matrix decompositions right now — LU, QR, eigendecomposition, SVD — with SVD on the list alongside eigendecomposition. Here's where SVD specifically shows up in today's material.

Atlas's "Real-to-Sim" pipeline starts from ordinary phone video and has to recover camera poses before it can build a 3D scene — this is classical structure-from-motion, and the textbook move at its center is an SVD. Given two calibrated views of a static scene, you estimate the **essential matrix** E from point correspondences, then decompose it: E = U Σ V^T. Because E is rank-2 with two equal singular values by construction, that decomposition pulls the camera's rotation and translation directly out of U and V (there's a standard trick using a fixed rotation matrix W applied to U and V^T to extract R, and the translation direction falls out of the third column of U up to sign) — four candidate (R, t) pairs, disambiguated by checking which one puts the reconstructed points in front of both cameras (the cheirality check). That SVD step is *the* moment a pile of 2D pixel correspondences becomes a 3D camera trajectory — it's the hinge between "I have a video" and "I have a scene I can simulate a robot inside." Every SVD exercise you work through by hand is the same factorization Atlas's front-end is presumably running (or something numerically equivalent to it) millions of times to turn phone footage into robot training environments.

---

### Threads worth pulling

1. **The audit paper's "state feedback" gap.** If the single most-neglected piece across current world models is closing the loop between an agent's action and what the model generates next, that's exactly what GAIA-4 claims to have built for driving specifically. Worth a closer read of GAIA-4's technical description against the audit paper's criteria — does a domain-specific system (autonomous driving) get closer to "true simulator" than a general one (Atlas, Genie) precisely because the domain constrains the physics enough to fake fewer things?

2. **Surface-bound vs. volumetric scene codes.** The place-cell terrain result is a direct challenge to assuming NeRF/Gaussian-splatting-style volumetric density fields are the "natural" representation just because they're what's computationally convenient right now. Worth digging into whether any recent scene-representation work has moved toward explicit surface/mesh representations for this reason, rather than density fields — and whether that would actually help or just be a different set of tradeoffs.

---

### Papers added to covered list

- World Labs, "Atlas: A World Model for Spatial Intelligence," September 1 2026
- "From Generation to Simulation: How Far Are World Models from Being True Simulators?," arXiv 2608.23070, August 24 2026
- Wayve, "GAIA-4: Multimodal World Models Powering Closed-Loop Simulation for Safe and Scalable Autonomy," August 2026
- NVIDIA, "Alpamayo 2 Super," open reasoning model for robotaxis built on Cosmos 3 Super Reasoner, August 2026
- "Hippocampal place cells map terrain geometry independently of behavior," *Science Advances*, 2026

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
