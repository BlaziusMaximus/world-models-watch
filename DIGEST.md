# World Models Watch — Digest

---

## 2026-11-06

*Sourcing note: arXiv and Wikipedia are blocked in this run's network, so the items below come from search results and secondary coverage, not the papers or primary posts. Treat the details as unverified until you open the originals.*

---

### What landed

**DeepMind Genie 4 (Oct 27, 2026, research preview)**
[Unite.AI](https://www.unite.ai/?p=495220) | [Northeast Times](https://northeasttimes.com/2026/10/27/google-deepmind-opens-genie-4-preview-to-testers-with-audio-and-two-minute/) | [explainer](https://www.eesel.ai/blog/genie-4)
Real-time interactive world model that now generates audio natively (music, ambience, dialogue). Coverage says it also simulates aerodynamics and collisions, adds characters that react to you, and has Gemini agents break goals into steps inside the simulated world. For the simulator framing: the claim to check is "collisions and aerodynamics". Is that a learned physics prior or just plausible-looking motion? There are no benchmarks or weights yet, so the contract (viewpoint consistency, counterfactuals) can't be tested from outside. The Gemini-in-the-loop setup is the planner-reads-from-simulator arrangement in Li's taxonomy.

**Runway GWM Worlds 2 (Sep 3, 2026)**
[Runway research](https://runway.com/research/introducing-gwm-worlds-2)
Real-time interactive worlds at 720p/24fps with audio, with no fixed duration limit. The notable detail is how it was built: a bidirectional audio-video generator was too slow, so it was post-trained into a causal autoregressive model. That distillation from bidirectional to autoregressive is the usual trade-off for making a generator interactive, and it is where long-horizon drift tends to come from. It is a renderer-leaning system with little explicit state, a useful contrast to the simulator ideal.

**GlobalSplat and AnchorSplat: feed-forward Gaussian splatting moves off pixel-aligned primitives (Apr 2026)**
[GlobalSplat arXiv:2604.15284](https://arxiv.org/abs/2604.15284) | [AnchorSplat arXiv:2604.07053](https://www.emergentmind.com/papers/2604.07053)
Earlier feed-forward 3DGS predicted one Gaussian per pixel per view, so splat counts grew with the number of views and the same surface got covered redundantly. GlobalSplat first builds a global latent scene representation and then decodes a fixed budget (reported as 16K) of Gaussians, regardless of how many views came in. AnchorSplat uses anchor-aligned Gaussians with multi-view-stereo priors and reports up to 20× fewer Gaussians. This matters for a simulator because a scene state whose size doesn't depend on how you looked at it is closer to a persistent world state than a per-frame artifact. It also looks like the explicit-plus-implicit hybrid: a latent holds the scene, and Gaussians are the readable output.

**WildSplat (arXiv:2607.05347, ECCV 2026)**
[arXiv](https://arxiv.org/abs/2607.05347)
Feed-forward 3DGS from unposed, in-the-wild photos with changing lighting, and appearance-conditioned novel views. Worth noting because it separates scene structure from appearance, which is the same separation a simulator needs (geometry persists, lighting and weather are variables you can change).

**NeuroHex (arXiv:2603.00376, 2026)**
[arXiv](https://arxiv.org/abs/2603.00376)
Proposes a hexagonal coordinate system for world models built on grid-cell-like reference frames. Per the snippet, initial tests are only 2D. It's a small, early bridge between grid-cell geometry and ML world models, not yet evidence for anything in 3D. I'd file it as a signal that people are trying this, not as a result.

---

### The LEARNING.md connection

Eigendecomposition again, from the feed-forward direction. In GlobalSplat-style models, a network has to *output* every Gaussian's covariance in one shot, with no per-scene optimization loop to fix mistakes. Σ = R S Sᵀ Rᵀ is what makes that safe. The network's raw outputs are a quaternion (normalized into R) and three scales (passed through exp or softplus into S). Any output the regression head can produce gives a valid symmetric PSD Σ, so the head never needs a constraint, and the loss never has to know about the PSD cone. A raw 3×3 regression head would regularly emit non-PSD garbage. This is the same factorization from your last entry, but the feed-forward case shows why it matters: the eigenvector/eigenvalue split is what lets an unconstrained network produce a constrained object. (I'm describing the standard 3DGS parameterization here; I haven't checked that these specific papers use it unchanged.)

Next track (optimization) connects here too: per-scene 3DGS runs gradient descent on millions of parameters for every scene, while feed-forward methods amortize that optimization into one trained network. Once you've covered Adam and SGD, "what did the network learn to do in one pass that Adam does in 30k steps?" is a good question.

---

### Threads worth pulling

1. **Fixed-budget scene state**: GlobalSplat's fixed Gaussian budget versus the variable splat counts of per-scene 3DGS. What's the right capacity for a scene, and is a fixed latent plus decoded Gaussians what a "simulator state" should look like? It links to Rolls' fixed attractor network from the last entry.

2. **Testing the Genie 4 "physics" claim**: what would a minimal test look like that separates learned dynamics from plausible video (e.g. does a thrown object's trajectory stay consistent when the camera orbits away and back)? That's Li's simulator contract turned into an experiment.

---

### Papers added to covered list

- Google DeepMind, "Genie 4," research preview, Oct 27 2026
- Runway, "Introducing GWM Worlds 2," Sep 3 2026
- GlobalSplat, arXiv:2604.15284, Apr 2026
- AnchorSplat, arXiv:2604.07053, Apr 2026
- WildSplat, arXiv:2607.05347, ECCV 2026
- NeuroHex, arXiv:2603.00376, 2026

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
