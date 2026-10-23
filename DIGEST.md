# World Models Watch — Digest

---

## 2026-10-23

*Thin week on confirmed primary sources (my fetch tool couldn't reach arXiv/vendor pages directly, so summaries below come from abstracts and search snippets — vendor numbers unverified).*

---

### What landed

**NVIDIA Cosmos-H-Dreams — real-time, action-conditioned surgical world model** (arXiv 2608.24199; checkpoint/code release reported ~Oct 20)
[arXiv](https://arxiv.org/pdf/2608.24199) | [HF blog](https://huggingface.co/blog/nvidia/cosmos-h-dreams) | [GitHub](https://github.com/isaac-for-healthcare/Cosmos-H-Dreams)
Cosmos-Predict2.5-2B post-trained on robot-surgery data with a 44-D unified action vector, then distilled via Self Forcing into a causal few-step student (~2 denoising steps/latent frame; claimed ~160 FPS on one RTX PRO 6000). Why it matters: it's the clearest "world model as policy-evaluation simulator" instance since Waymo WM — action in, next frames out, fast enough to close a loop. But it's still a video simulator: no explicit 3D state, so Li's geometric-consistency contract is satisfied only to the extent the pixels happen to behave. Release dates/names conflict across sources (Cosmos-H vs Cosmos3-H); treat as unsettled.

**"Is Gaussian Splatting Becoming Neural Again?" (arXiv 2609.12395, Sept 2026)**
[arXiv](https://arxiv.org/pdf/2609.12395)
Taxonomy of 19 methods that use networks to generate/share Gaussian parameters (attribute decoding, spatial sharing, view-conditioned decoding, topology generation, amortized inference), plus a controlled mip-NeRF 360 study. Finding: sharing appearance and opacity helps; *decoding geometry gives no further gain*. Conclusion: "selective neuralization" — share what's reusable, keep geometry explicit and local. For a simulator-centric view this is a useful data point: the explicit-vs-implicit question isn't binary, and geometry is precisely the part that seems to want to stay explicit.

**"Grounded world models in biological organisms and future embodied AI" (arXiv 2607.13560)**
[arXiv](https://arxiv.org/pdf/2607.13560)
Argues biological neural dynamics (place/grid cells) become aligned with world structure through *embodied action*, contrasting this with generative models trained passively on video. I only saw the intro and figure captions, so I can't vouch for specifics — but it's a direct statement of the gap between video prediction and simulation from the neuroscience side, and a good bridge to the Rolls / unified-representation items from June.

*Skimmed, not covered:* Odyssey-3 (closed-weights real-time interactive world model, self-reported claims); S2GS (ICML 2026, streaming semantic Gaussian field with instance memory, 1000+ frames) — worth a look if online scene memory interests you.

---

### The LEARNING.md connection

Eigendecomposition again, but from the opposite direction. In June we saw Σ = R S Sᵀ Rᵀ as the trick that lets gradient descent stay inside the PSD cone. The "Becoming Neural Again" study asks which of a Gaussian's attributes a network should *predict* versus leave as free per-primitive parameters. Its answer — shared decoding is fine for appearance/opacity, useless for geometry — is about exactly the (R, S, position) block. Geometry parameters are intrinsically local: each Gaussian's eigenvectors R and eigenvalues S encode the local surface orientation and extent, which differ from point to point, so there's little shared structure for a decoder to compress. A decent mental model: R and S are a per-point *local frame*; a network can predict colour from context, but a local frame is already the most compact description of local geometry. Also note the non-uniqueness problem flagged in a neighbouring paper (2509.22917): (R, S) → Σ is many-to-one (sign flips, permutations of eigenvectors), which is exactly the eigendecomposition ambiguity you've been meeting in the linear algebra track, and why feeding raw Gaussian parameters into a network is awkward.

---

### Threads worth pulling

1. **Why does geometry resist amortization?** Test the intuition above: is it because local frames are high-entropy, or because photometric loss gives weak gradients to geometry through a decoder? Connects to optimization (up next) — gradient flow through R and S.
2. **Video simulators with no 3D state vs. explicit-geometry simulators:** Cosmos-H-Dreams is fast and action-conditioned; Marble-style systems hold geometry. What would a hybrid look like — a distilled causal video model conditioned on a Gaussian scene state?

---

### Papers added to covered list

- NVIDIA, "Cosmos-H-Dreams: Real-Time Generative Physics Simulation for Surgical Robotics," arXiv 2608.24199 (release ~Oct 2026)
- "Is Gaussian Splatting Becoming Neural Again? A Taxonomy and Controlled Study of Learned Parameterization," arXiv 2609.12395
- "Grounded world models in biological organisms and future embodied AI," arXiv 2607.13560
- Skimmed: Odyssey-3 (Odyssey, Sept/Oct 2026); S2GS, arXiv 2603.14232; "Learning Unified Representation of 3D Gaussian Splatting," arXiv 2509.22917

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
