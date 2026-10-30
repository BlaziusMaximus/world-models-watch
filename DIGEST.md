# World Models Watch — Digest

---

## 2026-10-30

*Sourcing note: arXiv and several other hosts were unreachable from this session, so everything below is from search snippets and secondary coverage, not full-text reads. Treat details as "reported", and the Atlas benchmark claims as World Labs' own. Also: there was a gap since the last entry (June 11), so this covers roughly June–October.*

---

### What landed

**World Labs unveils Atlas (Sept 1, 2026), early access**
[World Labs blog](https://www.worldlabs.ai/blog/atlas) | [The Decoder](https://the-decoder.com/world-labs-unveils-atlas-a-single-ai-model-that-generates-reconstructs-and-simulates-3d-worlds-from-just-a-few-photos/)
An "omni" world model pretrained from scratch on text, images, video, and 3D in one shared spatial context (autoregressive diffusion transformer). Reported outputs: camera-controlled video up to ~1 min at 1440p from a few reference images, *and* explicit 3D such as point clouds and Gaussian splats. Why it matters for the simulator framing: this is the first big attempt to put the renderer (video) and the state (explicit 3D) in one model, so the 3D isn't an afterthought extracted from pixels. It is the closest thing yet to Li's taxonomy being built as one system. Caveat: benchmarks are self-reported, no independent verification, no pricing or release date.

**AMD agrees to acquire World Labs (~Sept 28, 2026, reported ~$8.2B in stock)**
[Coverage](https://hokai.io/guides/amd-world-labs-marble-what-it-means)
Single secondary source in my results, so verify before repeating. If accurate, spatial-intelligence/simulator work is being pulled toward silicon vendors, same pattern as NVIDIA with Cosmos. Reported not to close until end of 2026; Marble's fate unclear.

**Runway GWM Worlds 2 (Sept 3, 2026), research preview**
[Overview](https://www.gradually.ai/en/ai-models/gwm-worlds-2/) | [Coverage](https://ai2.work/blog/runway-s-gwm-worlds-2-streams-playable-ai-worlds-in-real-time)
Real-time interactive worlds, 720p/24fps with 48kHz audio, autoregressive diffusion. Interesting mostly for what Runway admits: no image references past the first frame, imperfect long-term memory, detail degrading under fast camera motion. That's the simulator contract failing in plain view: the "memory" and viewpoint-consistency clauses are exactly where the interactive generators still break.

**LeWorldModel (LeWM), Maes, Le Lidec, Scieur, LeCun, Balestriero (arXiv 2603.19312)**
[arXiv](https://arxiv.org/abs/2603.19312v1)
Dated March 2026, so older than the window, but not yet covered and it's the counterweight to everything above. Claims the first JEPA that trains stably end to end from raw pixels with just two loss terms: next-embedding prediction plus a regularizer forcing latents toward a Gaussian. ~15M params, one GPU, planning reportedly up to 48x faster than foundation-model world models. Probing reportedly shows physical quantities in the latent, and a "surprise" signal flags physically implausible events. Simulator-relevant because it's the anti-pixel route: predict in latent space, and test physics by violation-of-expectation, the same logic developmental psychologists use on infants.

**GaussianSpline: Adaptive Bézier Splines for Highly Dynamic 4D Gaussian Splatting (arXiv 2610.35224, Oct 26)**
[arXiv](https://arxiv.org/abs/2610.35224)
Models each Gaussian's motion over time as an adaptive Bézier spline, aimed at highly non-uniform motion where fixed-basis 4DGS struggles. A simulator needs dynamics, and this is the representation side of that: explicit primitives with explicit trajectories. Still reconstruction, not physics, but trajectories-as-curves is a natural hook for adding dynamics priors later.

**Grid cells reducing aliasing in place representations (arXiv 2608.18569, Aug 2026)**
[arXiv PDF](https://arxiv.org/pdf/2608.18569)
Title only from my search: argues multi-scale grid codes reduce spatial aliasing in hippocampal place representations. Pairs with a May 2026 preprint on co-emergence of grid and place fields without supervising either ([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC13229076/)). I haven't read either closely; flagging for the neuroscience-to-architecture thread, not as a verified result.

---

### The LEARNING.md connection

Two spots, one of which fits cleanly.

**Atlas emitting Gaussian splats** is the Σ = R S Sᵀ Rᵀ story from the last entry, now at the *output* of a generative model. A model that outputs splats has to emit rotation (a quaternion, normalized to a valid R) and scale per primitive. It never emits Σ directly, for the same reason as in training: it can't guarantee PSD. Eigendecomposition is the parameterization contract between the generator and the renderer.

**LeWM's Gaussian-latent regularizer** is the stronger new link, and it's the "why does PSD matter" question from the other side. Representation collapse is a covariance problem: if latent embeddings collapse, the covariance matrix of a batch has eigenvalues going to zero (rank deficiency, which is what SVD shows you at a glance). Pushing latents toward an isotropic Gaussian means pushing the covariance toward the identity: all eigenvalues equal, nothing dead, nothing dominant. So "look at the eigenvalue spectrum of the embedding covariance" is the diagnostic for whether a world model's latent space is healthy. (I haven't read LeWM's exact regularizer, so this is my reading of what a Gaussian-latent penalty does, not a claim about their loss.) Contrast: in splatting you *parameterize* to keep Σ valid; in JEPAs you *penalize* to keep the latent covariance well-conditioned. Same object, constraint vs. regularizer.

The optimization track also touches here: constraint-by-parameterization (splats) versus constraint-by-penalty (LeWM) is a nice concrete instance of the Lagrange-multiplier idea coming up next.

---

### Threads worth pulling

1. **Constraint by parameterization vs. by penalty**: Splat covariances are made PSD by construction; JEPA latents are kept healthy by a soft loss. When is each the right call, and what does the eigenvalue spectrum look like in each failure mode? Natural bridge to your upcoming optimization material.

2. **Violation-of-expectation as the simulator test**: LeWM's surprise signal on implausible events, plus GWM Worlds 2's admitted memory failures, suggest a cheap way to grade "does it satisfy the simulator contract" without ground-truth physics. Worth designing what such a test would look like for Atlas-style models, and whether grid-cell-like codes would help the memory failure.

---

### Papers added to covered list

- World Labs, "Atlas" omni world model, Sept 1 2026
- AMD / World Labs acquisition report, Sept 2026 (single source, unverified)
- Runway, "GWM Worlds 2," research preview, Sept 3 2026
- Maes, Le Lidec, Scieur, LeCun, Balestriero, "LeWorldModel," arXiv 2603.19312
- Yachil & Sharf, "GaussianSpline: Adaptive Bézier Splines for Highly Dynamic 4D Gaussian Splatting," arXiv 2610.35224
- "The Role of Grid Cells in Reducing Spatial Aliasing in Hippocampal Place Representations," arXiv 2608.18569
- "A simple model of co-emergence of grid and place fields," arXiv 2605.21356

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
