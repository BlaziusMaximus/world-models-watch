# World Models Watch — Digest

---

## 2026-06-26

### What landed this week

**PhyWorld: Physics-Faithful World Model for Video Generation (May 2026)**
[arXiv:2605.19242](https://arxiv.org/abs/2605.19242)
Two-stage post-training on a video generation backbone: first, flow-matching fine-tuning for temporal coherence; second, Direct Preference Optimization (DPO) over physics preference pairs — generated video continuations ranked by whether they respect basic physical laws. The training paradigm is borrowed directly from LLM alignment. Matters here because it's the clearest attempt so far to actually close the gap between "video that looks physically plausible" and "video that *is* physically consistent" — the exact distinction separating a renderer from a simulator. They also introduce a per-law physical faithfulness benchmark so the improvement is measurable rather than vibes. If this line of work matures, you'd have a training recipe, not just an architecture bet, for the physics side of the simulator contract.

**PAIWorld: A 3D-Consistent World Foundation Model for Robotic Manipulation (June 16, 2026)**
[arXiv:2606.18375](https://arxiv.org/abs/2606.18375)
Points at a concrete flaw in current world foundation models: they're predominantly single-view. PAIWorld targets multi-view 3D consistency specifically for robotic manipulation — it generates scenes that stay coherent from multiple camera positions simultaneously. This is striking because multi-view 3D consistency is *precisely* the first clause of Fei-Fei Li's simulator contract (geometric consistency under novel viewpoints). The fact that single-view failure is still a live research problem in June 2026 is a useful calibration: the renderer/simulator gap isn't rhetorical, it's structural. Robots provide ground-truth physical feedback, making this a sharper testbed than pure video benchmarks.

**Faster 3DGS Convergence via Structure-Aware Densification (April 2026)**
[arXiv:2604.28016](https://arxiv.org/abs/2604.28016)
Fixes a subtle flaw in 3D Gaussian Splatting training. The standard adaptive density control decides where to add new Gaussians by looking at positional gradients — but those gradients conflate two different problems: a Gaussian that's in the wrong place (geometry error) and a Gaussian that's the right size for geometry but too coarse for a high-frequency texture (aliasing error). The fix uses structure tensors — symmetric matrices built from image gradient outer products — to classify which regime each Gaussian is in. A Gaussian gets split when structure tensor eigenvalues say the local texture is too fine for the Gaussian's projected size. Simpler densification rule, smarter signal: same Gaussian budget, less blur. See the LEARNING.md section below for why this is the closest paper yet to what you're currently studying.

**Place Cells as Multi-Scale Position Embeddings (May 2026)**
[arXiv:2505.14806](https://arxiv.org/abs/2505.14806)
UCLA / UT Southwestern team derives hippocampal place cell activity from the spectral decomposition of random walk transition kernels on the environment's spatial graph. A random walk spreading from position x at "scale" t diffuses to neighboring positions — the probability distribution is a superposition of eigenfunctions of the walk operator. Place cells *are* those eigenfunctions; each eigenvalue selects a different spatial scale. The biological dorsoventral gradient (place fields grow larger from dorsal hippocampus to ventral) maps directly to the spectrum of slower-decaying eigenvectors. The attractive part: you derive the geometry of biological spatial memory from a single mathematical object (a transition matrix), not from hand-designed circuits. For world model thinking, this suggests a world model whose state space is organized by spectral scale would naturally recover something that looks like the hippocampal spatial representation — not as a design choice but as a consequence of the math.

---

### The LEARNING.md connection

The structure-aware densification paper is a direct hit on what you're currently studying.

The **structure tensor** at a pixel is M = Σ ∇I ∇Iᵀ — sum of outer products of the image gradient vector over a small neighborhood. This is a 2×2 symmetric matrix. Because it's a sum of outer products, each of which is PSD by construction (xᵀ(vvᵀ)x = (vᵀx)² ≥ 0), M is guaranteed PSD — no projection step needed, just like the covariance factorization in 3DGS from last week's entry. The two eigenvalues of M are the ones that matter: both large → corner or texture; one large, one near zero → edge; both near zero → flat region. The paper uses this eigenvalue classification to decide whether a Gaussian is projecting onto a region that *requires* more primitives. It's the "geometric reading: directions that only stretch" from your study notes, running live inside the training loop as a decision gate.

What's sharp about this connection: last entry showed eigendecomposition appearing in the *representation* of Gaussians (how each primitive stores its shape). This paper shows eigendecomposition appearing in the *training dynamics* of 3DGS (when to add new primitives). The same concept doing two different jobs in the same algorithm.

---

### Threads worth pulling

1. **DPO for physics.** PhyWorld borrows preference optimization from LLM alignment and applies it to physical plausibility. The specific question worth a deeper session: what goes into a "physics preference pair," who generates them, what laws are actually being tested, and how hard is it to construct pairs that don't accidentally teach the model to game the evaluator rather than learn real physics? The per-law benchmarking approach they introduce is also worth examining — if you can score a model on "does it conserve momentum" independently of "does it look good," you have a real simulator metric.

2. **Spectral scale as a unifying thread.** The place cells paper and the structure tensor paper both decompose a space by scale — one via random walk eigenspectra, one via Laplacian scale space on image gradients. This probably isn't coincidence. Scale-decomposed representations appear in wavelets, NeRF positional encoding, and now biological spatial memory. If you're interested in what the right *internal representation* looks like for a world model, the question of "what decomposition of state space is most efficient for prediction" points here. The answer in biology is: eigenvectors of the transition operator, sorted by eigenvalue magnitude.

---

### Papers added to covered list

- Zhao M et al., "Place Cells as Multi-Scale Position Embeddings: Random Walk Transition Kernels for Path Planning," arXiv:2505.14806, May 2026
- Lyu L et al., "Faster 3D Gaussian Splatting Convergence via Structure-Aware Densification," arXiv:2604.28016, April 2026
- PhyWorld team, "PhyWorld: Physics-Faithful World Model for Video Generation," arXiv:2605.19242, May 2026
- PAIWorld team, "PAIWorld: A 3D-Consistent World Foundation Model for Robotic Manipulation," arXiv:2606.18375, June 2026

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
