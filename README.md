---
library_name: transformers
base_model: openbmb/MiniCPM5-2B
pipeline_tag: text-generation
tags:
  - code
  - code-search
  - tool-use
  - reinforcement-learning
  - colgrep
---

# ColGrep Agent 2B

**A small, local agent that finds the files and lines relevant to a coding task.**

ColGrep Agent 2B is a 2B-parameter, non-reasoning model built from [MiniCPM5-2B](https://huggingface.co/openbmb/MiniCPM5-2B). Given a question or issue description, it searches a repository, reads candidate code, and returns precise file paths and line ranges. Its small size makes it practical for fast local inference on CPU or GPU.

Use it directly to navigate an unfamiliar codebase, or as a search sub-agent for a larger coding model. The larger model delegates repository exploration and receives focused code locations to use for planning, editing, or review.

## ColGrep

[ColGrep](https://github.com/lightonai/next-plaid) is a semantic code search tool for your terminal and coding agents, built on the NextPlaid search engine. It combines natural-language search with regex filtering to find code by both meaning and exact matches. ColGrep Agent adds an iterative search-and-read loop: it follows leads, inspects results, and selects the relevant lines.

## Quick start

Install ColGrep with Homebrew on macOS or Linux, select the [GGUF model](https://huggingface.co/lightonai/colgrep-agent-2B-GGUF), and download it:

```bash
brew install lightonai/tap/colgrep
colgrep settings --agent-model lightonai/colgrep-agent-2B-GGUF \
  --agent-model-file colgrep-agent-2B-Q8_0.gguf
colgrep --install-agent
```

The explicit model setting selects this release on ColGrep 1.8.x.

From your repository:

```bash
colgrep init
colgrep --agent "Where are sessions invalidated after logout?"

# Return locations as JSON for another agent or script
colgrep --agent --json "Where are retries configured?"
```

Inference runs locally through llama.cpp, with CPU support and GPU acceleration where available. See the [ColGrep documentation](https://github.com/lightonai/next-plaid/blob/main/colgrep/README.md#agent-mode---agent) for other installation methods and runtime settings.

## Training and harness

We started from MiniCPM5-2B, applied a small cold-start supervised fine-tuning (SFT) stage, then trained with reinforcement learning using **Group Relative Policy Optimization (GRPO)**. Training teaches the model how to use ColGrep within a dedicated code-localization harness, rewarding file and line accuracy while penalizing unnecessary turns.

The harness provides three tools:

| Tool | Purpose |
|---|---|
| `colgrep` | Search code semantically, with optional regex and path filters. |
| `terminal` | Read files and inspect the repository through read-only commands. |
| `finish` | Submit relevant locations as `path/to/file:START-END`. |

The model searches and reads within a 10-turn budget, then submits its selected locations. ColGrep's agent mode integrates this harness. It generates tool calls directly, without a reasoning trace.

## Evaluation

**ColGrep Agent 2B reaches 76.18 file F1 on SWE-bench Lite**, close to the 26–27B models below, while achieving the highest SWE-bench Lite line F1 in this comparison.

| Model | Parameters | SWE-bench Lite file F1 | SWE-bench Lite line F1 | Multi-SWE-bench file F1 | Multi-SWE-bench line F1 |
|---|---:|---:|---:|---:|---:|
| **ColGrep Agent 2B** | **2B** | **76.18** | **19.92** | **62.57** | **15.93** |
| Qwen 3.5 27B | 27B | 77.93 | 15.71 | 68.28 | 16.63 |
| Gemma 4 26B-A4B | 26B total / 4B active | 76.35 | 13.34 | 63.50 | 14.75 |
| Qwen 3.8 27B | 27B | 60.80 | 7.70 | 59.68 | 11.50 |

All rows use ColGrep in the file-and-line localization harness. Scores are F1 percentages, averaged across five seeds with 300 instances per benchmark per seed, using strict predictions without rescue. Evaluation uses a maximum of 10 turns, with a required final submission on turn 10; comparison models run without reasoning at temperature 0.6.

File F1 measures overlap between predicted and reference files; line F1 measures overlap at the line level. These are **code-localization scores**, not SWE-bench issue-resolution rates. The model returns locations for a developer or downstream agent to inspect and edit.
