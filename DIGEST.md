# World Models Watch — Digest

---

## 2026-07-10

### What landed this month

**"Hallucination in World Models Is Predictable and Preventable"** — Nicklas Hansen et al., arXiv, ~June 30 2026
[arXiv:2606.27326](https://arxiv.org/abs/2606.27326)
The most direct empirical hit on the simulator contract yet. The authors build MMBench2 (427 hours, 210 tasks, ground-truth actions/rewards/live simulators) and a 350M-parameter world model to study it, then split "hallucination" into three distinct failure modes: perceptual (the model literally misperceives), action-marginalized (it ignores the action it was conditioned on), and scene-diverging (it silently drifts from true dynamics while staying visually plausible). The finding that matters: these are predictable at runtime from data-coverage signals, and preventable by fixing training data distribution rather than redesigning architecture. This turns "does physics hold up over time" — Li's dynamics clause — from a vibe into a measured, fixable property. Directly useful as an eval methodology for judging whether any of this month's other releases actually satisfy the simulator contract or are just fluent video.

**AlayaWorld** — Shanda AI Research, arXiv, July 7 2026
[arXiv:2607.06291](https://arxiv.org/html/2607.06291v1)
Open-source interactive world model (fine-tuned from LTX-2.3) generating explorable, spatially coherent environments at 720p/24fps for 60+ seconds of continuous interaction — well past the point where most interactive world models drift or collapse. Long-horizon geometric coherence is the actual bottleneck standing between "cool demo" and "reusable simulator," so this is squarely aimed at the weak link, not the flashy part.

**Kairos** — a native world model stack for physical AI, arXiv, June 15 (updated July 3) 2026
[arXiv:2606.16533](https://arxiv.org/abs/2606.16533)
A 4B-parameter architecture fusing world understanding, generation, and action in one stack via hybrid linear attention — sliding-window for local dynamics, dilated windows for mid-range, gated linear attention for persistent long-horizon memory — tuned for real-time edge inference across robot embodiments. Reportedly beats 14B baselines on embodied benchmarks at a fraction of compute. Interesting as a live answer to a question Li's taxonomy leaves open: can renderer + simulator + planner live in one stack, or does persistent dynamic state need its own dedicated architecture? Kairos bets on the latter — the memory design exists specifically to hold state across time, which is the dynamics clause of the simulator contract, not the rendering clause.

**RAF: Scene-Level Heterogeneous Physics Simulation with 3D Gaussian Splats** — Xiaoyang Liu, Shangzhe Wu, Kai Han (HKU / Cambridge), arXiv, June 2026 (CVPR Findings 2026)
[arXiv:2606.21753](https://visual-ai.github.io/raf/)
Converts a whole captured scene — splats, meshes, fluids together, not one isolated object on an ideal plane — into a unified physics-particle set that a solver-agnostic physics kernel can act on, with collision geometry derived straight from the scene capture. This is the most concrete scene-level bridge so far from "3DGS renders it" to "3DGS is now a physics-actionable representation" — the geometry-to-physics step the whole splatting arc has been building toward.

**"Brain-AI convergence: Generative world models and hierarchical attention for human intelligence"** — Shogo Ohmae et al., *Patterns* (Cell Press), late June 2026
[ScienceDirect](https://www.sciencedirect.com/science/article/pii/S2666389926001029)
A perspective piece arguing neocortex, cerebellum, and modern transformer-based AI converge on four shared principles: building world models via prediction-error learning, reusing that model for both understanding and generation, hierarchical attention, and adaptive expert-switching. It's an explicit attempt to put "world model" as used in this digest and "world model" as used by neuroscientists on the same page — useful connective tissue, though its treatment of hippocampal/entorhinal spatial coding specifically is secondary to the broader cortex/cerebellum/LLM argument.

---

### The LEARNING.md connection

RAF is the one to sit with. Last entry's note was that a 3D Gaussian's shape lives in Σ = R S S^T R^T — eigenvectors (columns of R) give orientation, eigenvalues (from S) give the semi-axis lengths of the ellipsoid. RAF's job is to take that ellipsoid and turn it into physics-simulable *particles* — which means literally sampling points inside a region whose shape is defined by that same decomposition: start with points in a unit sphere, transform by R·S, and you land inside the correctly oriented, correctly scaled ellipsoid, ready to hand to a solver as point mass. That's eigendecomposition doing real work one layer downstream of where you left it — not just describing a shape, but seeding a simulation with it.

And it previews where you're headed next. Once those particles exist, RAF's physics kernel has to update them under contact and collision constraints — which is a constrained optimization problem, solved with exactly the machinery on your "Up Next" list (Lagrange multipliers, in particular — collision constraints are the textbook use case). The eigendecomposition you're studying now is what builds the geometry; the optimization you're about to study is what makes that geometry obey physics. RAF is the paper where those two halves of your syllabus meet.

---

### Threads worth pulling

1. **Use Hansen et al.'s failure taxonomy as a rubric.** Take AlayaWorld and Kairos — both make simulator-flavored claims (long-horizon coherence, persistent dynamic state) — and ask which of the three hallucination modes (perceptual / action-marginalized / scene-diverging) each is actually vulnerable to. That's a concrete way to pressure-test whether either satisfies Li's contract or just extends how long the video stays plausible before drifting.

2. **The neuroscience search came up thin this window** — the genuinely relevant cluster (a model where grid and place fields co-emerge unsupervised from next-observation prediction, arXiv:2605.21356; a hippocampal-entorhinal-inspired world model, arXiv:2605.15733; new CA3→CA1 coding and place-cell geometry papers in Nature/Science Advances) all landed in the two weeks just before this window opened. Worth a dedicated pass next digest — the co-emergence paper in particular, since "spatial cells emerge from a prediction objective" is about as on-thesis as neuroscience gets for this project.

---

### Papers added to covered list

- Hansen N. et al., "Hallucination in World Models Is Predictable and Preventable," arXiv:2606.27326, June 2026
- Shanda AI Research, "AlayaWorld," arXiv:2607.06291, July 2026
- "Kairos: A Native World Model Stack for Physical AI," arXiv:2606.16533, June 2026
- Liu X., Wu S., Han K., "RAF: Scene-Level Heterogeneous Physics Simulation with 3D Gaussian Splats," arXiv:2606.21753, June 2026 (CVPR Findings 2026)
- Ohmae S. et al., "Brain-AI Convergence: Generative World Models and Hierarchical Attention for Human Intelligence," *Patterns*, June 2026

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
