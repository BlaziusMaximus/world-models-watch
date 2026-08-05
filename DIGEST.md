# World Models Watch — Digest

---

## 2026-08-04

*Hands-on entry — not the weekly paper scan. This is the home-server swarm's **own** work landing in the feed: a proof-of-pipeline built by hand on the RTX 4090, surfaced here so the morning routine references what you've built alongside what you've read. This is the "connect swarm output to the habit" bridge from your backlog §6 ("World-Model North Star — Concrete Hands-On Bridge").*

### What the swarm built — COLMAP → 3D Gaussian Splatting, end-to-end on the 4090

**Renderer-layer proof-of-pipeline: image set → COLMAP poses → 3DGS → novel views + flythrough.**
The full classic pipeline now runs end-to-end on the desktop 4090 (gsplat 1.5.3, MCMC trainer). It takes the Tanks & Temples "truck" scene — 251 registered camera poses and 136,029 triangulated SfM points from genuine COLMAP models — trains a 3D Gaussian model, and renders held-out novel views plus a 240-frame orbit flythrough. Two runs:

| Run | Resolution | #Gaussians | Steps | PSNR | SSIM | LPIPS | Train time | Peak GPU |
|-----|-----------|-----------:|------:|-----:|-----:|------:|-----------:|---------:|
| hero | 979×546 (full) | 150,000 | 7,000 | 23.18 | 0.823 | 0.189 | ~30 s | 0.25 GB |
| tiny | 489×273 (½) | 136,029 | 4,000 | 23.67 | 0.865 | 0.102 | ~14 s | 0.18 GB |

This is squarely the **renderer** tier of Li's taxonomy — the concrete, contained thing §6 asked for ("just get COLMAP + one splatting implementation running on a short test clip and report back") before the simulator layer means anything beyond theory. It is deliberately the *conservative* baseline: the in-swarm 2026-07-28 digest flagged that the frontier has moved to *feed-forward* single-image→splat methods (AnySplat / Flash3D), which is the natural next hands-on demo and needs far less capture discipline.

**Operational finding worth keeping.** The run had to coexist with the 4090's presence-gated local LLM serve (a `qwen2.5:32b` holding ~18.8 of 24 GB while you were away), leaving ~1.1 GB free. Naïve full-density 3DGS OOM'd; a memory-bounded MCMC run (hard Gaussian cap + expandable segments) peaked at only ~0.18–0.25 GB and coexisted without touching the serve. The general shape: heavy GPU experiments and the away-hours LLM serve both want the 4090 — capped/low-res proofs fit alongside it, but a *large* splat/NeRF/training job needs a dedicated window.

*Full run report + artifacts (novel-view renders, flythrough mp4s, SfM reprojection / point-cloud views, COLMAP stats) live in the swarm sandbox and are CopyParty-served at `projects/migrated-routines/sandbox/world-model-bridge/run-report-2026-08-03.md`. All compute/artefacts stay on the 4090 desktop under `~/world-model-bridge/`; nothing is wired into the live swarm.*

### The LEARNING.md connection

You're studying eigendecomposition and how to parameterize PSD matrices so gradient descent can optimize them — and this run is that exact math *executing on real hardware*. Every one of the 136,029 Gaussians COLMAP seeded and gsplat optimized stores its shape as Σ = R S Sᵀ Rᵀ (the eigendecomposition split into an orthogonal rotation R and a diagonal scale S) precisely so the optimizer never has to project back onto the PSD cone — the constraint your covariance-matrix notes keep circling. The MCMC `cap-max` strategy that held the whole run inside 0.25 GB is doing eigenvalue-style bookkeeping to decide which Gaussians to relocate or prune under a fixed budget. So the "why factor a matrix" question from the linear-algebra track has now literally run on the box: the PSD parameterization pays off as a training curve you can watch, not just a proof you can read.

### Threads worth pulling

1. **The feed-forward fast path.** The 2026-07-28 in-swarm digest's headline for the renderer tier was that single-image→splat (AnySplat / Flash3D) is production-adjacent. Running one of those on a phone video of the apartment or shop is the cheapest "wow" next step and a truer test of where the field actually is in 2026 — worth a dedicated 4090 window while you're around (serve down).
2. **GPU contention as a scheduling problem.** The serve-vs-experiment collision above is a recurring constraint, not a one-off. Worth a standing policy — experiments claim the 4090 while you're present; the serve owns the away hours — rather than fighting it each run.

### Added to the swarm's hands-on log

- COLMAP → 3DGS proof-of-pipeline on the RTX 4090 — Tanks & Temples "truck", gsplat 1.5.3 MCMC, run 2026-08-03. *(migrated-routines task #537; feed-wiring task #634.)*

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
