# World Models Watch — Digest

---

## 2026-10-09

*A quieter stretch than June. One real headline (Atlas), one useful splatting study, and some context on the field's secrecy. Search tools couldn't open primary pages this run, so details come from search summaries; company numbers are self-reported.*

---

### What landed

**World Labs releases Atlas (Sept 1, 2026)**
[World Labs blog](https://worldlabs.ai/blog/atlas) | [The Decoder](https://the-decoder.com/world-labs-unveils-atlas-a-single-ai-model-that-generates-reconstructs-and-simulates-3d-worlds-from-just-a-few-photos/)
One "omni" model, pretrained from scratch on text, image, video and 3D, as a multimodal autoregressive diffusion transformer over a shared spatial context. It does camera-controlled video (up to a minute at 1440p, with camera pose as a geometric input rather than a text prompt), 3D reconstruction from as few as one image (point clouds, Gaussian splats), and real-to-sim scene capture for robots. Why it matters for the simulator direction: it's the first World Labs system that puts generation, explicit 3D, and sim under one set of weights. That is Li's taxonomy (renderer / simulator / planner) shipping as a single product. Caveat: dynamics and physics claims are thin so far. It's strong on the geometry half of the contract, and the benchmarks (e.g. 25.3 vs 28.7 error on few-view reconstruction) are company-reported, with access limited to partners.

**"Is Gaussian Splatting Becoming Neural Again?" (arXiv 2609.12395, Sept 2026)**
[arXiv](https://arxiv.org/abs/2609.12395)
Wang, Cao, Zhang (UTS) taxonomize 19 methods that use networks to produce or share Gaussian parameters (attribute decoding, spatial sharing, view-conditioned decoding, topology generation, amortized inference). In a controlled mip-NeRF 360 test, sharing appearance and opacity across the scene helps. Having a network decode *geometry* adds nothing. Their takeaway is "selective neuralization": learn the reusable parts, leave local geometry explicit. It bears on the representation question in CONTEXT.md. The NeRF→splat arc may end in a hybrid where explicit primitives carry geometry and learned functions carry appearance, which suits a simulator that needs inspectable, editable state.

**Context: world-model labs are going dark (TechCrunch, Sept 18)**
[TechCrunch](https://techcrunch.com/2026-09-18/world-model-companies-are-keeping-a-lot-of-secrets/)
AMI Labs and World Labs both say they're in a research and building phase and won't discuss timelines. Even data suppliers say they don't know what customers are building. Why it matters: claims like Atlas's can't easily be independently checked, which makes open work (Cosmos 3's open weights, academic benchmarks) more valuable as a reference point. (A report of an AMD–World Labs acquisition circulated, but I found only one low-quality source, so I'm not treating it as real.)

**Brain side: thin this cycle**
Nothing new bridging neural scene representations to 3D scene models in AI turned up. The nearest item is a June 2026 review (Ouchi, *Neurosci Res*) arguing place and grid cells encode *future* positions, not just current ones. I only saw the abstract. It's still worth a look alongside your earlier Rolls note: a predictive code is closer to what a simulator does than a purely positional one.

---

### The LEARNING.md connection

The splatting-neuralization paper lands right on your eigendecomposition track. Recall Σ = R S Sᵀ Rᵀ: rotation R holds the eigenvectors, S the scales (square roots of the eigenvalues). That *is* the "local geometric freedom" the authors say not to take away. Their experiment shows that replacing those per-Gaussian (R, S) parameters with a network's output buys nothing, while sharing appearance and opacity does. So the factorization you're studying isn't just a trick for staying inside the PSD cone. It's the right place to leave things explicit and directly optimizable. Each splat's eigenstructure is its own geometry (which way it's elongated, how thin it is), and that's what you'd read off to ask "is this a surface or a blob?" A simulator that wants inspectable 3D state benefits from being able to look at those eigenvalues. A decoder hiding them in weights removes that.

If you want a next-step exercise: take one Gaussian's Σ, compute its eigenvalues, and notice that a very small smallest eigenvalue means the splat is a flat disc, which is how splats approximate surfaces. Atlas outputting splats means that structure is what its reconstructions look like under the hood.

---

### Threads worth pulling

1. **Does Atlas satisfy Li's simulator contract?** It handles geometric consistency well by construction, but what about dynamics? Worth working out what a test of "obeys physics" would even look like for a model that outputs video and splats, and whether real-to-sim for robots quietly outsources the physics to a conventional engine.

2. **Selective neuralization as a design principle for simulators.** If explicit geometry plus shared learned appearance wins in splatting, is there an analog in world models: explicit 3D state with learned dynamics and appearance heads? Compare to Atlas's "shared spatial context" and to the predictive grid-cell story. Is the brain's code explicit-geometry-plus-learned-prediction too?

---

### Papers added to covered list

- World Labs, "Atlas: A World Model for Spatial Intelligence," Sept 1 2026
- Wang, Cao, Zhang, "Is Gaussian Splatting Becoming Neural Again? A Taxonomy and Controlled Study of Learned Parameterization," arXiv 2609.12395, Sept 2026
- Brandom, "World model companies are keeping a lot of secrets," TechCrunch, Sept 18 2026
- Ouchi, review on predictive place/grid cell representations, *Neurosci Res*, June 2026 (abstract only)

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
