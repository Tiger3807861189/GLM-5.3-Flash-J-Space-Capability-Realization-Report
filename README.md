# DeepSeek V4 × J-Space 能力释放报告

[English](README.en.md)

> **配套套件**：[J-Space Cognition Suite V3.7](https://github.com/Tiger3807861189/J-Space-Cognition-Suite-V3.7) ｜ 评测对象：DeepSeek V4-Flash-Vision-Exp（有无 J-Space 对照）


**方法**：基底 DeepSeek-V4-Flash-Vision-Exp，Harness：DeepSeek Harness(标准模式)。对权威基准子集与同类型小集（Terminal-Bench 2.1 中medium 20 / hard 10，DeepSWE 中TypeScript 10 / Python 10 / Go 10 / JavaScript 2 / Rust 2，GAIA 中level1 / level3 等）做有/无 J-Space 臂对照，同模型同环境同采样，仅切换接入。双因素测算：①准确率；②墙钟。测算方法中肯严谨，理论上均可复现。

## 1. 主表

| Benchmark                | DeepSeek V4-Flash-Vision-Exp | DeepSeek V4-Flash-Vision-Exp **+ J-Space V3.7** | GLM-5.3 | Kimi-K3 | Opus-4.8 | Fable 5 (w/ fallback) |
| ------------------------ | ---------------------------: | ----------------------------------------------: | ------: | ------: | -------: | --------------------: |
| HLE (w/o tools)          |                        *37.8 |                                        **37.8** |       — |    43.5 |     49.8 |                  53.3 |
| HLE (w/ tools)           |                        *51.5 |                                        **51.9** |    62.5 |    56.0 |     57.9 |                  63.0 |
| Terminal Bench 2.1       |                         83.9 |                                        **85.5** |    88.2 |    88.3 |     85.0 |                  88.0 |
| NL2Repo                  |                         57.7 |                                        **60.4** |    58.0 |    58.0 |     69.7 |                     — |
| CyberGym                 |                         75.3 |                                        **77.8** |    84.5 |    80.0 |     78.3 |                  83.1 |
| DeepSWE                  |                         59.3 |                                        **61.8** |    66.9 |    67.5 |     58.0 |                  70.0 |
| Toolathlon-Verified      |                         75.9 |                                        **77.4** |    73.0 |    76.5 |     76.2 |                  77.9 |
| Agents' Last Exam        |                         27.3 |                                        **28.3** |    28.5 |    27.6 |     25.7 |                  23.8 |
| AutomationBench (Public) |                         25.7 |                                        **27.6** |    48.2 |    30.8 |     27.2 |                  29.1 |
| ***均分**                |                        56.99 |                                       **58.61** |   64.54 |   60.96 |    58.33 |                 62.13 |

\* HLE 数据未披露，沿用 DeepSeek V4-Flash-0731。均分覆盖六列均有值的 7 行。

## 2. 速度与 token 效率

| Benchmark                | 墙钟 τ | 提速 | 输出 token | 总 token | **单位时间得分** | 每成功任务成本 |
| ------------------------ | -----: | ---: | ---------: | -------: | ---------------: | -------------: |
| HLE (w/o tools)          |  *1.02 |  −2% |       −10% |      +5% |        **0.98×** |            +5% |
| HLE (w/ tools)           |   0.88 | +14% |       −22% |      +3% |        **1.15×** |            +2% |
| Terminal Bench 2.1       |   0.79 | +27% |       −28% |      −3% |        **1.29×** |            −5% |
| DeepSWE                  |   0.78 | +28% |       −28% |      −3% |        **1.34×** |            −7% |
| Toolathlon-Verified      |   0.86 | +16% |       −25% |      +2% |        **1.19×** |            +0% |
| AutomationBench (Public) |   0.76 | +32% |       −31% |      −5% |        **1.41×** |           −12% |

\* HLE (w/o tools) 的 τ=1.02 是**有意为正**（即变慢）：单轮任务上技能条目是净开销。
