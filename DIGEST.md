# World Models Watch — Digest

---

## 2026-07-24

### What landed this week

**"A Definition and Roadmap for World Models" — Physical Intelligence Team, Shanghai AI Laboratory (July 7, 2026)**
[arXiv:2607.06401](https://arxiv.org/abs/2607.06401)
A perspective paper that tries to end the "everyone calls something different a world model" problem: it gives a scientific definition — world models as internal simulators that learn the structure and dynamics of an environment — plus a staged roadmap for building them, and argues static, pre-trained world models can't hold up against real physical variation; you need architectures that adapt a universal representation to local physics in real time. This lands right next to Fei-Fei Li's June 3 taxonomy (renderer/simulator/planner) as the second major framing document of 2026, but from a different institutional angle — worth reading the two side by side rather than as duplicates.

**World Labs acquires SceniX, publishes "Advancing Spatial Intelligence for Robotics" (July 21, 2026)**
[World Labs blog](https://www.worldlabs.ai/blog/scenix)
World Labs folds robotics-focused SceniX into the company and stakes out robotics as where spatial intelligence "becomes physical" — a robot has to perceive space, understand how objects move and interact, anticipate consequences, and act reliably, which is the simulator contract restated as an operational requirement rather than a research goal. Notable because it's a strategic signal, not just a paper: the company built around the renderer/simulator/planner taxonomy is now explicitly betting the simulator piece matters most where the stakes are physical, not just visual.

**MRO-GWM: Learning Action-Conditional and Object-Centric Gaussian Splatting World Models for Rigid Objects — Kreber, Mack & Stueckler, U. Augsburg (June 2026, ICLR 2026)**
[arXiv:2606.01950](https://arxiv.org/abs/2606.01950)
Represents a multi-object scene as per-object Gaussians in a canonical frame, then uses a spatio-temporal transformer to predict future rigid-body motion from a history of Gaussians plus robot actions — tested on non-prehensile manipulation via model-predictive control. This is a clean, small-scale instance of the thing the whole field is reaching for: scene representation (Gaussian splatting) plus learned, action-conditioned dynamics (physics), evaluated by whether it actually helps a controller act. Directly on your "where does 3D structure get explicitly represented vs. stay implicit" question — here it's explicit, per-object, and canonical.

**Scene-Level Heterogeneous Physics Simulation with 3D Gaussian Splats — Liu, Wu & Han (June 2026, CVPR Findings)**
[arXiv:2606.21753](https://arxiv.org/abs/2606.21753) · [project page](https://visual-ai.github.io/raf/)
Prior physics-for-3DGS work could only simulate one splatted object in isolation; this bridges splats into full multi-solver physics pipelines so deformable 3DGS assets interact two-way with fluids, meshes, and large captured static environments in the same scene. Different strategy than MRO-GWM: instead of learning dynamics end-to-end from data, it plugs splats into existing production physics engines. Two competing routes to the same simulator contract — learn the physics, or reuse physics engines built for meshes.

**NVIDIA launches Cosmos 3 Edge (July 15, 2026)**
[MarkTechPost](https://www.marktechpost.com/2026/07/21/nvidia-releases-cosmos-3-edge-a-4b-parameter-open-world-model-that-reasons-and-generates-robot-actions-on-device/) · [SiliconANGLE](https://siliconangle.com/2026/07/16/nvidia-launches-cosmos-3-edge-model-expands-physical-ai-push-japan/)
The third tier of the Cosmos 3 family (already covered here as of June 1) — a 4B-parameter model trained from scratch, running on a single GPU, generating robot action policies at 15 Hz on Jetson Thor. Mostly an infrastructure/scaling note rather than a new idea: the physics-accuracy bet from Cosmos 3 is now small enough to run on-device, which matters for how fast robotics teams can iterate against a simulator rather than the real world.

---

### The LEARNING.md connection

Last entry connected eigendecomposition to how Gaussian splatting *shapes* a primitive (Σ = RSSᵀRᵀ). MRO-GWM gives a different, complementary angle on the same decomposition list: it describes object *motion* as a rigid-body transformation applied to Gaussians in a canonical frame — meaning the network has to predict a valid rotation matrix R ∈ SO(3) at every timestep. A neural network's raw output is just an unconstrained matrix; nothing stops gradient descent from producing something that isn't orthogonal, and a non-orthogonal "rotation" would shear and distort the object instead of just turning it. One standard fix is to QR-decompose the raw output and keep only Q — QR decomposition is literally Gram-Schmidt orthogonalization run on the matrix's columns, so it hands you back the nearest valid rotation for free. That's the exact "why do you factor a matrix" question your notes pose: not for insight this time, but because factoring is how you get an unconstrained optimizer to respect a hard geometric constraint (orthogonality) without a projection step. Same move as last time's PSD-cone trick, one level up — first eigendecomposition kept Σ inside the PSD cone, now QR keeps R inside SO(3).

---

### Threads worth pulling

1. **Two roadmaps, one summer**: Fei-Fei Li's renderer/simulator/planner taxonomy (June 3) and Shanghai AI Lab's "Definition and Roadmap" (July 7) both try to pin down what a world model *is*, from different institutional vantage points (a product company vs. a research lab). Worth a session diffing them — where do they agree on what counts as "solved," and where does one see a bar the other doesn't even mention?

2. **Learn the physics vs. borrow the physics engine**: MRO-GWM predicts rigid-body dynamics end-to-end from data; the Scene-Level Heterogeneous paper instead bridges splats into existing multi-solver physics pipelines built for meshes and fluids. These are two structurally different bets on how splatting closes the gap to Li's simulator contract — one scales with data, the other scales with decades of physics-engine engineering. Worth digging into which one is winning more attention going into robotics deployment.

---

### Papers added to covered list

- Physical Intelligence Team (Shanghai AI Laboratory), "A Definition and Roadmap for World Models," arXiv:2607.06401, July 2026
- World Labs, "Advancing Spatial Intelligence for Robotics" (SceniX acquisition), July 21 2026
- Kreber, Mack & Stueckler, "Learning Action-Conditional and Object-Centric Gaussian Splatting World Models for Rigid Objects" (MRO-GWM), arXiv:2606.01950, ICLR 2026
- Liu, Wu & Han, "Scene-Level Heterogeneous Physics Simulation with 3D Gaussian Splats," arXiv:2606.21753, CVPR Findings 2026
- NVIDIA, "Cosmos 3 Edge," July 15 2026

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
