# Awesome Physical AI

A curated list of work on **physical AI**: frontier VLMs / LLMs and coding agents that act as robot policies, write and evolve robot skills, or orchestrate learned policies — with a running scorecard of how far this line has gotten on **RoboDojo** and related manipulation benchmarks.

Scope: papers, reports and repos from 2026 (plus a few earlier foundations). Entries are ordered newest first within each section. Numbers are those reported by the respective authors under their own protocols; treat cross-paper comparisons with care.

## Contents

- 🎯 [RoboDojo Scorecard](#-robodojo-scorecard)
- 📊 [Benchmarks & Evaluation Infrastructure](#-benchmarks--evaluation-infrastructure) — 5 entries
- 🤖 [VLM as Policy (direct closed-loop control)](#-vlm-as-policy-direct-closed-loop-control) — 11 entries
- 🛠️ [Coding Agents that Write / Evolve Robot Policies](#️-coding-agents-that-write--evolve-robot-policies) — 12 entries
- 🧠 [Harnesses that Orchestrate Learned Policies](#-harnesses-that-orchestrate-learned-policies) — 11 entries
- ⚠️ [Negative Results & Lessons](#️-negative-results--lessons)
- 📚 [Surveys & Position Papers](#-surveys--position-papers)
- 🧩 [Harness Design Takeaways](#-harness-design-takeaways)
- 🤝 [Contributing](#-contributing)
- 🙏 [Credits](#-credits)

---

## 🎯 RoboDojo Scorecard

[RoboDojo](https://robodojo-benchmark.com/) ([paper](https://arxiv.org/abs/2607.04434) · [code](https://github.com/robodojo-benchmark/RoboDojo) · [leaderboard](https://robodojo-benchmark.com/LeaderBoard)) is a unified sim-and-real benchmark: 42 Isaac Sim tasks on a bimanual ARX X5 across five capability dimensions (Generalization, Memory, Precision, Long-Horizon, Open) plus 18 real-world tasks on three embodiments. 50 episodes per task, 2,100 episodes total; the aggregate is the mean over the five dimensions. Observations are RGB from a head camera and two wrist cameras plus robot state; no privileged object poses.

**Leaderboard rules that matter for agentic entries:** verified entries must run through the official online evaluation, report three seeds (mean ± std), pass hidden-layout verification, and release checkpoint / code / configs / videos via XPolicyLab. Results without released artifacts are listed separately. How closed-source API models map onto "three seeds" and "reproducible checkpoint" has no precedent yet — check with the maintainers.

### Simulation, average over 5 dimensions

| Method | Type | Score | SR (%) | Notes |
|---|---|---|---|---|
| Human expert (VR teleop) | reference | 80.42 | 76.03 | excluded from ranking |
| RoboDawn 1-shot (GPT-6 Astra) | VLM as policy | 54.63 | 47.17 | 5 runs, discrete EE commands + ICL |
| RoboDawn 0-shot (GPT-6 Astra) | VLM as policy | 39.92 | 35.67 | 5 runs |
| GPT-6 Astra via RoboProbe | VLM as policy | 28.97 | 22.48 | official RoboDojo report, 1 seed, fixed harness |
| DM0.5 | VLA | 24.90 | 19.34 | post-trained on full set |
| G0.5 | VLA | 20.23 | 14.88 | Aug-4 leaderboard #1 |
| Hy-Embodied-0.5-VLA | VLA | 13.07 | 8.80 | paper-freeze (Jul-3) #1 |
| π0.5 | VLA | 11.41 | 6.91 | |
| GPT-5.5 via RoboProbe | VLM as policy | — | 0.88 | same harness as Astra |

Dimension-level pattern for VLM-as-policy so far: strong on Open and Generalization, workable on Long-Horizon and Memory, weak on **Precision** (Astra ≈ 4% SR) and anything dynamic or requiring tight bimanual coordination.

---

## 📊 Benchmarks & Evaluation Infrastructure

#### [RoboDojo: A Unified Sim-and-Real Benchmark for Comprehensive Evaluation of Generalist Robot Manipulation Policies](https://arxiv.org/abs/2607.04434)
**Source:** Chen et al., MMLab@HKU and collaborators · [code](https://github.com/robodojo-benchmark/RoboDojo) · [leaderboard](https://robodojo-benchmark.com/LeaderBoard) · **Published:** 2026-07-05
42 sim + 18 real tasks, heterogeneous parallel Isaac Sim evaluation, remote real-robot evaluation (RoboDojo-RealEval), hidden-layout anti-gaming checks. 30 policies integrated at launch; best VLA at 8.80% SR vs. 76% for humans.

#### [XPolicyLab: A Unified Standard and Open Ecosystem for Robot Policy Evaluation and Deployment](https://arxiv.org/abs/2608.09892)
**Source:** XPolicyLab team · [code](https://github.com/XPolicyLab/XPolicyLab) · **Published:** 2026-08
The policy-server interface RoboDojo evaluates through (observation update, batched action prediction, reset). Any agentic policy targeting the verified leaderboard has to speak this protocol.

#### [CaP-X: A Framework for Benchmarking and Improving Coding Agents for Robot Manipulation](https://arxiv.org/abs/2603.22435)
**Source:** Fu, Yu et al., Berkeley / NVIDIA / Stanford · ICML 2026 · [code](https://github.com/capgym/cap-x) · **Published:** 2026-03
CaP-Gym (39 tasks across Robosuite, LIBERO-PRO, BEHAVIOR), CaP-Bench (8 tiers: single- vs multi-turn, high- vs low-level primitives, visual grounding modes), CaP-Agent0 (training-free agent) and CaP-RL (GRPO on code actions). Key finding: success drops sharply when human-crafted abstractions are removed, and is partly recovered by multi-turn interaction, execution feedback, visual differencing and skill synthesis.

#### [RPent Leaderboard](https://github.com/RLinf/RPent#leaderboard)
**Source:** RLinf team · **Published:** 2026-09-21
Public leaderboard for agentic-planner + frozen-VLA systems on LIBERO, LIBERO-PRO, RoboCasa365 Target50 and RoboTwin C2R. Reported GPT-6 Astra: 92.63% across all LIBERO-PRO suites vs. 82.4% with Claude Opus 4.7 and 50.0% for the frozen VLA alone.

#### [HumanCLAW-Bench](https://arxiv.org/abs/2607.27180)
**Published:** 2026-07
Harness + motion generator for evaluating whether VLMs can act through a humanoid body; used in several Astra demos.

---

## 🤖 VLM as Policy (direct closed-loop control)

A frozen VLM observes, reasons, emits semantic / discrete actions; a deterministic interpreter grounds them. No task-specific training.

#### [RoboICL: Embodied In-Context Learning with GPT-6 Astra](https://arxiv.org/abs/2609.34261)
**Published:** 2026-09
Studies which forms of in-context evidence actually help a frozen VLM on RoboDojo; compares interfaces against concurrent work (GPT-Policy, RoboDawn).

#### [An Unexpected Robot Policy: Early Evaluations of GPT-6 Astra on RoboDojo and Beyond](https://arxiv.org/abs/2609.24170)
**Source:** Zhang, Wang, Ouyang et al., RoboDojo team · [report](https://robodojo-benchmark.com/report/gpt-6-astra-eval) · **Published:** 2026-09-16 (arXiv 09-21)
"LLM as policy" through the fixed, non-learned RoboProbe harness on all 42 tasks. GPT-6 Astra: 22.48% SR / 28.97 score, above all 40 public policies; GPT-5.5 and DeepSeek-Flash under 2% with the same harness. Sharply polarized profile (semantic tasks good, precision/dynamic/bimanual bad); one-shot demos gave no aggregate gain under their protocol; real-robot testing halted for safety.

#### [Transferring the Intelligence of VLMs to Robotic Control (RoboDawn)](https://arxiv.org/abs/2609.22966)
**Source:** Guo et al., Tsinghua / Tencent Hunyuan · [project](https://robodawn.top/) · **Published:** 2026-09-19
Discrete `move / rotate / point / gripper / home / wait / done` commands defined on the gripper interaction point, each grounded into a planned motion; grid-annotated multi-view images; interface-aligned in-context demos. RoboDojo: 35.67% → 47.17% SR (0 → 1 shot). RoboTwin 2.0 C2R: 53.2% → 73.6%, beating π0.5 (46.0%) and HarnessVLA (58.4%). Ablations: reasoning, grid localization and command primer all matter; model choice swings results from 14% (GPT-5.6-Luna) to 73.6% (Astra). Latency ≈ 10 s per decision.

#### [In-Context Robot Learning with VLM Agents (GPT-Policy)](https://arxiv.org/abs/2609.19138)
**Source:** Cheng, Yi, Fang et al. · [eval repo](https://github.com/cheng-haha/GPT-Policy-Eval) · **Published:** 2026-09-16
Context compiler + VLM proposing robot-tool actions + constrained controller that verifies and executes. Human video demos help even without robot action labels; aligned action references help further on contact-sensitive tasks (e.g. plug insertion).

#### [Agent as Policy for Robotic Manipulation (AGP)](https://arxiv.org/abs/2609.12541)
**Source:** Jia et al. · [code](https://github.com/agent-as-policy-2026/agent-as-policy) · **Published:** 2026-09-11
Preparation agent writes a reusable task definition; execution agent writes/selects programs at runtime, issues motion commands, and revises from outcomes. Real dual-arm tasks: video-guided assembly, image-guided block construction, die flipping, targeted throwing, towel folding; ≥ 8/10 on assembly, construction and dice.

#### [Show-Harness: Just a VLM Agent Can Play Robots](https://arxiv.org/abs/2609.10522)
**Source:** Chen, Bai, Cao, Zeng et al., Show Lab NUS · [code](https://github.com/showlab/Show-Harness) · **Published:** 2026-09-10
Discrete semantic action units with embodiment-specific interpreters; one vocabulary across Franka, AgileX Piper (single/dual), ManiSkill and Isaac Lab. Two modes: frontier VLM zero-shot, or small open VLM fine-tuned in a few H200 GPU-hours. GUMI extends the action space to GUI-based demo collection.

#### GPT-6 Astra as an Embodied Policy: Direct End-Effector Control vs. Hybrid Control with π0.5
**Source:** Su et al. · **Published:** 2026-09 · *(arXiv ID to be added)*
Controlled comparison on RoboDojo tasks between Astra directly controlling the end-effector and Astra delegating contact phases to π0.5.

#### [PhysEvo: Astra can act, let it](https://anonymous-report-777.github.io/)
**Source:** anonymous report · **Published:** 2026-09
Evolution-based Astra policy on a 16-task RoboDojo subset; reports 79.80 mean score on the 10 tasks from Su et al. vs. 62.60 hybrid and 38.50 direct.

#### [GPT-as-Policy](https://github.com/anonymous-report-421/GPT-as-Policy)
**Source:** Galbot · **Published:** 2026-09-16
Public benchmark repo and report evaluating GPT-6 Astra as a robot policy.

#### [RoboCurve GPT-6 Astra evaluation](https://openai.robocurve.org/gpt-6-astra/)
**Published:** 2026-09-04
Controlled YAM-arm comparison; 19/20 bowl-task completions with 80% fewer output tokens than the previous generation.

#### [Act-Observe-Rewrite: Multimodal Coding Agents as In-Context Policy Learners for Robot Manipulation](https://arxiv.org/abs/2603.04466)
**Source:** Kumar · **Published:** 2026-03-03
Characterises the conditions under which a multimodal model can learn to manipulate purely from reasoning about its own failures, without gradients, demos or reward engineering.

---

## 🛠️ Coding Agents that Write / Evolve Robot Policies

The agent (Codex, Claude Code, …) is given primitives and a simulator/robot, and iterates code, tools and skills. The deployed artifact is a program or skill library, not a per-step LLM call.

#### Coding Agents for Generalized Task and Motion Planning Problems
**Published:** 2026-09-24 · *(arXiv ID to be added)*
Coding agents used to synthesise TAMP formulations and solvers rather than per-task programs.

#### [RACaP: Agentic Reasoning, Acting, and Coding as Policies for Evolvable Robot Learning](https://arxiv.org/abs/2609.29394)
**Source:** Li et al., CUHK / HKUST(GZ) / Knowin AI · **Published:** 2026-09-24
Moves coding to an offline evolution stage (capability curriculum S0–S6, then autonomous self-evolution) that produces typed, frozen Policy APIs, a ReAct harness and experience memory; at runtime a ReAct loop calls the APIs. No privileged state during evolution. LIBERO-90 54.4%, zero-shot LIBERO-PRO 45.0%, LIBERO-Long 46.0% (CaP baselines ≤ 4% on long-horizon). Distils GPT-5.6 ReAct decisions into Qwen3-VL-8B for 13.2× faster per-decision inference.

#### [Coding Agents with an Obstacle-Aware Harness for Safe Robot Manipulation](https://arxiv.org/abs/2609.20822)
**Published:** 2026-09
Adds collision awareness to the harness so that semantically correct plans do not fail on IK-level collisions — one of the three dominant RoboDawn failure modes.

#### [WetRobo: A Reproducible Robot Kit for Coding Agents in Biological Laboratories](https://arxiv.org/abs/2609.18435)
**Published:** 2026-09
Coding agent completes wet-lab primitives on real hardware, autonomously pulling in OpenCV pose estimation, RANSAC plane fitting, SAM segmentation and MuJoCo collision checks as needed; argues for letting the agent acquire tools rather than fixing the toolset.

#### [PhysCaP: Grounding Code-as-Policy Agent with Physics-Informed Exploration](https://arxiv.org/abs/2608.21031)
**Published:** 2026-08
Lets the code-as-policy agent probe task-relevant physical properties instead of reasoning only over semantics and procedure.

#### [GaP: A Graph-as-Policy Multi-Agent Self-Learning Harness for Variational Automation Tasks](https://arxiv.org/abs/2607.05369)
**Source:** Chen, Xie, Fu et al., Berkeley / NVIDIA · **Published:** 2026-07
Policies represented as computation graphs, improved by a multi-agent self-learning loop.

#### [ASPIRE: Agentic Skills Discovery for Robotics](https://arxiv.org/abs/2607.00272)
**Source:** Lu, Wu, Kou et al., NVIDIA GEAR / Michigan / Berkeley · [code](https://github.com/NVlabs/ASPIRE) · **Published:** 2026-06-30
Coordinator–actor coding agents (Codex GPT-5.5) write and repair control programs in an execution engine that exposes per-primitive multimodal traces (perception overlays, grasp candidates, trajectories, collision feedback); validated repairs are distilled into a persistent skill library; evolutionary search over program populations. Up to +77% over prior methods on LIBERO-PRO; skills transfer sim → real bimanual YAM.

#### [ENPIRE: Agentic Robot Policy Self-Improvement in the Real World](https://arxiv.org/abs/2606.19980)
**Source:** NVIDIA GEAR · [code](https://github.com/NVlabs/ENPIRE) · [project](https://research.nvidia.com/labs/gear/enpire/) · **Published:** 2026-06
Harness with Environment (auto reset + verification), Policy Improvement, Rollout (robot fleet) and Evolution modules. Frontier coding agents reach 99% on real PushT, pin-box organisation and zip-tie cutting; supports CaP heuristics, BC, offline/online RL as improvement regimes.

#### [Playful Agentic Robot Learning (RATs)](https://arxiv.org/abs/2606.19419)
**Published:** 2026-06
Agent proposes its own exploratory tasks during "play", distils successes into a code skill library; +20.6 / +17.0 points over CaP-Agent0 on LIBERO-PRO / MolmoSpaces; skills plug into other CaP agents via retrieval.

#### [RHO: Your Coding Agent is Secretly a Roboticist](https://arxiv.org/abs/2606.16458)
**Source:** Elmaaroufi, Svegliato et al., UC Berkeley / AMD · [project](https://rho-robotics.github.io/) · **Published:** 2026-06-15
"Repositories-as-Policies": a tool-enabled coding agent (Codex GPT-5.5) mutates a multi-file policy repo under a Pareto-frontier reflective optimizer (HELIX, generalising GEPA), scored on environment reward. Deployment is single-turn with no LLM in the loop. Robosuite 70.0% (prior multi-turn record 68.29%); LIBERO-PRO 45.0% with the same low-level primitives (π0.5 12.83%); also optimises a live LLM harness on RAI/O3DE from 23.5% to 44.3%.

#### [Revisiting the "Push-T" Robot Manipulation Task with Agentic Robotics](https://arxiv.org/abs/2608.18227)
**Published:** 2026-08
Agentic re-take on the classic Push-T task.

#### [Code as Policies: Language Model Programs for Embodied Control](https://arxiv.org/abs/2209.07753)
**Source:** Liang et al., Google · ICRA 2023
The origin of the paradigm everything above extends.

---

## 🧠 Harnesses that Orchestrate Learned Policies

The agent plans, remembers, verifies and recovers; frozen VLAs, RL policies, TAMP or analytic primitives execute.

#### [Robo-Harness K1: Harnessing Robot-Use Agents via Perception Augmentation](https://arxiv.org/abs/2609.29389)
**Published:** 2026-09-24
Perception exposed as tools so a single robot-use agent gets better per-step evidence; positioned as complementary to policy-selection harnesses like RoboHarness / HarnessVLA.

#### [Harness VLA / RPent: Steering Frozen VLAs into Reliable Manipulation Primitives via Memory-Guided Agents](https://arxiv.org/abs/2607.08448)
**Source:** Zhang et al., RLinf · [project](https://harnessvla.github.io/) · [code](https://github.com/RLinf/RPent) · **Published:** 2026-07
Agentic planner (Codex / Claude Code / GPT-6 Astra) composes frozen VLA calls with analytic primitives, using memory and visual feedback for retargeting, ordering and retry. RoboTwin 2.0 C2R: 58.0% (Codex) / 58.4% (Claude Code) with LingBot-VLA.

#### [EMERGE-Policy: A Robot Mind Emerges Beyond a Single Policy](https://arxiv.org/abs/2608.29896)
**Published:** 2026-08
Orchestration with a world-model variant; 93.9% on LIBERO-Plus vs. 93.2% reported for RoboHarness.

#### [Don't Drop the BATON: Long-Horizon Manipulation via Agentic Subtask Exploration and Transition-aware Memory](https://arxiv.org/abs/2608.16889)
**Published:** 2026-08

#### [RoboHarness: Memory-Driven Orchestration of Heterogeneous Robot Policies for Long-Horizon Planning](https://arxiv.org/abs/2607.18060)
**Source:** Huang et al. · **Published:** 2026-07
Wraps VLAs, RL policies and TAMP as agentic skills with capability-boundary modules; Memory Bridge retrieves trajectories to steer the robot into the next policy's in-distribution region before handoff. Three public benchmarks, 500 custom tasks, 135 real trials.

#### [Cortex: A Bidirectionally Aligned Embodied Agent Framework for Long-Horizon Manipulation](https://arxiv.org/abs/2607.05377)
**Published:** 2026-07

#### VoLo: A Physical Orchestrator for Open-Vocabulary Long-Horizon Manipulation
**Published:** 2026-07 · *(arXiv ID to be added)*

#### [Goal2Skill: Long-Horizon Manipulation with Adaptive Planning and Reflection](https://arxiv.org/abs/2604.13942)
**Published:** 2026-04

#### [RoboHarness: A Memory-Augmented Policy Harness for VLA Robustness via In-Context Adaptation](https://arxiv.org/abs/2603.24060)
**Source:** LZY-1021 · [code](https://github.com/LZY-1021/RoboHarness) · **Published:** 2026-03
Not the same paper as Huang et al. above. Dual-memory RAG + MLLM orchestrator + MCP interventions around π0 / π0.5 / SmolVLA; +56.6% average SR on LIBERO-PRO and LIBERO-RoboHarness.

#### [RoboClaw: An Agentic Framework for Scalable Long-Horizon Robotic Tasks](https://arxiv.org/abs/2603.11558)
**Published:** 2026-03

#### [Diagnosing Semantic Handoff Failures in Agent-Orchestrated VLA Skill Composition](https://arxiv.org/abs/2607.06256)
**Published:** 2026-07
Why chained VLA skills break at the boundaries.

---

## ⚠️ Negative Results & Lessons

#### [What Stops Recursive Self-Improvement in Robotics? Lessons from 123 Rounds of Agentic Skill Discovery](https://arxiv.org/abs/2609.31760)
**Source:** Wang, NUS · **Published:** 2026-09-23
An agent watched a RoboCasa robot fail, diagnosed missing capabilities, wrote skills or installed external models, tested in sim, and repeated for 123 rounds over several weeks. It did discover capabilities (requested and deployed an active-viewing model), but the target task never succeeded. Three causes: (1) chained perception modules do not understand relations — SAM 3 finds shelves but not "the top shelf", and geometric patch rules never converged; (2) skill chains lock learning onto the first step, so later skills are rarely reached or improved; (3) the harness decides what is learned — the agent optimised exactly what the evaluator measured, including its mistakes.

#### Agentic AI for Robot Control: Flexible but Still Fragile
**Source:** Lima et al., AAAI Symposium Series 2026 · *(link to be added)*

#### RoboDawn / Astra-report failure modes
Insufficient precision at the final placement/insertion stage; IK-level collisions from semantically valid plans; mismatched success judgement (e.g. stopping a pour early); no aggregate benefit from one-shot demos unless they are interface-aligned and sparsified.

---

## 📚 Surveys & Position Papers

- [Self-Evolving Coding Agents: From Digital Programs to Physical-World Intelligence](https://arxiv.org/abs/2609.35432) — 2026-09-29
- [Weights or Skills? A Survey of Robot-Learning Techniques: from Action-Predicting Weights to Robots that Write their Own Skills](https://arxiv.org/abs/2608.01851) — 2026-08
- [From Question Answering to Task Completion: A Survey on Agent System and Harness Design](https://arxiv.org/abs/2606.20683) — 2026-06
- [Code as Agent Harness: Toward Executable, Verifiable, and Stateful Agent Systems](https://arxiv.org/abs/2605.18747) — 2026-05

---

## 🧩 Harness Design Takeaways

Distilled from the entries above; each is backed by an ablation or a reported failure mode.

1. **Action interface over model capacity.** Discrete, semantically named end-effector commands (RoboDawn, Show-Harness) let frontier VLMs beat full-set-trained VLAs zero-shot; the same models collapse to ~1% behind a poor harness (GPT-5.5 / DeepSeek in the RoboDojo report).
2. **Cheap visual grounding is the highest-ROI component.** Grid overlays on images: removing them costs more than removing reasoning (RoboDawn: 47.0 → 32.4 vs. 34.8).
3. **Demos must be in the agent's own action space.** Raw trajectories → waypoints → command sequences, with the image budget capped (≈16) and only semantically informative rounds kept. Unaligned demos gave no gain in the Astra report.
4. **Give the agent a command budget and let it spend it.** Success scales monotonically with per-episode command budget (RoboDawn: 31.2 → 47.2% one-shot from 60 → 240 commands).
5. **Move coding offline.** Runtime code generation is slow and entangles task specifics; evolve typed APIs / skill libraries / repos first (RHO, RACaP, ASPIRE), call them from a lightweight ReAct policy at deployment, and distil that policy into a small VLM if latency matters.
6. **Expose execution traces, not just success.** Per-primitive overlays, grasp candidates, trajectories and collision feedback (ASPIRE, CaP-X visual differencing) are what make failure attribution possible.
7. **Precision needs a learned or servoed primitive.** Every VLM-as-policy result bottoms out on insertion / capping / stacking accuracy; hybrid designs that hand contact phases to a VLA or a visual-servo controller (Su et al., HarnessVLA/RPent) are the current fix.
8. **Perception tools must understand relations.** SAM-style segmenters plus geometric rules do not converge on "the top shelf"; plan for a relational grounding model, not more rules.
9. **Guard against evaluator over-fitting.** Self-improving loops optimise the scorer, including its bugs; hold out layouts (RoboDojo uses hidden verification layouts) and audit memory.
10. **Budget for cost and reproducibility.** ~10 s per decision and frontier-API token prices make 2,100-episode runs expensive; leaderboards additionally expect multi-seed, artifact-released submissions.

---

## 🤝 Contributing

Pull requests are welcome. Please include: title with link, authors / lab, publication date, and a two-to-three-sentence summary that states the interface (VLM-as-policy, coding agent, orchestration), the benchmark, and the headline number as reported by the authors. Keep entries newest-first within each section.

## 🙏 Credits

Credit belongs to the original authors and projects referenced by each entry. This list is a non-commercial compilation and technical summary; it claims no ownership of the underlying work.
