# World Models Watch — Digest

---

## 2026-10-02

*Caveat: arXiv and most blogs were blocked from this sandbox, so details below come from search snippets and secondary coverage, not full-text reads. Treat specifics as "per reports" until you open the papers.*

### What landed

**World Labs ships Atlas (Sept 1, 2026)**
[SiliconANGLE](https://siliconangle.com/2026/09/01/fei-fei-lis-world-labs-debuts-atlas-a-world-model-showcase-for-advanced-spatial-intelligence/) | [DEV writeup](https://dev.to/adilaidev/atlas-world-labs-new-omni-world-model-for-spatial-intelligence-1kp8) | [skeptical take (Kingy AI)](https://kingy.ai/blog/world-labs-atlas-world-model-deep-dive/)
A multimodal autoregressive diffusion transformer taking text, images, video, camera poses and depth, and emitting video *and* explicit 3D (point clouds, Gaussian splats). Reported: up to ~a minute of 1440p video from one image with rigid geometric consistency, plus a real-to-sim path (phone capture → navigable robot sim). Why it matters: this is the first thing from Li's own lab that tries to be renderer and simulator at once — generation and an explicit, queryable 3D state in one model. The skeptical coverage's headline is the right question: it's a serious spatial generator that hasn't yet shown *dynamics/physics*. Marble also went generally available (Sept 13).

**"From Generation to Simulation: How Far Are World Models from Being True Simulators?" (arXiv 2608.23070, Aug 24)**
[arXiv HTML](https://arxiv.org/html/2608.23070v1) | [repo](https://github.com/AtongWang/world-model-simulators)
Audits 200 world-model papers (2018–Jun 2026) against eight things a classical simulator provides: asset construction, physics engine, interaction, controllability, stability, state feedback, diversity, evaluation. Verdict: models have substituted for simulators in interaction/controllability, but lack formal physics guarantees, reproducible long-horizon rollouts, and — the striking number — only 6 of 163 implementation papers expose a runtime interface to query entity state or physical parameters. This is basically your simulator contract turned into a scorecard, and it's a ready-made lens for judging Atlas/Cosmos 3/Genie 3.

**Grid cells fix place-cell aliasing (Johnson, Ghizawi, Minai; arXiv 2608.18569)**
[arXiv](https://arxiv.org/abs/2608.18569)
Place cells built only from boundary-vector inputs confuse locations in symmetric/repetitive environments. Adding multiple analytically constructed grid-cell modules cut aliasing 94–99%, most in the most visually symmetric environment. Takeaway for simulators: geometry-from-sensing alone is ambiguous; an internally generated, periodic positional code (path-integration-like) disambiguates — an argument for explicit pose/position state rather than appearance-only memory.

**Also on the radar (not read, just flagged):** GaussianDream++ ([2608.25659](https://arxiv.org/pdf/2608.25659), Gaussian world modeling for manipulation) and Envision4D ([2606.10656](https://arxiv.org/pdf/2606.10656), feed-forward 4D Gaussian splatting that predicts future driving frames) — both push Gaussians from "reconstruction" toward "predictive state."

---

### The LEARNING.md connection

Eigendecomposition again, now aimed at the audit paper's biggest gap: **state feedback**. Atlas reportedly outputs Gaussian splats, and a splat's covariance is stored as Σ = R S Sᵀ Rᵀ. That factorization is not just a trick to keep Σ positive semi-definite for gradient descent — it makes the state *legible*. The columns of R are the principal axes of each blob, the diagonal of S its extents along them; you can read off "this primitive is a 2 cm × 2 cm × 30 cm thin rod pointing along (0.1, 0.9, 0.4)" without any further computation. That is exactly the kind of runtime-queryable entity state that 157 of 163 world models don't expose. A pure-video model has to be probed to learn anything about geometry; an eigendecomposed primitive hands it to you. And it points forward: a physics engine wants inertia tensors, which are themselves symmetric PSD matrices diagonalized into principal axes — the same math, one step on. (Optimization connection, for later: the aliasing paper is, loosely, about an ill-conditioned inverse problem — many locations map to near-identical sensory vectors — and grid modules add independent coordinates that break the degeneracy.)

---

### Threads worth pulling

1. **Score Atlas against the eight-capability audit.** Which of asset construction, physics, state feedback, stability does a "generate + reconstruct + simulate" model actually satisfy? Does exposing Gaussian splats count as state feedback, or is it a render-side artifact with no dynamics attached?

2. **From splat covariance to inertia tensor.** Could you read rigid-body physical parameters (principal axes, moments) straight off a reconstructed Gaussian scene? What breaks — density, mass, contact — and does anyone do it? A nice way to make eigendecomposition feel pointed at "simulator."

---

### Papers added to covered list

- World Labs, "Atlas," Sept 1 2026 (and Marble general availability, Sept 13 2026)
- Wang et al., "From Generation to Simulation: How Far Are World Models from Being True Simulators?" arXiv 2608.23070
- Johnson, Ghizawi, Minai, "The Role of Grid Cells in Reducing Spatial Aliasing in Hippocampal Place Representations," arXiv 2608.18569
- GaussianDream++, arXiv 2608.25659 (flagged only)
- Envision4D, arXiv 2606.10656 (flagged only)


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
