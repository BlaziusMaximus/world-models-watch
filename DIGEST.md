# World Models Watch — Digest

---

## 2026-11-13

*Five months since the last entry (2026-06-11), so this one covers a wider window and keeps only what moved the simulator question. Web fetch was blocked in this run, so primary pages (DeepMind's Genie 4 post, arXiv abstracts) weren't opened directly. Details come from search snippets and secondary coverage, and the uncertain ones are flagged.*

---

### The one-line version
The field is splitting the simulator into parts. Genie 4 puts a planner (Gemini) on top of a renderer, WorldParticle learns physics directly on particles, and the new benchmarks are finally testing whether worlds stay coherent past a 10-second clip.

---

### What landed

**DeepMind Genie 4, research preview (Oct 27, 2026)**
[the-decoder](https://the-decoder.com/google-deepminds-genie-4-puts-a-modern-spin-on-agent-designs-that-predate-llms/) · [Pebblous explainer](https://blog.pebblous.ai/blog/genie-4-world-model-robot-verification/en/) · [eesel explainer](https://www.eesel.ai/blog/genie-4)
A real-time world model with native audio and promptable mid-session events. A Gemini model plans, checks and corrects each frame, which DeepMind calls an early "hybrid world model." **Why it matters for you:** this is Li's renderer/simulator/planner split built as an actual system, but the part in the middle is still missing. Objects drift and "complex physics" is listed as unsolved. Memory is described as "minutes," up from Genie 3's roughly one minute, though sources disagree on the exact figure. Gemini is making up for the absence of a real simulator by checking frames after they're drawn.

**WorldParticle: one transformer for many physical materials (MIT/UBC; preprint May, MIT News Nov 9)**
[arXiv 2605.15305](https://arxiv.org/abs/2605.15305) · [project](https://worldparticle.github.io/WorldParticle_Web/) · [TechXplore](https://techxplore.com/news/2026-11-ai-simulates-behavior-substances-sand.html)
Learned Lagrangian simulation covering cloth, elastic solids, fluids, granular media and molecular dynamics. Each step makes a physics-based prediction from known forces, then a transformer corrects the remaining error. The transformer merges particle tokens into a smaller set of "super-tokens" (half as many at each level). It is one architecture, but a separate network is trained for each material category. **Why it matters:** this sits at the opposite end from Genie. The state is explicit, physics is a built-in prior, and nothing is rendered. Splat-physics papers (PhysGaussian and its successors) already treat Gaussians as material points. Pair those with a learned corrector like this one and you get a scene representation that renders and also simulates.

**WorldRoamBench: long-horizon stability benchmark (Jul 2026)**
[arXiv 2606.31672](https://arxiv.org/abs/2606.31672) · [HF papers](https://huggingface.co/papers/2606.31672)
600+ cases of 10–60s continuous WASD control. It scores four axes separately: per-frame action following, visual drift by segment, physics checks that only count when the model is actually following controls, and memory split into *scene* memory and *subject* memory. The headline finding: **rankings on short clips don't predict rankings on long rollouts**, and strong visuals don't mean good physics or memory. Genie 3 ranks near the top for first-person. **Why it matters:** it's the closest thing yet to an operational test of the simulator contract. The scene-memory vs subject-memory split also maps onto allocentric vs egocentric representation from the neuroscience side.

**PrimeSim: action-conditioned multiview robot simulator (Nov 3, 2026)**
[arXiv 2611.04870](https://arxiv.org/abs/2611.04870) · [GitHub](https://github.com/PrimebotTech/PrimeSim) · [HF](https://huggingface.co/qiyuan-robot/PrimeSim)
Turns a pretrained video model (Wan2.2 backbone) into a multi-camera simulator conditioned on actions. Trained on 30k hours of demonstrations from 22 robot platforms, with all action spaces mapped into one shared 31-dim layout. Runs at 10 FPS with open weights. **Why it matters:** the cross-embodiment action space is the interesting part. It's a bet that one learned dynamics model can sit underneath many different robot bodies, the way a physics engine does. Multiview conditioning gives it some geometric consistency, but only implicitly. There is no explicit 3D state.

**NVIDIA Cosmos 3: four open interactive-sim checkpoints (Nov 5, 2026)**
[HF blog](https://huggingface.co/blog/nvidia/cosmos-interactive-world-simulation) · [coverage](https://letsdatascience.com/news/nvidia-releases-four-open-cosmos-3-checkpoints-for-world-sim-36487992)
A follow-up to the Cosmos 3 launch covered in June. There are 16B post-trained Nano variants for streaming driving simulation (**Sim-Auto**, which outputs camera and LiDAR together), offline multi-camera generation (Transfer-Auto, up to 11 views), sim-to-photoreal translation, and bimanual robot control. **Why it matters:** generating LiDAR and camera together is a quiet geometric constraint. Sensors that measure depth directly are hard to fool with output that is plausible but inconsistent.

**World Labs opens Gaussian-splat creation in Marble (Oct 30, 2026)**
[Radiance Fields](https://radiancefields.com/world-labs-opens-gaussian-splat-creation-in-marble)
A Marble Studio feature plus a beta API task, labeled "Marble 2" in one source (unconfirmed). It turns posed RGBD views into refined SPZ splats for $5 a run. **Why it matters:** World Labs is making explicit Gaussians the format you get out of their world model, which is the explicit-representation bet. It's also today's hook into your linear algebra (below).

**Also noted:** Pezzulo et al., *Grounded world models in biological organisms and future embodied AI* ([arXiv 2607.13560](https://arxiv.org/abs/2607.13560)), a position paper arguing that interaction-grounded world models come first and language is attached afterward, with entorhinal–hippocampal navigation as the central example. StreamSplat ([arXiv 2608.01659](https://arxiv.org/abs/2608.01659)) does causal feed-forward 3DGS with a *persistent scene state* updated chunk by chunk. Wayve GAIA-4 (reportedly Aug 2026, closed-loop with the driving policy inside the sim) has only wiki-level sourcing so far, so it waits for a primary source.

---

### The LEARNING.md connection: PSD survives the camera

Last entry: Gaussian splatting stores each 3D covariance as Σ = R S Sᵀ Rᵀ so gradient descent can't leave the PSD cone. Here's the next step in the pipeline, which is what Marble's posed-RGBD-to-splat endpoint runs thousands of times per view.

To draw a 3D Gaussian on screen, the rasterizer needs its **2D** covariance. It gets it with

  **Σ′ = J W Σ Wᵀ Jᵀ**

where W is the camera's view rotation and J is the Jacobian of the perspective projection, a local linear approximation of the nonlinear divide-by-depth. (That's the EWA splatting trick, and it connects to your later graphics unit on projection matrices.)

The PSD question you're studying is exactly why this is safe. **For any matrix A, if Σ is PSD then AΣAᵀ is PSD.** Proof in one line: xᵀ(AΣAᵀ)x = (Aᵀx)ᵀ Σ (Aᵀx) ≥ 0. So the projected footprint is always a valid ellipse, and nothing downstream needs a check. This is a *congruence transform*, and it's worth doing the proof by hand once.

Then eigenvalues come back. The rasterizer computes the two eigenvalues of the 2×2 Σ′ in closed form (trace/determinant formula) and uses **3·√λ_max** as the splat's pixel radius, which decides which screen tiles the Gaussian touches. Eigenvectors give the ellipse's orientation on screen, and eigenvalues give how far it stretches. So: factor (eigendecomposition), transform (congruence keeps PSD), inspect (eigenvalues set the footprint). That's the whole splatting forward pass, and it's the material in your current unit.

*Optimization teaser for the next unit:* WorldParticle's predict-then-correct step is also how a lot of constrained optimization works. You take an easy step from the known prior, then learn or solve for the residual.

---

### Threads worth pulling

1. **Planner-as-patch vs. simulator-as-substrate.** Genie 4 checks pixels with Gemini after they're drawn, while WorldParticle simulates physics on explicit particles and never draws anything. Is there an architecture where particles or Gaussians *are* the world state, a WorldParticle-style corrector advances them, and rendering is just splatting? What would break first? (Probably contact and topology changes. Worth working through.)

2. **Scene memory vs. subject memory.** WorldRoamBench scores these separately, and the brain also seems to separate them (allocentric spatial view cells and place cells vs. egocentric/object coding). StreamSplat keeps an explicit persistent scene state. Does keeping scene memory explicit fix the long-horizon drift that hits pixel-space models? This picks up the continuous-attractor thread from June.

---

### Papers added to covered list

- Google DeepMind, "Genie 4" research preview, Oct 27 2026
- Wang, Chen et al., "WorldParticle: Unified World Simulation of Lagrangian Particle Dynamics via Transformer," arXiv 2605.15305 (MIT News Nov 9 2026)
- "WorldRoamBench: An Open-World Benchmark for Long-Horizon Stability of Interactive World Models," arXiv 2606.31672
- "PrimeSim: A Generalist World Model Simulator from Large-Scale Cross-Embodiment Demonstrations," arXiv 2611.04870
- NVIDIA, Cosmos 3 interactive simulation checkpoints (Transfer-Auto, Sim-Auto, Sim-Transfer, Sim-Bimanual), Nov 5 2026
- World Labs, Marble Gaussian-splat creation + Developer API beta task, Oct 30 2026
- Pezzulo et al., "Grounded world models in biological organisms and future embodied AI," arXiv 2607.13560
- "StreamSplat: Streaming Feed-Forward 3D Gaussian Splatting," arXiv 2608.01659

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
