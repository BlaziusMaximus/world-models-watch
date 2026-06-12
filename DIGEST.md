# World Models Watch — Digest

---

## 2026-06-12

### What landed today

**Mirage: Latent Spatial Memory for Video World Models** (Microsoft Research, arXiv:2606.09828, June 8)
Earlier video world models that care about spatial consistency maintain a point cloud in RGB space, then rasterize it and re-encode it each time a new frame needs to be generated — a round trip that is expensive and lossy. Mirage skips the RGB step: it lifts tokens directly into 3D using depth-guided back-projection, stores them as a persistent cache in the diffusion latent space, and queries that cache by warping latents to the new viewpoint before decoding. One latent-resolution projection instead of a full render-and-encode cycle. This is a concrete architectural answer to the spatial memory problem in video world models — not just "attend to past frames" but "maintain a 3D register in the model's own representational space." Worth watching for how well the geometry holds under aggressive viewpoint changes: that's where simulator-grade claims get stress-tested.

**Genie 3 × Google Street View** (Google DeepMind, May 19)
DeepMind plugged Genie 3's 11B-parameter world model into Google's 280-billion-image Street View archive, so you can now pick a real location, drop a character in, and explore. The framing is robotics and AV agent training: real-world geometry provides physical grounding that purely synthetic worlds lack. The telling detail in coverage: simulated characters pass through obstacles. Real geometry got in; physics did not. This is the renderer/simulator gap made literal and public — Genie 3 is a very capable renderer, but the moment you ask whether the agent can't walk through a wall, it fails the simulator's contract. The Street View grounding makes the failure more visible, not less.

**Echo-Memory: A Controlled Study of Memory in Action World Models** (arXiv:2606.09803, June 2026)
A clean ablation of four memory mechanisms in action-conditioned video world models, all run on the same Wan video backbone: Context (past frames in context), Compression (learned summary token), Spatial (explicit 3D cache), and State-Space (structured hidden state like an SSM). Spatial memory and State-Space memory win on long-horizon scene consistency; Context memory degrades fastest. The cleanness of the setup matters — most papers ship one memory type and claim it works; this one isolates the variable. The upshot for the simulator framing: explicit 3D structure (Spatial) is genuinely better at maintaining scene state over time, but learned hidden states (State-Space) come close and are cheaper to query.

---

### The LEARNING.md connection

You're working through SVD — here's where it sits in the pipeline you've been building up.

The previous entry showed that **each Gaussian's shape** is encoded as Σ = RSS^TR^T — eigendecomposition applied to a PSD matrix so gradient descent can roam free. But before any Gaussian gets placed, you need to know *where cameras are* in 3D space. That comes from Structure from Motion (SfM), and SVD is the engine that runs it.

Here's the specific step. Given 2D keypoint correspondences across images, you build the system Af = 0, where A is an n×9 matrix assembled from pairs of matching points and f is the vectorized 3×3 fundamental matrix F. The solution — the f that minimizes ‖Af‖ subject to ‖f‖ = 1 — is the **last column of V** in the SVD A = UΣV^T. This is the workhorse of linear algebra for null-space problems: when you want the vector that a matrix sends closest to zero, you take the singular vector corresponding to the smallest singular value.

Then, to extract camera rotation from the essential matrix E, you decompose E = UΣV^T again and form candidate rotations as R = UWV^T and R = UW^TV^T (where W is a fixed 90° rotation). The R you extract here is exactly the same orthogonal rotation matrix that appears in Σ = RSS^TR^T from the Gaussian splatting entry.

So the decomposition chain is: **SVD recovers camera geometry → eigendecomposition parameterizes Gaussian shape**. They're the same family of matrix factorizations, applied at the front and back of the same reconstruction pipeline. When you practice solving Ax = 0 via SVD, you're practicing the step that generates the 3D point cloud that initializes every Gaussian splatting scene.

---

### Threads worth pulling

1. **Where should 3D scene memory live?** Mirage says: in the latent space of the diffusion model. Prior work said: in an explicit RGB point cloud. The debate is about representational fidelity vs. information richness. A deeper conversation here would map the tradeoffs — latent-space memory is cheaper and integrates seamlessly with the generative model, but it's harder to inspect and correct. There's a design question lurking about what the "right" inductive bias for a simulator's scene memory actually is.

2. **Genie 3's physics gap as a benchmark target**: the "characters walk through walls" observation is informal, but it points at a real gap. What would a physics-aware constraint look like inside a video diffusion backbone? Collision avoidance is a hard constraint; gravity is a soft prior. A conversation worth having: is physics best injected as a training loss, as a structured latent (like occupancy grids in the video diffusion latent space), or as a separate physics simulator the model is distilled against?

---

### Papers added to covered list

- Wang W et al., "Latent Spatial Memory for Video World Models," Microsoft Research, arXiv:2606.09828, June 2026
- Google DeepMind, "Genie 3 × Street View: Simulate real-world places with Project Genie," May 2026
- Echo-Memory: "A Controlled Study of Memory in Action World Models," arXiv:2606.09803, June 2026

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
