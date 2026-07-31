# World Models Watch — Digest

---

## 2026-07-31

### What landed since the last digest (seven weeks: June 11 → July 31)

**World Labs acquires SceniX (July 21, 2026)**
[World Labs blog](https://www.worldlabs.ai/blog/scenix) | [Coverage](https://www.techstartups.com/2026/07/21/world-labs-acquires-scenix-to-bridge-generative-world-models-and-physical-robotics-for-embodied-ai/)
SceniX built a "real-to-sim-to-real" (R2S2R) engine: it turns a real robot, environment, and interaction into a simulation, then multiplies one physical task into many controllable, reusable synthetic worlds for policy training. World Labs is folding this into Marble — which already outputs Gaussian splats *plus* collision meshes a physics engine can act on — so the generated 3D world isn't just something to look at, it's something a robot can be trained inside. This is Li's simulator contract made operational: the acquisition is explicitly about closing the loop from generated geometry to physically valid training data, which is the hardest, least-solved part of the taxonomy she published in June.

**NVIDIA ships Cosmos 3 Edge (July 15–21, 2026)**
[MarkTechPost](https://www.marktechpost.com/2026/07/21/nvidia-releases-cosmos-3-edge-a-4b-parameter-open-world-model-that-reasons-and-generates-robot-actions-on-device/) | [SiliconANGLE](https://siliconangle.com/2026/07/16/nvidia-launches-cosmos-3-edge-model-expands-physical-ai-push-japan/)
The third tier of the Cosmos 3 family (after 64B Super and 16B Nano, both shipped May 31): a 4B-parameter model that runs entirely on a single Jetson Thor or RTX GPU, no cloud round-trip. It does on-device vision reasoning *and* emits robot action tokens — 640×360, 32 actions/inference, 15 Hz — and can be adapted to a new robot or vehicle in about a day. Notable less for raw capability than for what it signals: the industry is now treating "world model that outputs actions in real time on-device" as a shippable product category, not a research demo. FANUC, Yaskawa, Toyota-backed Preferred Networks, and 20+ others have joined the Cosmos Coalition around it.

**PhysisForcing: Physics Reinforced World Simulator for Robotic Manipulation** (arXiv, June 26, 2026)
[arXiv:2606.28128](https://arxiv.org/abs/2606.28128)
A training-time, plug-and-play fix for a specific failure mode in video-generation world models: robot-object contact frequently produces discontinuous motion or objects that deform implausibly on contact. The fix supervises an intermediate diffusion-transformer feature with a pixel-level trajectory loss and a semantic-level relational loss, focused specifically on interaction-critical regions — zero extra inference cost since it's training-time only. Used as a world model under the WorldArena action-planner protocol, it raises closed-loop task success from 16.0% to 24.0%. This is a direct, measurable attack on the "dynamics that hold up over time" clause of the simulator contract — a concrete number for how far current video world models are from physically faithful, and how much a targeted physics loss buys back.

**Gaussian Point Splatting** (SIGGRAPH 2026 / ACM ToG, presented July 20, 2026)
[Project page](https://momentsingraphics.de/Siggraph2026.html) | [Paper](https://doi.org/10.1145/3811272) | [Code](https://github.com/JorisAR/gaussian-point-splatting)
Rijsdijk, Peters, Weinmann & Marroquim replace the standard tile-sort-blend Gaussian splatting renderer with a stochastic one: sample pixel-sized opaque points from each Gaussian's distribution, splat them into the framebuffer with 64-bit atomics, and let stochastic transparency substitute for sorting entirely. Result: 425M Gaussians rendered interactively on a single RTX 4070 Ti Super — an order of magnitude past what sort-based pipelines handle at that frame rate. Relevant to the scene-representation thread because it's evidence the Gaussian splatting rendering pipeline itself — not just scene optimization — is still being fundamentally rethought, three years after the original paper.

**Nalluru, Glitz, Przygodda, Mancini & Barron, "The hippocampus as a generative model"** (*Phil. Trans. R. Soc. B*, July 9, 2026)
[Royal Society](https://royalsocietypublishing.org/rstb/article/381/1954/20250247/482493/The-hippocampus-as-a-generative-model)
Argues the hippocampus should be understood not merely as a map-maker but as a *generative* model of the environment — one that supports sampling and prediction, not just localization — and reviews evidence for predictive-map-like coding (place cells indexing expected future states, not just current position). The framing overlaps directly with the renderer/simulator/planner split: this paper is making the case that biological "world models" already separate a generative/predictive component from a pure spatial-index component, which is exactly the distinction Li's taxonomy draws for artificial ones.

---

### The LEARNING.md connection

Different piece of the Gaussian splatting pipeline than last time — last digest connected eigendecomposition to how a Gaussian's *shape* is parameterized (Σ = RSS^ᵀR^ᵀ) so gradient descent can optimize it freely. Today's connection is about what happens one step later, at render time, and it's squarely inside the "how to guarantee PSD-ness" question you're studying.

Every Gaussian splatting renderer — including the new stochastic one above — has to take a 3D covariance matrix Σ (which is symmetric PSD by construction) and project it down to a 2D covariance Σ' on the screen, so it can figure out the pixel footprint of each splat. It does this with a *congruence transformation*: Σ' = J Σ Jᵀ, where J is the Jacobian of the camera projection at the Gaussian's center. This formula is doing real work for you: for *any* matrix J — square or not, full-rank or not — if Σ is PSD then JΣJᵀ is guaranteed PSD too. That's a theorem, not a numerical accident, and it's why the renderer never needs a projection-onto-the-PSD-cone step after this transform, unlike naive parameterizations that can drift out of the valid region. So the PSD guarantee you're studying shows up twice in the same pipeline, in two different roles: once at *optimization* time (RSSᵀRᵀ keeps gradient descent inside the cone) and once at *rendering* time (congruence transforms keep you inside the cone automatically when you change coordinate systems). Worth sitting with why congruence preserves PSD-ness the next time eigendecomposition comes up — it's the same quadratic-form argument (xᵀΣ'x = (Jᵀx)ᵀΣ(Jᵀx) ≥ 0) that explains why Σ = RSSᵀRᵀ works at all.

---

### Threads worth pulling

1. **SceniX + Marble as a real test of the simulator contract.** Li's taxonomy was mostly aspirational in June; the SceniX acquisition is the first move that looks like an actual engineering answer to "how do you get a generated 3D world to produce data a robot policy can trust." Worth watching whether World Labs publishes anything about *sim-to-real transfer rates* — that number, more than any rendering benchmark, is the real scorecard for the simulator framing.

2. **PhysisForcing's 16%→24% closed-loop number as a baseline.** That's a rare case of someone actually quantifying "how physically faithful is this world model" in task-success terms rather than pixel metrics. Worth tracking whether other groups adopt WorldArena's closed-loop protocol as a standard benchmark — if the field converges on a shared physics-fidelity number, that's the metric that would let you compare Cosmos 3, GAIA-3, and Genie 3 apples-to-apples on the simulator contract specifically, instead of on video quality.

---

### Papers added to covered list

- World Labs, "World Labs Acquires SceniX," blog post, July 21 2026
- NVIDIA, "Cosmos 3 Edge," launched July 15 2026
- "PhysisForcing: Physics Reinforced World Simulator for Robotic Manipulation," arXiv:2606.28128, June 26 2026
- Rijsdijk J, Peters C, Weinmann M, Marroquim R, "Gaussian Point Splatting," ACM ToG (Proc. SIGGRAPH) 45(4), July 2026
- Nalluru et al., "The hippocampus as a generative model," *Phil. Trans. R. Soc. B* 381(1954), July 9 2026

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
