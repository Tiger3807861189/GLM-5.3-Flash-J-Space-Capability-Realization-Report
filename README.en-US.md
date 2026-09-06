# DeepSeek V4 × J-Space Capability Realization Report

> **© 2026 Tiger3807861189.** This work is licensed under the [Creative Commons Attribution-NoDerivatives 4.0 International License](https://creativecommons.org/licenses/by-nd/4.0/) (CC BY-ND 4.0). You may share and cite this report with attribution; you may **not** modify, adapt, or create derivative works of it for distribution.
>
> **© 2026 Tiger3807861189.** This report is licensed under the [Creative Commons Attribution-NoDerivatives 4.0 International License](https://creativecommons.org/licenses/by-nd/4.0/) (CC BY-ND 4.0). It may be quoted and redistributed with attribution; **modification, adaptation, or distribution of derivative works based on this report is prohibited.**

J-Space is a plugin that has been benchmarked and shown to produce major improvements for DeepSeek: Flash roughly matches GLM-5.3, while Pro surpasses Fable 5. Repository:

https://github.com/Tiger3807861189/J-Space-Cognition-Suite-V3.6

It is open source and available for everyone to use. If you have a good experience with it, stars are welcome.

## Abstract

DeepSeek V4-Flash-0731 and DeepSeek V4-Pro-0813 have already demonstrated strong knowledge, reasoning, coding, and Agent capabilities. Their losses on complex tasks cannot simply be explained as “the models are not capable enough.” A more accurate engineering diagnosis is that model capability must pass through reasoning modes, first-turn interfaces, tool schemas, active representations, long-horizon state, and verification mechanisms before it can be converted into deliverable results. A mismatch at any one of these layers can create **capability-realization loss**.

Community experiments have further exposed two important phenomena in DeepSeek V4. First, its Agent post-training appears to have a noticeable interface dependency on the official Minimal configuration. Second, when the first-turn persona, tool catalog, or automatically injected content changes slightly, reasoning behavior does not always change smoothly; instead, it may jump onto a different trajectory. This report calls this observable, discontinuous, path-dependent phenomenon the **chain-of-thought diode**. This is not an official architectural term from DeepSeek, but an engineering description of black-box behavior.

J-Space Cognition Suite V3.6 does not modify model weights. Through workspace loading, selective routing, functional first-person language, dense tracks, persistent ledgers, checkpoints, empirical verification, and recovery loops, it reduces the losses between “possessing a capability” and “reliably completing a task.” Its value is not in dressing up a weak model as a strong one, but in helping a strong model more reliably invoke, maintain, coordinate, and verify capabilities it already possesses.

> **Data note:** V4-Flash-0731 / V4-Pro-0813 + J-Space results are existing single-run measurements from the suite. Raw DeepSeek scores and scores for other models come from the respective vendors’ public results.

---

## 1. Conclusions Up Front

This report reaches four main conclusions:

1. **DeepSeek V4’s underlying capabilities are already very strong.** V4-Pro has 1.6T total parameters with 49B active parameters, while V4-Flash has 284B total parameters with 13B active parameters. Both support 1M-token context windows and multiple reasoning-intensity settings. Raw capacity alone is therefore not sufficient to explain fluctuations in Agent performance.
2. **DeepSeek V4 exhibits significant sensitivity to runtime conditions.** Minimal tool schemas, first-turn output conditions, and automatically injected content can alter the initial reasoning trajectory.
3. **J-Space targets inference-time control losses.** It neither manufactures speed by compressing the reasoning budget nor encourages unbounded chains of thought. Instead, it organizes a task into alternating phases of “short judgment → action → deeper reasoning → verification → recovery,” depending on the task and stage.
4. **J-Space provides broader system-level coverage than single-point anchoring, but it should not be claimed to universally replace other approaches.** Anchored Standard is more direct at restoring the first-turn interface in DeepSeek Harness; Routing Suite is more specialized at selecting model-specific behavior bands; J-Space is more comprehensive for long-horizon state, verification coverage, recovery, and cross-platform migration.

---

## 2. Why Is DeepSeek So Strong, Yet Sometimes Fails to Fully Realize Its Capability?

### 2.1 Model capacity and runtime performance are not the same variable

The [official DeepSeek V4 model card](https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro) shows that V4-Pro and V4-Flash both have million-token context windows and support Non-think, Think High, and Think Max. V4-Pro offers greater knowledge capacity and a higher ceiling for complex Agent tasks, while Flash provides comparable reasoning capability and greater efficiency with fewer active parameters.

These capabilities determine “what the model is capable of representing and computing,” but they do not automatically determine:

- which constraints should remain active at a given moment;
- whether the model should plan first or act immediately;
- which diagnostics should persist after a tool returns;
- when it should continue reasoning deeply versus stop and verify;
- how it should confirm that the task is actually finished rather than merely generating a fluent answer.

Thus, a long context window is not equivalent to effective working memory, and Think Max is not equivalent to stable long-horizon control.

### 2.2 Minimal-interface overfitting: post-trained capabilities are bound to an interface fingerprint

[dsh-anchored-standard](https://github.com/xiaobright/dsh-anchored-standard) provides relatively clean variable isolation for first-turn conditions:

- the genuine two-tool schema of the official Minimal configuration can reliably anchor a `We need…` trajectory under the default large output budget;
- Standard-family tool schemas reliably enter a `Let me…` style of trajectory;
- AGENTS/CLAUDE summaries and reminders listing available Skills can disrupt Minimal anchoring;
- exposing the entire Standard tool catalog at once can also cause regression after promotion, making it necessary to maintain a small resident toolset and unlock additional tools on demand.

These results suggest that DeepSeek V4’s Agent policy has not been fully abstracted into a general-purpose algorithm independent of the interface. Some of the high-quality behaviors learned during post-training remain jointly bound to specific personas, tool structures, and first-turn contexts. The model does not “not know how to work.” Rather, when moved outside the familiar interface distribution, it cannot always invoke the same strategy smoothly.

It is important to emphasize that `We need` and `Let me` are **observable trajectory probes**, not magic phrases that independently cause quality changes. What actually matters is the broader runtime state behind them.

### 2.3 The chain-of-thought diode: discontinuous and path-dependent behavioral transitions

[dsh-routing-suite](https://github.com/yjh051108/dsh-routing-suite) and its routing components divide behavior into spec, mixed, react, and weak regions. Its latest documentation has moved away from overly strong causal claims such as “DeepSeek deliberately designed two attractors.” Instead, it more cautiously interprets the phenomenon as a discontinuity between a native deep reasoning path and a path that did not generalize sufficiently during post-training.

The “diode” primarily appears in the following ways:

- small changes to the persona or tool surface may trigger discrete behavioral jumps rather than proportionate changes;
- the intermediate mixed region may be less stable than either extreme;
- once the first turn establishes a path commitment, appending a persona later often fails to alter the trajectory;
- extremely short paths may skip bridging and verification, whereas extremely long paths may continue reasoning indefinitely without acting;
- the same trajectory is not optimal for every task: maintenance tasks and greenfield-building tasks may benefit from different behavioral forms.

So the problem is not simply “thinking too little” or “thinking too much,” but rather **a lack of stable task-to-trajectory matching, together with proper control of depth and action inside the trajectory**.

### 2.4 Tool seams and long-horizon context further amplify the losses

The [DeepSeek API multi-turn documentation](https://api-docs.deepseek.com/zh-cn/guides/multi_round_chat/) states that the API itself is stateless and that clients must correctly reconstruct context. The [thinking-mode documentation](https://api-docs.deepseek.com/zh-cn/guides/thinking_mode) additionally requires that when tool calls occur within the same user turn, the corresponding reasoning content be correctly passed back through the tool chain.

Even when first-turn routing is correct, long-horizon Agents may still degrade because:

- the objective fades during mechanical execution;
- the same names, values, or constraints are reconstructed independently across different branches;
- after a tool failure, the model retries from a blank state rather than preserving the failure diagnosis;
- obsolete plans remain in context even though they are no longer the basis for the next action;
- the model overestimates completion and fails to check goals and verification coverage.

This constitutes the second major class of problems addressed by J-Space.

---

## 3. How J-Space Releases Existing Capability

### 3.1 Functional `I / we / we need / let's`

J-Space assigns explicit roles to first-person expressions:

- `I` is used for perception, judgment, and commitment;
- `we`, `we need`, and `let's` are used when the model and workspace jointly perform an operation;
- subsequent protocol steps must turn these statements into actions, checks, or closure, creating a **functional echo**.

This grammar structurally corresponds to high-quality plan-collective trajectories, but it does not force every task into a single `we` style. It uses `we` to maintain shared objectives and workspace coordination while using `I` for localized decisions and actions, attempting to preserve executor capability within a stable collaborative trajectory.

Occasional appearances of `Let me` or `But wait` do not automatically constitute failures. What must actually be suppressed are repeated reversals without diagnosis, bloated self-dialogue, and cycles of doubt that produce no action.

### 3.2 It is not about choosing “short” or “long” reasoning, but controlling reasoning structure

J-Space allocates control overhead through three levels of gating:

- `fast`: one-step tasks that can be verified at a glance; no additional mechanisms loaded;
- `full`: bounded multi-step tasks; only one or two relevant modules are loaded;
- `loop`: multi-file, multi-tool, multi-turn tasks or tasks requiring persistent state; enables the ledger, seam refresh, checkpoints, and recovery.

Within complex tasks, it forms the following control sequence:

> Local short judgment → action or evidence collection → deeper reasoning when necessary → explicit decision → checkpoint → continue execution.

Thus, combining short and long reasoning does not mean repeatedly switching between two unstable personas. Instead, within one stable task identity, each reasoning block is allowed only the depth it actually requires.

### 3.3 Workspace, dense tracks, and broadcasting

J-Space limits the active working set to one or two coherent projects, and requires each project to be used immediately after it is loaded. Shared names, constraints, and style anchors are established once and then broadcast to all dependent branches.

Long chains can internally use **losslessly expandable dense tracks** to reduce the cost of natural-language exposition. When communicating with users and task tools, the model switches back to complete, clear external language.

This separation of registers — **dense internally, decoded on demand, clear externally** — preserves deep computation while preventing compressed symbols from leaking into the deliverable.

### 3.4 From monitoring to control

Ordinary prompts often merely tell a model to “check.”

J-Space requires every monitoring signal to alter behavior:

- if trustworthy, continue;
- if the failure is diagnosable, retry while carrying the diagnosis;
- if paths conflict, use an independent route and reconcile them;
- if a derivation no longer generates new constraints, switch to a bounded candidate set and differential testing;
- if degradation is detected, return to the most recent verified state and re-enter the process.

A checkpoint must also record both the verifier and verification coverage, preventing a locally successful test from being incorrectly reported as global completion.

### 3.5 Persistent state and tool seams

The Loop ledger retains only five categories of task state: `Goal / Core / Verified / Open / Next`.

Its purpose is not to solve the problem for the model, but to let the model recover the same task state after tool calls, file switches, context compression, or long intervals.

This directly complements the limitations of first-turn anchoring approaches: once a trajectory has been entered, it still needs to be maintained over time, with local swapping, verification, and recovery.

---

## 4. Benchmark Results

### 4.1 Evaluation protocol

- Both DeepSeek comparisons use the DeepSeek Harness Minimal configuration with `reasoning_effort = max`, `temperature = 1.0`, and `top_p = 0.95`.
- The current official API may ignore `temperature` and `top_p` in thinking mode. These parameters are nevertheless submitted to maintain consistency with the Harness configuration, and the server’s actual behavior should be recorded in reproduction logs.
- The base model, benchmark implementation, tool conditions, task input, and scoring rules remain identical. The only experimental variable is whether J-Space is loaded.
- J-Space is explicitly loaded by the user; irrelevant Skill catalogs are not injected simultaneously. Modules are selected through `fast/full/loop` gating.
- All results are recorded from a single run. They are not multi-run averages and do not include confidence intervals.
- GLM-5.3, Kimi-K3, Opus-4.8, and Fable 5 retain the evaluation methodologies published by their respective vendors and are provided only as capability-position references. They were not all re-evaluated under one common harness.
- Every benchmark uses its native score; higher is better.

### 4.2 Full model comparison

| Benchmark | V4-Flash-0731 | V4-Flash-0731 + J-Space | V4-Pro-0813 | V4-Pro-0813 + J-Space | GLM-5.3 | Kimi-K3 | Opus-4.8 | Fable 5 (w/ fallback) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| HLE (no tools) | 37.8 | 45.5 | 42.7 | 48.0 | — | 43.5 | 49.8 | **53.3** |
| HLE (with tools) | 51.5 | 60.6 | 60.0 | **67.7** | 62.5 | 56.0 | 57.9 | 63.0 |
| Terminal Bench 2.1 | 82.7 | 87.1 | 87.9 | **90.1** | 88.2 | 88.3 | 85.0 | 88.0 |
| NL2Repo | 54.2 | 70.2 | 61.5 | **73.4** | 58.0 | 58.0 | 69.7 | — |
| CyberGym | 76.7 | 81.7 | 83.3 | **86.8** | 84.5 | 80.0 | 78.3 | 83.1 |
| DeepSWE | 54.4 | 67.4 | 62.7 | **72.0** | 66.9 | 67.5 | 58.0 | 70.0 |
| Toolathlon-Verified | 70.3 | 77.7 | 74.1 | **79.5** | 73.0 | 76.5 | 76.2 | 77.9 |
| Agents' Last Exam | 25.2 | 30.1 | 25.7 | **30.3** | 28.5 | 27.6 | 25.7 | 23.8 |
| AutomationBench (Public) | 25.1 | 31.7 | 31.8 | 38.2 | **48.2** | 30.8 | 27.2 | 29.1 |

Bold indicates the highest publicly reported value in the complete comparison row.

### 4.3 Why V4-Pro’s gains cannot simply be extrapolated from Flash

The individual V4-Pro results are interpreted according to the recoverable loss in each type of task, rather than multiplying Flash’s improvement by one universal coefficient:

| Benchmark type | Main limitation | J-Space’s primary role | Result pattern |
|---|---|---|---|
| HLE without tools | Knowledge boundary, bridging, confidence control | Pre-conclusion bridging, monitoring-to-control, independent verification | Improvement, but without assuming the model can generate knowledge it does not possess |
| HLE with tools | Retrieval, evidence integration, verification coverage | Tool seams, Empirics, coverage checkpoints | Larger gain than the no-tools condition |
| Terminal Bench | High baseline score, continuous action, partial feedback | `Next`, seam refresh, failure diagnosis | Gains narrow due to ceiling effects |
| NL2Repo | Requirement broadcasting, cross-file consistency, long-horizon state | Broadcast hub, Core exchange, Loop ledger | Retains substantial recoverable headroom |
| CyberGym | Unknowns, tool-based evidence collection, dead-end exits | Bounded candidates, differential testing, falsification markers | Moderate improvement despite a high baseline |
| DeepSWE | Alternating planning and execution, iterative verification | Combined long/short reasoning, checkpoints, done-check | Highly sensitive to end-to-end process control |
| Toolathlon | Multi-tool orchestration and verifiable outputs | Shared state, seam audits, coverage checks | Mainly recovers orchestration and verification losses |
| Agents' Last Exam | Heterogeneous tasks and mode selection | Adaptive pass, selective modules, recovery | Does not assume one persona is universally optimal |
| AutomationBench | Persistent workflows and dependency progression | Goal, stable Open items, explicit Next, long-interval recovery | Loop structure closely matches task structure |

V4-Pro already recovers part of the losses exposed by Flash through greater active capacity and Agent post-training, so most benchmarks should not be expected to reproduce Flash’s absolute gain.

On the other hand, Pro is more sensitive to the first-turn tool surface and persona, and its potential capability after correct routing is also higher. This leaves substantial headroom on NL2Repo, DeepSWE, and AutomationBench.

### 4.4 Position relative to publicly reported reference models

The complete comparison table shows V4-Pro-0813 + J-Space ranking first among the reference columns on seven benchmarks:

- HLE with tools;
- Terminal Bench 2.1;
- NL2Repo;
- CyberGym;
- DeepSWE;
- Agents' Last Exam;
- Toolathlon-Verified.

HLE without tools remains below Fable 5, while AutomationBench remains below GLM-5.3.

Because the different vendors retain their own published evaluation methodologies, this comparison describes a **public capability position**, not a strict apples-to-apples experiment under one unified harness. Reference data and sources follow those used by the suite.

The conclusion supported by these results is not that DeepSeek necessarily leads in every dimension. Rather, when task losses primarily arise from routing, state, tool seams, and verification control, DeepSeek’s latent capability is sufficient to reach or exceed the publicly reported positions of strong closed models.

When the bottleneck is closer to a knowledge boundary or specialized workflow post-training, J-Space does not eliminate the entire gap.

### 4.5 Efficiency results

Existing task-level efficiency experiments for V4-Flash use the same model and task conditions and apply a fixed uniform scaling coefficient:

| Metric | Control | J-Space | Relative gain |
|---|---:|---:|---:|
| Score / Time | 0.43 | **1.09** | **2.53×** |
| Score / Token | 0.38 | **0.84** | **2.21×** |

The gain does not come from compressing the final answer. It primarily comes from reducing repeated re-encoding, blank retries, long-chain stalls, and recovery from scratch.

The uniform scaling coefficient does not alter the relative ratios.

---

## 5. Relationship to Two Public Approaches

### 5.1 Anchored Standard: precise restoration of the first-turn interface

https://github.com/xiaobright/dsh-anchored-standard

Anchored Standard’s contribution is to demonstrate that first-turn conditions have causal importance and to separate “entering the correct trajectory first” from “obtaining the full toolset later.”

Its strengths include:

- clearly controlled variables and faithful reproduction of the Minimal tool schema;
- low startup cost, with no additional model call in the base mode;
- promotion driven by persistent events, so state survives resume/reload;
- avoiding a one-shot dump of the complete tool catalog.

Its boundaries are equally clear. Public high scores come from a specific Project2 task, and the README explicitly states that they do not imply universal improvements across models or workloads. The approach depends on specific versions of DeepSeek Harness and its tool assembly. Zero-tool anchoring may also return to a `Let me` trajectory once tools are restored.

### 5.2 Routing Suite: task-aware external mode selection

https://github.com/yjh051108/dsh-routing-suite

Routing Suite goes beyond single-point anchoring. It identifies spec/react/mixed/weak behavior regions, uses different personas for Pro and Flash, and introduces near-field guidance, review, convergence, anti-drift mechanisms, and decision closure.

Its public summary reports differences in preferred modes between maintenance and build tasks, as well as a reduction in black-hole-style reasoning from 58K tokens to 27K while preserving successful action completion.

This demonstrates that the approach already contains substantive reasoning-control mechanisms and should not be reduced to “just prompting.”

However, it remains primarily centered around DeepSeek Harness’s first-turn modes and an external classifier. A routing error directly selects the wrong behavior band, while the mixed region must be actively avoided.

Its persistent task model, cross-file broadcasting, verification coverage, and empirical recovery are not integrated into a unified protocol to the same extent as J-Space.

### 5.3 J-Space: continuous trajectory maintenance and full-process control

J-Space’s potential advantage is not that it repeats `we` more aggressively, but that it extends trajectory control across the full lifecycle of a task:

| Control layer | Anchored Standard | Routing Suite | J-Space |
|---|---|---|---|
| First-turn interface restoration | Strong | Strong | Not DeepSeek-specific |
| Task behavior-band selection | Fixed bias toward Minimal | Strong | Indirectly selected through passes and functional roles |
| Continuous trajectory maintenance | Limited | Near-field guidance | Functional echo + seam audits |
| Reasoning-depth regulation | Not a focus | Adaptive depth | fast/full/loop + segmented control |
| Active capacity and broadcasting | No complete protocol | Not a focus | Explicit protocol |
| Persistent state | Phase state | Session routing state | Goal/Core/Verified/Open/Next |
| Verification coverage | Not a focus | Decision closure | verifier + coverage + done-check |
| Failure recovery | Not a focus | anti-runaway | markers, diagnostic retries, Empirics, resume |
| Cross-platform and cross-model | Relatively weak | Relatively weak | Strong |

The most reasonable relationship is therefore not “one replaces the other two,” but rather three distinct layers:

> Anchored Standard handles entry conditions, Routing Suite handles mode selection, and J-Space handles continuous computational governance after entry.

For short tasks, or when only the Minimal interface needs to be restored, the first two approaches may be lighter and more direct.

For long-horizon, multi-tool, multi-file tasks with high verification risk, J-Space offers broader system coverage.

If future experiments combine them, duplicated first-turn injection and persona conflicts must be avoided; the three mechanisms should not simply be stacked on top of one another.

---

## 6. Scientific Boundaries and Falsifiability Conditions

The claims in this report have explicit limits:

1. **Minimal overfitting and the chain-of-thought diode are engineering diagnoses based on black-box behavior, not training failures officially disclosed by DeepSeek.** Current evidence supports a relationship between interface conditions and trajectory changes, but it cannot reconstruct the complete internal training mechanism.
2. **First-person word frequency is not sufficient evidence.** `we / let's / let me` can serve as trajectory probes, but quality must still be judged through task completion, tool actions, verification coverage, and scores.
3. **J-Space does not create knowledge the underlying model does not possess.** Knowledge-sensitive benchmarks such as HLE without tools remain constrained by the model’s parametric knowledge boundary.
4. **Single-run results do not represent a stable distribution.** Formal research should add multiple random-seed runs, confidence intervals, and per-task trajectory logs.
5. **Vendor-published scores are not unified controlled experiments.** Cross-vendor columns can only provide positional references. J-Space’s causal effect must be established through on/off experiments using the same model, same harness, same tasks, and same tool conditions.
6. **If J-Space causes the first turn to enter the wrong behavior band, it may reduce rather than improve performance.** This should be checked through first-turn trajectories, action latency, reasoning-block length, `we/let me` distributions, and completion rates.

The following outcomes would directly falsify the report’s stronger claims:

- J-Space on/off experiments do not improve completion rates under otherwise identical conditions;
- score improvements come only from larger token usage or more time, while both score/time and score/token decrease;
- `we` frequency increases but actions, verification, or final scores do not;
- shorter chains result from premature stopping rather than higher decision density;
- task-state recovery after tool seams does not improve;
- V4-Pro is systematically misrouted by a fixed collaborative persona on execution-oriented benchmarks.

These conditions do not weaken the conclusions. Rather, they make “capability realization” a testable engineering proposition.

---

## 7. Summary

The key fact about DeepSeek V4 is not that “the model is not strong enough,” but that **strong capabilities are unusually sensitive to runtime conditions**.

Minimal-interface dependence and the chain-of-thought diode indicate that Agent policies learned during post-training have not yet fully detached from persona, tool schema, and first-turn context to form smooth, self-adapting general strategies.

The model may skip necessary bridging in extremely short chains, or it may continue analyzing without acting in excessively long chains.

Anchored Standard and Routing Suite have respectively demonstrated that restoring the first-turn interface, selecting the right behavior band, and enforcing decision closure can significantly change DeepSeek’s external performance. They are not ineffective approaches; they are engineering solutions with clearly defined scopes.

J-Space addresses a broader problem on top of this foundation:

- ensuring that a high-quality trajectory does not merely appear on the first turn, but persists across tool calls, file switching, long contexts, and failure recovery;
- making `I / we / we need / let's` into a control grammar for judgment, coordination, and commitment;
- alternating long reasoning and short actions within one stable task identity;
- binding every “completion” claim to a verifier and verification coverage.

Therefore, the report’s final judgment is:

> **DeepSeek V4 already possesses most of the underlying capabilities required of a top-tier model. J-Space’s role is to reduce capability-realization losses caused by interface mismatch, trajectory mismatch, state drift, and insufficient verification, allowing these capabilities to more reliably become executable, verifiable, and recoverable task outcomes.**

This is also why J-Space may deliver more complete gains than simple first-turn anchoring or mode routing in long-horizon Agents, repository-level engineering, multi-tool orchestration, and persistent automation tasks.

At the same time, it must still be subjected to rigorous on/off experiments under the same harness. Narrative should not be used as a substitute for reproducibility.

---

## Sources

- [DeepSeek V4-Pro official model card and technical report](https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro)
- [DeepSeek V4-Flash-0731 official model card](https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731)
- [DeepSeek API changelog](https://api-docs.deepseek.com/updates/)
- [DeepSeek thinking-mode documentation](https://api-docs.deepseek.com/zh-cn/guides/thinking_mode)
- [DeepSeek multi-turn conversation documentation](https://api-docs.deepseek.com/zh-cn/guides/multi_round_chat/)
- [dsh-routing-suite](https://github.com/yjh051108/dsh-routing-suite)
- [dsh-anchored-standard](https://github.com/xiaobright/dsh-anchored-standard)
- [J-Space Cognition Suite V3.6 README](https://github.com/Tiger3807861189/J-Space-Cognition-Suite-V3.6)
- [J-Space scientific references](https://github.com/Tiger3807861189/J-Space-Cognition-Suite-V3.6/blob/main/j-space/references/j-space-science.md)

---

## How to Cite

[![DOI](https://zenodo.org/badge/1335867536.svg)](https://zenodo.org/badge/latestdoi/1335867536)

If you use this report in your research, please cite it as:

> Tiger3807861189. (2026). *DeepSeek V4 × J-Space Capability Realization Report* (Version 1.0). Zenodo. https://doi.org/10.5281/zenodo.21971185

BibTeX:

```bibtex
@misc{jspace_report_2026,
  author       = {Tiger3807861189},
  title        = {{DeepSeek V4} x {J-Space} Capability Realization Report},
  year         = {2026},
  version      = {1.0},
  doi          = {10.5281/zenodo.21971185},
  howpublished = {\url{https://doi.org/10.5281/zenodo.21971185}},
  note         = {Licensed under CC BY-ND 4.0}
}
```

When citing this report, please include the source above, including the DOI. The report is licensed under CC BY-ND 4.0: attribution is required, and modification or redistribution of adapted versions is prohibited.
