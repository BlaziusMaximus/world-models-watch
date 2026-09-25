# World Models Watch — Digest

---

## 2026-09-25

*First real update since the June 11 bootstrap — roughly 3.5 months of movement. Five items, chosen because each one either tests the simulator claim directly or does real construction work toward it.*

---

### What landed since June

**"From Generation to Simulation: How Far Are World Models from Being True Simulators?" — Wang et al., arXiv, Aug 24 2026**
[arXiv](https://arxiv.org/abs/2608.23070) | [HTML](https://arxiv.org/html/2608.23070.pdf)
This is the paper Li's June taxonomy was waiting for: instead of arguing definitions, it takes her renderer/simulator/planner split and operationalizes it into eight measurable capabilities a traditional simulator has — asset construction, physics engine, interaction, controllability, stability, state feedback, diversity, evaluation metrics — then maps 200 world-model papers (2018–June 2026) onto them. Verdict: strong on interaction and controllability, still short on *formal guarantees* of physical law. That's the precise gap the simulator framing cares about — not "does it look right" but "does it provably not break Newton's laws," and the answer right now is no. Worth treating as the field's current scorecard.

**Wayve GAIA-4 — Aug 2026**
[Wayve](https://wayve.ai/thinking/gaia-4/)
Puts Wayve's own AI Driver *inside* the generative loop: the world model renders video+radar from recorded sensor data, the driving policy acts on it, and its actions change what the model generates next — a real closed loop, not scripted playback. Supports world-on-rails replay, reactive agents, and counterfactual scenario generation for safety validation. This is a simulator being used for its actual contractual job (does the policy behave safely under distribution shift) rather than for pretty video — closer to the eight-capability bar above than most.

**World Labs acquires SceniX — July 21, 2026**
[World Labs](https://www.worldlabs.ai/blog/scenix) | [Coverage](https://techstartups.com/2026/07/21/world-labs-acquires-scenix-to-bridge-generative-world-models-and-physical-robotics-for-embodied-ai/)
SceniX (cofounders Yunzhu Li, Changxi Zheng) built a "real-to-sim-to-real" engine that turns one recorded physical task into many controllable, reusable training worlds for robots. Folding that into World Labs alongside Marble is a direct bet that the simulator role is validated by whether a policy trained inside it survives contact with a real robot — the most unforgiving version of the physics contract there is, since a bad simulator gets discovered by a robot falling over rather than by a benchmark score.

**"Scene-Level Heterogeneous Physics Simulation with 3D Gaussian Splats" — arXiv, June 19 2026**
[arXiv](https://arxiv.org/abs/2606.21753) | [Project page](https://visual-ai.github.io/raf/)
Names the exact gap CONTEXT.md tracks: 3DGS scenes look photoreal but production physics engines don't understand the representation, and prior physics-for-splats work was object-centric and isolated. This framework lets deformable 3DGS assets two-way-interact with fluids, meshes, and large captured static environments in one multi-solver simulation — splats colliding with, and being deformed by, conventional CG physics. It's an actual bridge from renderer-grade representation to simulator-grade interactivity, not just a proposal for one.

**"The Role of Grid Cells in Reducing Spatial Aliasing in Hippocampal Place Representations" — Johnson, Ghizawi, Minai (U. Cincinnati), arXiv, Aug 19 2026**
[arXiv](https://arxiv.org/abs/2608.18569)
Boundary-vector-cell-only place codes alias: symmetric or repetitive environments (a long hallway, a grid of identical rooms) produce near-identical representations for genuinely different locations. Adding multiple grid-cell modules — each a periodic signal at a different spatial scale, generated independently of the environment's geometry — cuts that aliasing by 94–99% across three test environments. The brain's fix for "my representation collapses distinct states onto the same code" is to add more independent, differently-scaled channels until the mapping becomes injective again.

---

### The LEARNING.md connection

You're mid-way through eigendecomposition and just wrote down the geometric reading: eigenvectors are directions that only stretch, never rotate. The grid cell paper above is the same idea wearing a different coat.

A single grid-cell module is a periodic function over space — a sine wave at a fixed spatial frequency. Multiple modules at different scales are multiple frequency channels stacked together. That's not a coincidence of vocabulary: **the Fourier basis *is* an eigenbasis** — specifically, the eigenvectors of a circulant (shift-invariant) matrix are exactly the Fourier modes. When you use only a coarse, low-frequency place code (few "eigenvectors," in this framing), points that differ only along the discarded high-frequency directions become indistinguishable — that's aliasing, and it's the exact same failure mode as a truncated SVD: keep only the top-k singular directions and any two inputs that differ solely along a discarded direction get mapped to the same low-rank approximation. Grid cells "fix" hippocampal aliasing the way adding back singular vectors fixes a truncated SVD — not by increasing precision within a channel, but by increasing the *rank* of the representation until the map from world-state to internal-code stops losing information, i.e. until it has a trivial null space.

So next time you compute a rank-deficient matrix's null space, or truncate an SVD and watch the reconstruction error go up: that's the same mechanism the brain uses to avoid confusing two different hallways. Eigendecomposition isn't just about *how* to shape a Gaussian splat (last entry's connection) — it's also the formal language for *when a representation is even capable of telling two things apart.*

---

### Threads worth pulling

1. **The eight-capability scorecard vs. the field's actual systems.** The Aug 24 "true simulators" paper gives a concrete rubric — asset construction, physics engine, interaction, controllability, stability, state feedback, diversity, evaluation metrics. Worth a session actually scoring Cosmos 3, GAIA-4, and Marble against those eight axes by hand, rather than taking anyone's marketing claim about being a "world model" at face value.

2. **Two different bets on how a simulator earns trust.** World Labs+SceniX validates the physics contract by putting a real robot in the loop (it either survives contact with the world or it doesn't). Wayve's GAIA-4 validates it by putting its own driving policy in a closed generative loop and watching for safety failures. Both are more rigorous than "does the video look plausible" — worth comparing which failure mode each actually catches, and which it can't.

---

### Papers added to covered list

- Wang T, Deng H, Yang M, He Y, Kuang X, Zhao G, "From Generation to Simulation: How Far Are World Models from Being True Simulators?," arXiv:2608.23070, Aug 24 2026
- Wayve, "GAIA-4: Multimodal World Models Powering Closed-Loop Simulation for Safe and Scalable Autonomy," Aug 2026
- World Labs, "World Labs Acquires SceniX," blog, July 21 2026
- "Scene-Level Heterogeneous Physics Simulation with 3D Gaussian Splats," arXiv:2606.21753, June 19 2026
- Johnson A, Ghizawi O, Minai AA, "The Role of Grid Cells in Reducing Spatial Aliasing in Hippocampal Place Representations," arXiv:2608.18569, Aug 19 2026

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
