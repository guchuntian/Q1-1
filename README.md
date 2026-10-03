| Metric | SWE-bench Lite | Multi-SWE-bench Flash |
|---|---:|---:|
| File F1 (%) | 75.19 ± 0.70 | 61.59 ± 0.47 |
| Line F1 (%) | 21.28 ± 0.56 | 16.57 ± 0.19 |
| Avg. assistant turns | 6.72 | 6.90 |
| Avg. colgrep calls | 2.76 | 3.08 |
| Avg. terminal calls | 2.96 | 2.82 |
| Avg. generated tokens / trajectory | 400 | 386 |
| Avg. final context tokens (final prompt + completion) | 6,134 | 6,805 |
| Evaluation trajectories | 300 instances × 5 seeds | 300 instances × 5 seeds |

F1: mean ± 95% confidence interval across seeds, using the evaluator’s rescued predictions. Averages are per trajectory.
