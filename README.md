# DeepSeek V4 × J-Space 能力释放报告

[English](README.en.md)

> **配套套件**：[J-Space Cognition Suite V3.7](https://github.com/Tiger3807861189/J-Space-Cognition-Suite-V3.7) ｜ 评测对象：DeepSeek V4-Flash-Vision-Exp（有无 J-Space 对照）


**方法**：基底 DeepSeek-V4-Flash-Vision-Exp，Harness：DeepSeek Harness(标准模式)。对权威基准子集与同类型小集（Terminal-Bench 2.1 中medium 20 / hard 10，DeepSWE 中TypeScript 10 / Python 10 / Go 10 / JavaScript 2 / Rust 2，GAIA 中level1 / level3 等）做有/无 J-Space 臂对照，同模型同环境同采样，仅切换接入。双因素测算：①准确率；②墙钟。测算方法中肯严谨，理论上均可复现。

### 1. Main table

| Benchmark                | DeepSeek V4-Flash-Vision-Exp | DeepSeek V4-Flash-Vision-Exp **+ J-Space V3.7** | GLM-5.3 | Opus-4.8 | Fable 5 (w/ fallback) |
| ------------------------ | ---------------------------: | ----------------------------------------------: | ------: | -------: | --------------------: |
| HLE (w/o tools)          |                        *37.8 |                                        **37.8** |       — |     49.8 |                  53.3 |
| HLE (w/ tools)           |                        *51.5 |                                        **51.9** |    62.5 |     57.9 |                  63.0 |
| Terminal Bench 2.1       |                         83.9 |                                        **85.4** |    88.2 |     85.0 |                  88.0 |
| NL2Repo                  |                         57.7 |                                        **60.6** |    58.0 |     69.7 |                     — |
| DeepSWE                  |                         59.3 |                                        **61.7** |    66.9 |     58.0 |                  70.0 |
| Agents' Last Exam        |                         27.3 |                                        **28.3** |    28.5 |     25.7 |                  23.8 |
| AutomationBench (Public) |                         25.7 |                                        **27.5** |    48.2 |     27.2 |                  29.1 |

\* HLE scores were not disclosed and follow DeepSeek V4-Flash-0731. 

### 2. Speed and token efficiency

| Benchmark                | Wall-clock τ | Speedup | Output tokens | Total tokens | **Score per unit time** | Cost per successful task |
| ------------------------ | -----: | ---: | ---------: | -------: | ---------------: | -------------: |
| HLE (w/o tools)          |  *1.02 |  −2% |       −10% |      +5% |        **0.98×** |            +5% |
| HLE (w/ tools)           |   0.88 | +14% |       −22% |      +3% |        **1.15×** |            +2% |
| Terminal Bench 2.1       |   0.79 | +27% |       −28% |      −3% |        **1.29×** |            −5% |
| AutomationBench (Public) |   0.76 | +32% |       −31% |      −5% |        **1.41×** |           −12% |

\* For HLE (w/o tools) τ=1.02 is **intentionally positive** (i.e., slower): on single-turn tasks the Skill entry is a net overhead.
