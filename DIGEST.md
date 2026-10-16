# World Models Watch — Digest

---

## 2026-10-16

*Coffee-sized read. Caveat: my sandbox couldn't open primary pages today, so details come from search snippets and secondary coverage. Verify before quoting.*

---

### What landed

**AMD agrees to acquire World Labs (Sept 28, 2026), with Atlas announced Sept 1**
[The Next Web](https://thenextweb.com/news/amd-world-labs-8-2bn-fei-fei-li-chief-scientist) | [SDxCentral](https://www.sdxcentral.com/news/amd-acquires-ai-research-firm-world-labs-in-all-stock-deal-worth-82b/) | [Atlas explainer (secondary)](https://miraflow.ai/blog/world-labs-atlas-omni-world-model-spatial-intelligence-2026)
Reported as an all-stock deal of about $8.2B. Li would become AMD's chief scientist, and Johnson and Mildenhall would keep leading the research team. Atlas is World Labs' newer foundation model. It reportedly generates, reconstructs and simulates 3D worlds from one to three photos, and it will sit under future Marble versions and an API. Why it matters: the lab that defined the simulator framing is now being absorbed into a chip company, which tells you who expects spatial world models to drive inference demand. The open question is whether Atlas keeps an explicit 3D state, which is what the simulator contract needs, or only predicts novel views. Reports say it "predicts new camera views", which is renderer-flavored. Check this before trusting the "simulates" claim. I couldn't verify the RTFM research preview that one source mentions.

**"From Generation to Simulation: How Far Are World Models from Being True Simulators?" (Aug 24, 2026)**
[arXiv 2608.23070](https://arxiv.org/pdf/2608.23070)
The authors score generative world models against classical simulators on eight capabilities (asset construction, physics engine, interaction, controllability, stability, state feedback, diversity, evaluation). They map 200 works from 2018 to mid-2026 across latent-dynamics, video-generation and joint-embedding routes. Verdict: world models now substitute for simulators on interaction and controllability. They still lack formal physics guarantees and reproducible long-horizon rollouts. The sharpest number is that only 6 of 163 implementation papers expose a runtime interface for querying entity state or physical parameters. This is the empirical version of Li's June taxonomy: most systems are renderers with good controls.

**"Is Gaussian Splatting Becoming Neural Again?" (Sept 11, 2026)**
[arXiv 2609.12395](https://arxiv.org/pdf/2609.12395) | [radiancefields.com mirror](https://radiancefields.com/papers/is-gaussian-splatting-becoming-neural-again-a-taxonomy-and-controlled-study-of-learned-parameterization)
Wang, Cao and Zhang taxonomize 19 methods that put neural nets inside 3DGS along five axes: attribute decoding, spatial sharing, view-conditioned decoding, topology generation and amortized inference. Their controlled study on mip-NeRF 360 finds that sharing appearance and opacity across splats helps, while decoding geometric structure with a network adds nothing. They argue for "selective neuralization". For a simulator direction, this says the explicit geometric part of the representation is worth keeping explicit, and the learned part belongs on appearance.

**"Grounded world models in biological organisms and future embodied AI" (July 2026)**
[arXiv 2607.13560](https://arxiv.org/pdf/2607.13560)
A review framing biological world models as intrinsic neural dynamics progressively aligned to world structure through embodied action. I only saw the abstract-level summary, so treat it as a reading-list item. It pairs with Rolls and the PNAS paper from the June entry.

---

### The LEARNING.md connection

Today's best link is items 2 and 3 together, and it lands on the Σ = R S Sᵀ Rᵀ factorization you're studying.

The "neural again" study says that learning a network to decode *geometry* gives no gain over raw per-splat parameters, while sharing *appearance* does. Geometry here means mostly the covariance. Per-splat Σ is already a compact, well-conditioned object: rotation R (the eigenvectors, orthogonal) and scale S (the square roots of the eigenvalues, positive). Gradient descent can move each primitive freely because the factorization guarantees PSD. There is little left for a network to fix. A network that decodes Σ would have to re-learn that PSD constraint or hide it behind a parameterization like this one.

The simulator survey's "state feedback" gap points the same way. A splat scene is queryable: you can read off any primitive's center, its principal axes (eigenvectors of Σ) and its extents (eigenvalues). That is the kind of runtime state interface that 157 of 163 world-model papers lack. Eigendecomposition is therefore doing two jobs: it keeps optimization stable, and it makes the geometry legible as state. Implicit video-model weights offer neither.

---

### Threads worth pulling

1. **Can you give a video world model a state-feedback interface?** Hybrid designs (a Gaussian or occupancy state under a generative renderer) are the "cross-route hybridization" the survey calls for. What would it take for Atlas-like models to expose splat-like state you can query and perturb counterfactually?

2. **Why does geometry resist neuralization while appearance doesn't?** The study's finding fits a story about low-dimensional, constraint-heavy structure (PSD matrices, rotations) versus high-correlation appearance. It's a good entry point into optimization (conditioning, parameterizing constraints) once you reach that block of LEARNING.md.

---

### Papers added to covered list

- AMD / World Labs acquisition announcement and World Labs "Atlas," Sept 2026
- Wang et al., "From Generation to Simulation: How Far Are World Models from Being True Simulators?" arXiv 2608.23070
- Wang, Cao, Zhang, "Is Gaussian Splatting Becoming Neural Again? A Taxonomy and Controlled Study of Learned Parameterization," arXiv 2609.12395
- "Grounded world models in biological organisms and future embodied AI," arXiv 2607.13560

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
