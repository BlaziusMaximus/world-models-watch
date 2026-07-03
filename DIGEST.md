# World Models Watch — Digest

---

## 2026-07-03

### What landed since the last check-in

**World Labs ships Marble 1.1 / 1.1 Plus (June 12, 2026)**
[Release notes](https://docs.worldlabs.ai/marble/release-notes) | [Coverage](https://quasa.io/media/marble-1-1-world-labs-just-made-their-world-model-significantly-better)
First substantive Marble update since the taxonomy post covered last time. Two changes matter for the simulator framing specifically: cleaner geometry (fewer artifacts in the underlying Gaussian splat reconstruction — a direct fidelity improvement to the representation itself, not just the renderer output), and Marble 1.1 Plus, which auto-expands spatial coverage when a prompt implies a bigger environment than one splat pass would normally cover. That second point is a small but real step toward the simulator contract: the system is starting to reason about how much *world* a prompt implies, rather than just how much frame.

**Kairos: A Native World Model Stack for Physical AI (arXiv 2606.16533, June 15, 2026)**
[arXiv](https://arxiv.org/abs/2606.16533) | [Hugging Face](https://huggingface.co/papers/2606.16533)
A 4B-parameter unified model for world understanding, generation, and action that beats 14B baselines on embodied benchmarks. The interesting piece isn't the size efficiency — it's the "Hybrid Linear Temporal Attention": sliding-window attention for local dynamics, dilated windows for mid-range dependencies, and gated linear attention carrying persistent global memory, with a *formal bound* on how much error this factorization lets accumulate over long horizons. That's a direct attack on one of the hardest parts of the simulator contract — dynamics that hold up over time, not just over the next few frames — and it's rare to see an actual mathematical guarantee attached to that claim rather than an empirical one.

**"Critiques of World Models" gets a major revision — the PAN counter-proposal (arXiv 2507.05169, revised June 17, 2026)**
[arXiv](https://arxiv.org/abs/2507.05169) | [Hugging Face](https://huggingface.co/papers/2507.05169)
Eric Xing and coauthors argue the field has narrowed "world model" down to mean "JEPA-style latent-space video prediction," and that this is a mistake — you can't get physical, agentic, or nested (hypothetical-within-hypothetical) reasoning out of a single continuous latent predictor. Their proposed alternative, PAN, bolts an LLM (high-level symbolic/causal reasoning) onto a diffusion model (low-level continuous dynamics) in a hierarchical stack. Read against Li's renderer/simulator/planner taxonomy from last cycle, this is essentially the same worry from a different lab: current systems are very good at the renderer's job and are quietly being credited with the simulator's. Worth a deeper look at whether PAN's LLM-diffusion split actually earns geometric consistency and physical law-following, or just relocates the same gap.

**EntON: Eigenentropy-Optimized Neighborhood Densification in 3D Gaussian Splatting (arXiv 2603.06216)**
[arXiv](https://arxiv.org/abs/2603.06216)
This one's from March and I'd missed it — surfacing it now because it's an unusually clean case study for what you're studying right now. Covered below.

**Structure Abstraction and Generalization in a Hippocampal-Entorhinal Inspired World Model (arXiv 2605.15733, May 15, 2026)**
[arXiv](https://arxiv.org/abs/2605.15733)
Zhang, Lyu, Liu & Wu propose a hierarchical model that splits a predictive visual world model into two coupled pieces: an entorhinal-cortex-like module holding *relational structure* (the abstract transition dynamics — how things move and transform, independent of what they look like) and a hippocampus-like module holding *integrated episodic scenes* (the concrete bound-together content). Trained this way, the structural part transfers to new scenes the content-only part can't. This is a sharper, more architectural version of the PNAS piece from last cycle (spatial and semantic representations as one computation) — here the claim is that separating "the shape of change" from "the content being changed" is what makes the structure reusable at all, which is exactly the generalization problem that makes video-prediction-only world models brittle across novel scenes.

---

### The LEARNING.md connection

You're mid-way through eigendecomposition, with PSD matrices as the throughline — last cycle's connection was Gaussian splatting's Σ = RSSᵀRᵀ parameterization for gradient descent. EntON gives you a different, more specific angle on the same matrix.

For every local neighborhood of Gaussians in a splat reconstruction, EntON computes the covariance matrix of the neighborhood's spatial scatter, takes its eigenvalues, normalizes them to sum to 1 (so they read like a probability distribution over the three principal directions), and computes the Shannon entropy of that distribution — the "eigenentropy." High eigenentropy means the local neighborhood is roughly spherical: the scatter is spread evenly across all three eigen-directions, no single direction dominates. Low eigenentropy means one or two eigenvalues dominate the others — the neighborhood is needle-like or plate-like, concentrated along a line or a plane. That's precisely the geometric reading LEARNING.md already has for eigenvectors ("directions that only stretch, never rotate") pushed one step further: the *eigenvalues themselves*, once normalized, become a shape descriptor. EntON's rule is: split Gaussians in low-eigenentropy (flat, anisotropic) regions preferentially, because that's where fine surface detail — edges, thin structures — lives, and where a single blobby Gaussian is most likely to be hiding real geometry. So the algorithm's entire densification decision reduces to: compute eigenvalues of a local covariance matrix, then ask how *unevenly* they're distributed. That's an eigendecomposition problem wearing an information-theory costume — the same operation you're studying, applied to decide where a 3D reconstruction needs more capacity.

---

### Threads worth pulling

1. **PAN vs. the latent-only mainstream**: Xing et al.'s critique is basically Li's renderer/simulator distinction, restated as an architectural indictment rather than a taxonomy — with a concrete (if hybrid and inelegant) counter-proposal bolting an LLM onto a diffusion model. Worth a session on whether PAN's hierarchical split actually buys geometric consistency and physical-law adherence that a pure JEPA-style latent predictor doesn't, or whether it's the same gap moved up a level of abstraction.

2. **Structure/content separation as a world-model inductive bias**: the hippocampal-entorhinal paper's split — relational structure in one module, bound episodic content in another — is a concrete, testable version of "what would it take for a world model to generalize physics across scenes it's never seen." Worth comparing directly against how current video-prediction world models (Genie 3, Cosmos 3) fail to generalize dynamics across novel scene content, and whether an explicit structure/content split like this is the missing inductive bias.

---

### Papers added to covered list

- World Labs, "Marble 1.1 / 1.1 Plus" release, June 12, 2026
- "Kairos: A Native World Model Stack for Physical AI," arXiv:2606.16533, June 15, 2026
- Xing E, Deng M, Hou J, et al., "Critiques of World Models" (PAN architecture), arXiv:2507.05169, revised June 17, 2026
- "EntON: Eigenentropy-Optimized Neighborhood Densification in 3D Gaussian Splatting," arXiv:2603.06216
- Zhang T, Lyu M, Liu X, Wu S, "Structure Abstraction and Generalization in a Hippocampal-Entorhinal Inspired World Model," arXiv:2605.15733, May 15, 2026

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
