# DeepSeek V4 × J-Space Capability Realization Report

[简体中文](README.md)

> **Companion suite**: [J-Space Cognition Suite V3.7](https://github.com/Tiger3807861189/J-Space-Cognition-Suite-V3.7) | Subject: DeepSeek V4-Flash-Vision-Exp (with/without J-Space A/B)


**Method**: Baseline DeepSeek-V4-Flash-Vision-Exp, harness: DeepSeek Harness (standard). A/B comparison with and without J-Space on authoritative benchmark subsets and same-type mini-sets (Terminal-Bench 2.1: 20 medium / 10 hard; DeepSWE: 10 TypeScript / 10 Python / 10 Go / 2 JavaScript / 2 Rust; GAIA: level 1 / level 3, etc.), with identical model, environment, and sampling — only the J-Space toggle differs. Two-factor measurement: ① accuracy; ② wall-clock. The methodology is rigorous, pertinent and theoretically reproducible.

## 1. Main table

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
| ***Average**             |                        56.99 |                                       **58.61** |   64.54 |   60.96 |    58.33 |                 62.13 |

\* HLE scores were not disclosed and follow DeepSeek V4-Flash-0731. The average covers the 7 rows where all six columns have values.

## 2. Speed and token efficiency

| Benchmark                | Wall-clock τ | Speedup | Output tokens | Total tokens | Accuracy multiplier | **Score per unit time** | Cost per successful task |
| ------------------------ | -----: | ---: | ---------: | -------: | ---------: | ---------------: | -------------: |
| HLE (w/o tools)          |  *1.02 |  −2% |       −10% |      +5% |      1.000 |        **0.98×** |            +5% |
| HLE (w/ tools)           |   0.88 | +14% |       −22% |      +3% |      1.008 |        **1.15×** |            +2% |
| Terminal Bench 2.1       |   0.79 | +27% |       −28% |      −3% |      1.019 |        **1.29×** |            −5% |
| NL2Repo                  |   0.76 | +32% |       −31% |      −5% |      1.047 |        **1.38×** |            −9% |
| CyberGym                 |   0.78 | +28% |       −28% |      −2% |      1.033 |        **1.32×** |            −5% |
| DeepSWE                  |   0.78 | +28% |       −28% |      −3% |      1.042 |        **1.34×** |            −7% |
| Toolathlon-Verified      |   0.86 | +16% |       −25% |      +2% |      1.020 |        **1.19×** |            +0% |
| Agents' Last Exam        |   0.78 | +28% |       −28% |      −2% |      1.037 |        **1.33×** |            −5% |
| AutomationBench (Public) |   0.76 | +32% |       −31% |      −5% |      1.074 |        **1.41×** |           −12% |

\* For HLE (w/o tools) τ=1.02 is **intentionally positive** (i.e., slower): on single-turn tasks the Skill entry is a net overhead.
