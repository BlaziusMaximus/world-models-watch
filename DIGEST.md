# World Models Watch — Digest

---

## 2026-09-18

*First check-in since June 11 — a wide window, so this leans toward the handful of things that actually moved the simulator question, not a full three-month scroll.*

---

### What landed this quarter

**World Labs ships Atlas, an omni world model (September 1, 2026)**
[World Labs blog](https://www.worldlabs.ai/blog/atlas) | [Radiance Fields coverage](https://radiancefields.com/world-labs-announces-new-world-model-atlas)
Atlas is a multimodal autoregressive rectified-flow transformer that folds images, camera poses, and depth maps into one shared 3D spatial context, then generates conditioned on it — camera motion is supplied as an actual geometric path, not a text description of "pan left." It outputs up to a minute of video at 1440p, and critically, it writes its output natively as point clouds or Gaussian splats rather than just pixels. Feed it two unrelated reference images and it will invent the geometry connecting them. This matters for the simulator framing specifically because it's an explicit bet that generation and 3D reconstruction should be the *same* model rather than a generator bolted to a separate reconstruction pipeline — geometric consistency as an architectural commitment, not a post-hoc check. Whether that shared context actually holds up as a persistent scene state (Li's "simulator" contract) versus a very good autoregressive renderer is exactly the open question below.

**"From Generation to Simulation: How Far Are World Models from Being True Simulators?" (arXiv, August 2026)**
[arXiv:2608.23070](https://arxiv.org/abs/2608.23070)
This is the paper CONTEXT.md has been waiting for: a capability audit, not another architecture. The authors define eight concrete simulator capabilities — asset construction, physics engine, interaction, controllability, stability, state feedback, diversity, evaluation metrics — and score 200 world-model papers (2018–June 2026) against them across three technical routes (latent dynamics, video generation, joint-embedding prediction). The headline finding: current systems have functionally substituted for simulators on interaction and controllability in narrow scenarios, but fail hard on formal physical guarantees, long-horizon reproducibility, and above all **state feedback** — only 6 of 163 implementation papers expose any runtime interface for querying an entity's position, velocity, or physical parameters. That's a sharp, falsifiable version of the renderer/simulator line Li drew in June: a system that can't answer "where is this object and how fast is it moving" isn't a simulator no matter how good the frames look. A companion piece, "A Definition and Roadmap for World Models" ([arXiv:2607.06401](https://arxiv.org/abs/2607.06401)), converges on nearly the same conclusion from theory rather than audit, defining a world model as a compression of state-transition dynamics under finite compute — which is itself a claim that the *state* has to exist as a queryable object, not just something implicit in weights.

**Wayve launches GAIA-4 (August 3, 2026)**
[Wayve GAIA-4](https://wayve.ai/thinking/gaia-4/)
GAIA-4 puts Wayve's actual AI driver inside the world model's generation loop: it reconstructs driving scenes as video and radar, and the driver's decisions change what sensor input it receives next — a real closed loop, not scripted replay. It supports world-on-rails replay, reactive agents, and counterfactual scenario generation, and Wayve is explicit that the point is now safety *evaluation* at scale, not just synthetic training data. This is the AV thread from CONTEXT.md's connected interests moving from "generate plausible video" toward "state feedback closes a control loop" — the same gap the capability-audit paper above flags as most neglected industry-wide, being addressed in one specific, high-stakes deployment.

**"Is Gaussian Splatting Becoming Neural Again?" (arXiv, September 11, 2026)**
[arXiv:2609.12395](https://arxiv.org/abs/2609.12395)
A taxonomy-and-controlled-study paper asking whether 3DGS's recent trend of using small neural nets to generate or share Gaussian parameters is quietly turning an explicit representation back into an implicit one. They test 19 methods along five axes (attribute decoding, spatial sharing, view-conditioned decoding, topology generation, amortized inference) and find the honest answer is "it depends on which attribute": sharing appearance and opacity across nearby Gaussians via a learned function improves reconstruction, but decoding *geometry* through a network buys nothing over letting each Gaussian's position and shape stay explicit and locally free. Relevant to CONTEXT.md's open question about where 3D structure should live explicitly vs. implicitly in weights — this is a controlled experiment on exactly that boundary, one attribute at a time, inside a single representation.

**"Faster 3D Gaussian Splatting Convergence via Structure-Aware Densification" (SIGGRAPH 2026)**
[Project page](https://vcai.mpi-inf.mpg.de/projects/SAD-GS/)
Max Planck Institute for Informatics. Standard 3DGS decides whether to split a Gaussian using screen-space positional gradients alone, which can't tell geometric misplacement apart from a Gaussian just being too coarse to cover a busy texture — so training either over-blurs fine detail or over-densifies everywhere. SAD-GS instead looks at the multiscale structure of the *input image* at that location and compares it to the Gaussian's projected footprint before deciding to split, and in which direction. Converges dramatically faster (53s to target quality on Mip-NeRF360) with better perceptual detail. See the LEARNING.md connection below — this is the same matrix math as last time, aimed at a different matrix.

**"The Hippocampus as a Small-World Cognitive Map" (bioRxiv, May 2026)**
[bioRxiv](https://www.biorxiv.org/content/10.64898/2026.02.07.704615v1.full)
Kim et al. model hippocampal cognitive maps as small-world graphs: dense local clustering preserves fine spatial/relational structure, while a small number of long-range "bridge" connections give shortcut access between distant states — the same topology that makes social networks and neural wiring efficient to search. Framed explicitly as a solution to a search problem intelligent systems share: how do you keep an internal model of the world both locally detailed and globally navigable without it becoming too expensive to query. That's a structural hint worth sitting with for CONTEXT.md's brain-inspired-inductive-bias question — most artificial scene representations (voxel grids, splat clouds, NeRFs) are locally dense but have no analog of long-range shortcut edges at all.

---

### The LEARNING.md connection

Last entry's connection was Σ = R S S^T R^T — the 3D covariance of a Gaussian splat factored into rotation and scale so gradient descent never has to stay inside the PSD cone, with the largest eigenvalue used to decide when a Gaussian is too elongated and should split. SAD-GS (above) keeps that machinery but adds a *second* eigendecomposition earlier in the pipeline: it builds a 2×2 structure tensor from local image gradients (J = ΣᵢΣⱼ ∇I ∇Iᵀ over a window), and the eigenvectors of that tensor point along the dominant and minor directions of local image variation, while the eigenvalues measure how strong that variation is in each direction. That's exactly the "directions that only stretch, never rotate" reading of eigendecomposition from LEARNING.md — applied not to the Gaussian's own shape this time, but to the *target* the Gaussian is trying to cover. The split decision becomes a comparison between two eigen-structures: the Gaussian's own covariance eigenvalues versus the image structure tensor's eigenvalues at that location. Once you can read a symmetric matrix's eigenvalues as "how much variation, in which directions," you start seeing the same move everywhere in this field — it isn't a Gaussian-splatting-specific trick, it's the generic way anisotropic decisions get made from local structure.

---

### Threads worth pulling

1. **State feedback as the missing simulator primitive.** The capability-audit paper's number (6 of 163) is concrete enough to build a session around: what would a minimal runtime state-feedback API for a world model even look like — an entity registry with position/velocity/mass that the generator has to stay consistent with, rather than pixels the generator merely renders? Atlas's "shared spatial context" and GAIA-4's closed loop are two very different partial answers to the same question.

2. **Small-world shortcuts for scene representations.** The hippocampus paper's local-clustering-plus-long-range-bridges structure has no obvious analog in NeRF/Gaussian-splat scene graphs, which are locally dense but topologically flat — every query walks the same spatial structure regardless of distance. Worth a deeper look at whether anything like a bridge-edge shortcut has been tried for large-scale scene reconstruction (the ICLR 2026 "Signal Structure-Aware Gaussian Splatting for Large-Scale Scene Reconstruction" poster that turned up in this search might be adjacent — not yet read closely enough to cover properly).

---

### Papers added to covered list

- World Labs, "Atlas: A World Model for Spatial Intelligence," September 1 2026
- "From Generation to Simulation: How Far Are World Models from Being True Simulators?," arXiv:2608.23070, August 2026
- "A Definition and Roadmap for World Models," arXiv:2607.06401, July 2026
- Wayve, "GAIA-4: Multimodal World Models Powering Closed-Loop Simulation," August 3 2026
- "Is Gaussian Splatting Becoming Neural Again? A Taxonomy and Controlled Study of Learned Parameterization," arXiv:2609.12395, September 2026
- Lyu et al., "Faster 3D Gaussian Splatting Convergence via Structure-Aware Densification," SIGGRAPH 2026
- Kim et al., "The Hippocampus as a Small-World Cognitive Map," bioRxiv, May 2026

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
