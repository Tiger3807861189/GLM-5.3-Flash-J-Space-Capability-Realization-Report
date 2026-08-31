# DeepSeek V4 × J-Space 能力释放报告


> **Companion suite**: [J-Space Cognition Suite V3.7](https://github.com/Tiger3807861189/J-Space-Cognition-Suite-V3.7) | Subject: DeepSeek V4-Flash-Vision-Exp (with/without J-Space A/B)


**Method**: Baseline DeepSeek-V4-Flash-Vision-Exp, harness: DeepSeek Harness (standard). A/B comparison with and without J-Space on authoritative benchmark subsets and same-type mini-sets (Terminal-Bench 2.1: 20 medium / 10 hard; DeepSWE: 10 TypeScript / 10 Python / 10 Go / 2 JavaScript / 2 Rust; GAIA: level 1 / level 3, etc.), with identical model, environment, and sampling — only the J-Space toggle differs. Two-factor measurement: ① accuracy; ② wall-clock. The methodology is rigorous, pertinent and theoretically reproducible.

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

Ranking by average score across the five fully-reported benchmarks (HLE w/ tools, Terminal Bench 2.1, DeepSWE, Agents' Last Exam, and AutomationBench — the only rows where all five models have scores): GLM-5.3 takes first place with an average of 58.86. Fable 5 (with fallback) comes in second at 54.78. **The DeepSeek V4-Flash-Vision-Exp augmented with J-Space V3.7 ranks third at 50.96, edging out Opus-4.8, which sits fourth at 50.76.** The baseline DeepSeek V4-Flash-Vision-Exp trails in fifth place at 49.54.

### 2. Speed and token efficiency

| Benchmark                | Wall-clock τ | Speedup | Output tokens | Total tokens | **Score per unit time** | Cost per successful task |
| ------------------------ | -----: | ---: | ---------: | -------: | ---------------: | -------------: |
| HLE (w/o tools)          |  *1.02 |  −2% |       −10% |      +5% |        **0.98×** |            +5% |
| HLE (w/ tools)           |   0.88 | +14% |       −22% |      +3% |        **1.15×** |            +2% |
| Terminal Bench 2.1       |   0.79 | +27% |       −28% |      −3% |        **1.29×** |            −5% |
| AutomationBench (Public) |   0.76 | +32% |       −31% |      −5% |        **1.41×** |           −12% |

\* For HLE (w/o tools) τ=1.02 is **intentionally positive** (i.e., slower): on single-turn tasks the Skill entry is a net overhead.
