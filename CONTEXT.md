# Research North Star

## The Central Question
What would it mean for an AI to have a *simulator* of the world — not just a predictor of the next frame, but a model that maintains coherent geometry, respects physical laws, and supports counterfactual queries? Fei-Fei Li frames this capacity as "spatial intelligence." I think of it as the difference between a model that knows what a ball looks like versus one that knows it will roll.

## Primary Interest: World Models as Simulators
The specific framing I care about is the **simulator** role — distinct from renderer (makes pretty pictures) and planner (picks actions). A simulator's contract: geometric consistency under novel viewpoints, physics that respects Newton's laws, dynamics that hold up over time. This is the hardest part and the least solved.

Key questions I track:
- How do we get from video prediction → genuine physical simulation?
- Where does 3D scene structure get explicitly represented vs. remain implicit in weights?
- What's the right representation: implicit (NeRF), explicit (Gaussian splatting), occupancy voxels, something else?
- How do biological brains represent space, and does that suggest better inductive biases for artificial systems?

## Framing Sources
- Fei-Fei Li / World Labs — the simulator framing, spatial intelligence manifesto
- Ha & Schmidhuber (2018) — original "World Models" paper for planning
- The NeRF → Gaussian splatting arc as an ongoing scene representation story

## Connected Interests
- **Scene representation**: NeRF, 3D Gaussian splatting, occupancy networks, neural radiance fields
- **Cognitive science / neuroscience**: hippocampal place cells, grid cells, spatial view cells, cognitive maps as biological world models
- **Autonomous driving**: where simulator-quality world models matter most (Waymo, Wayve, NVIDIA Cosmos)
- **Embodied AI / robotics**: world models as training substrates for physical agents

## Labs & Groups to Track
- **World Labs** (Fei-Fei Li, Ben Mildenhall, Justin Johnson, Christoph Lassner)
- **Google DeepMind** — Genie series, Dreamer, Gemini spatial understanding
- **NVIDIA** — Cosmos platform, physical AI pipeline
- **Wayve** — GAIA model, AV world models
- **Waymo** — Waymo World Model
- **MIT Brain & Cognitive Sciences / Kanwisher / DiCarlo labs** — neural scene representation
- **Edmund Rolls (Oxford/Warwick)** — hippocampal scene representation theory
