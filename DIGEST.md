# World Models Watch — Digest

---

## 2026-06-19

### What landed this week

**ABot-Earth 0.5: Generative 3D Earth Model** (Amap CV Lab, arXiv:2606.09967, June 2026)
Satellite imagery → navigable 3D Gaussian splat world, at planetary scale. The model ingests geospatially tagged satellite photos and generates 3DGS scenes at under 10 minutes per square kilometer, with hierarchical level-of-detail so the result loads in a web browser. Currently covers 300+ cities across 190+ countries. Why it matters for the simulator direction: this is the first public demonstration of 3DGS being used not as a reconstruction tool (scan a room, get a splat) but as a *generative* tool — the model hallucinates geometry and texture for any patch of Earth it hasn't explicitly seen. The physics fidelity question is wide open, but the scale and the Gaussian-native output format are exactly what an embodied AI training substrate needs.

**Hippocampal Place Cells Map Terrain Geometry Independently of Behavior** (Science Advances, 2026, doi:10.1126/sciadv.adz9893)
Rats navigated ridged terrain while researchers recorded hippocampal place cells. Key finding: place fields elongated *parallel to terrain contours* and a third repeated on each ridge — tracking the surface shape, not the animal's movement path. A boundary vector cell model reproduced the pattern; a predictive coding model didn't. This is a small paper with a large implication: the brain's spatial map appears to be surface-bound, not volumetric. For the AI world-model question, that's a pointed hint about inductive bias — 3DGS uses volumetric Gaussians, but the biological cognitive map might be better modeled as a surface mesh or 2.5D representation rather than full 3D volume.

**PhyWorld: Physics-Faithful World Model for Video Generation** (arXiv:2605.19242, May 2026)
Two-stage post-training on a video diffusion model: first, fine-tune for temporal coherence with flow matching; second, apply Direct Preference Optimization (DPO) using "physics preference pairs" — generated clips scored against a taxonomy of physical laws (collisions, fluids, shadows, ballistics, rolling/sliding). Result: average physics-faithfulness score climbs from 2.99 to 3.09 vs. best baselines, on a 250-prompt benchmark organized by law type. Not a revolution, but it's the most systematic attempt to date to actually *measure* how much a video model respects each specific physical law — and the answer is "not much yet, but DPO helps." The benchmark alone (Physics-IQ) is worth knowing about.

**YoCausal: How Far Is Video Generation from a World Model? A Causality Perspective** (arXiv:2605.30346, May 2026)
Elegant benchmark: reverse real-world videos, then ask whether a video diffusion model can tell the forward from the reversed version. Two metrics: RSI (Reverse Surprise Index, does the model assign lower loss to forward-time?) and CCI (Causality Cognition Index, can it distinguish causally valid from non-causal clips?). They tested 13 state-of-the-art models and found that perceiving the arrow of time does not imply understanding causality — models can tell "this clip runs forward" without knowing *why* the ball goes down and not up. The gap to human causal cognition is large. This is essentially a formal argument that current video generation sits firmly in Li's "renderer" box, not "simulator."

---

### The LEARNING.md connection

You're finishing up eigendecomposition and about to move into optimization. Here's where **constrained optimization and Lagrange multipliers** show up in one of today's papers before you've even opened that chapter.

PhyWorld's second training stage uses DPO (Direct Preference Optimization) to enforce physical plausibility. DPO can be derived from a constrained optimization problem: *maximize* the probability the model assigns to physics-plausible clips while *constraining* the model to stay close to the pretrained distribution (measured by KL divergence). The solution to that constrained problem has a closed form — and the β coefficient that appears in the DPO loss is exactly the Lagrange multiplier enforcing the KL constraint. When you tune β, you're deciding how hard to pull the model toward physics versus how much to let it drift from what it already knows. That tradeoff between objective and constraint, with a multiplier controlling the strength of the leash, is Lagrange's method applied to a loss landscape over billions of parameters. The next section of your study track isn't a detour from physics-faithful world models — it's the math that generates them.

---

### Threads worth pulling

1. **The volumetric vs. surface-bound representation question**: The terrain place cells paper says brains use a surface-bound map. ABot-Earth uses 3DGS (volumetric Gaussians). These are in tension, and neither is obviously right for an AI world model. Worth a conversation about what "surface-bound" would even mean as an architectural prior, and whether something like mesh-based neural rendering gets closer to the biological inductive bias.

2. **YoCausal's arrow-of-time test as a simulator litmus**: The benchmark is elegant enough that you could apply it to any world model system — not just video generators. Marble, Cosmos 3, the Waymo model: do any of them actually pass the causality test, or are they all renderers wearing a simulator's hat? The benchmark paper gives you a concrete tool for interrogating the systems you're already tracking.

---

### Papers added to covered list

- Amap CV Lab, "ABot-Earth 0.5: Generative 3D Earth Model," arXiv:2606.09967, June 2026
- "Hippocampal Place Cells Map Terrain Geometry Independently of Behavior," *Science Advances*, 2026, doi:10.1126/sciadv.adz9893
- "PhyWorld: Physics-Faithful World Model for Video Generation," arXiv:2605.19242, May 2026
- "YoCausal: How Far Is Video Generation from World Model? A Causality Perspective," arXiv:2605.30346, May 2026

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
