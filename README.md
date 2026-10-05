---
library_name: transformers
pipeline_tag: text-generation
tags:
- code
- colgrep
---

# ColGrep Default MiniCPM5 2B

RL reward averages file and line F1, discounted 2% per turn after two free turns (16% cap), with grep and missing-search/finish penalties.

## Evaluation results

| Metric | SWE-bench Lite | Multi-SWE-bench Flash |
|---|---:|---:|
| File F1 (%) | 76.18 ± 0.93 | 62.57 ± 0.62 |
| Line F1 (%) | 19.92 ± 0.95 | 15.93 ± 0.76 |
| Avg. assistant turns (including finish) | 6.19 | 6.75 |
| Avg. colgrep calls | 2.26 | 2.41 |
| Avg. terminal calls | 2.94 | 3.34 |
| Avg. generated tokens per trajectory | 314 | 307 |
| Avg. final context tokens (final prompt + completion) | 5,924 | 6,809 |

F1: mean ± 95% confidence interval across five evaluation seeds (300 tasks per benchmark), using strict predictions without rescue. Averages are per trajectory.
