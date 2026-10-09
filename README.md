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
language:
- en
---

# ColGrep-Agent-2B

![colgre-agent](media/banner.png)

**A small, local agent that finds the files and lines relevant to a coding task.**

ColGrep Agent 2B is a 2B-parameter, non-reasoning model built from [MiniCPM5-2B](https://huggingface.co/openbmb/MiniCPM5-2B). Given a question or issue description, it searches a repository, reads candidate code, and returns precise file paths and line ranges. Its small size makes it practical for fast local inference on CPU or GPU.

Use it directly to navigate an unfamiliar codebase, or as a search sub-agent for a larger coding model. The larger model delegates repository exploration and receives focused code locations to use for planning, editing, or review.

## ColGrep

[ColGrep](https://github.com/lightonai/next-plaid) is a semantic code search tool for your terminal and coding agents, built on the NextPlaid search engine. It combines natural-language search with regex filtering to find code by both meaning and exact matches. ColGrep Agent adds an iterative search-and-read loop: it follows leads, inspects results, and selects the relevant lines.

## Quick start

Install ColGrep with Homebrew on macOS or Linux, select the [GGUF model](https://huggingface.co/lightonai/colgrep-agent-2B-GGUF), and download it:

```bash
brew install lightonai/tap/colgrep
colgrep --install-agent

# From your repository
colgrep init
colgrep --agent "Where are sessions invalidated after logout?"
```


Inference runs locally through llama.cpp, with CPU support and GPU acceleration where available. See the [ColGrep documentation](https://github.com/lightonai/next-plaid/blob/main/colgrep/README.md#agent-mode---agent) for other installation methods and runtime settings.

## Training and harness

We started from MiniCPM5-2B, applied a small cold-start supervised fine-tuning (SFT) stage, then trained with GRPO for 500 steps. Training teaches the model how to use ColGrep within a dedicated code-localization harness, rewarding file and line accuracy while penalizing unnecessary turns.

The harness provides three tools:

| Tool | Purpose |
|---|---|
| `colgrep` | Search code semantically, with optional regex and path filters. |
| `terminal` | Read files and inspect the repository through read-only commands. |
| `finish` | Submit relevant locations as `path/to/file:START-END`. |

The model searches and reads within a 10-turn budget, then submits its selected locations. ColGrep's agent mode integrates this harness. It generates tool calls directly, without a reasoning trace.

## Evaluation

We evaluate ColGREP-Agent-2B on [SWE-Bench Lite](https://huggingface.co/datasets/SWE-bench/SWE-bench_Lite) and [Multi-SWE-Bench Flash](https://huggingface.co/datasets/ByteDance-Seed/Multi-SWE-bench-flash).


<table>
  <thead>
    <tr>
      <th rowspan="2">Model</th>
      <th rowspan="2">Harness</th>
      <th colspan="2">SWE-bench Lite</th>
      <th colspan="2">Multi-SWE-bench</th>
    </tr>
    <tr>
      <th>File F1</th>
      <th>Line F1</th>
      <th>File F1</th>
      <th>Line F1</th>
    </tr>
  </thead>
  <tbody>
    <tr style="background-color: rgba(0, 102, 255, 0.10);">
      <td><b>ColGrep-Agent-2B</b></td>
      <td><b>ColGrep Harness</b></td>
      <td><b>76.18</b></td>
      <td><b>19.92</b></td>
      <td><b>62.57</b></td>
      <td><b>15.93</b></td>
    </tr>
    <tr>
      <td>Qwen-3.5 27B</td>
      <td>ColGrep Harness</td>
      <td>77.93</td>
      <td>15.71</td>
      <td>68.28</td>
      <td>16.63</td>
    </tr>
    <tr>
      <td>Gemma-4-26B-A4B</td>
      <td>ColGrep Harness</td>
      <td>76.35</td>
      <td>13.34</td>
      <td>63.50</td>
      <td>14.75</td>
    </tr>
    <tr>
      <td>Qwen-3.8-27B</td>
      <td>ColGrep Harness</td>
      <td>60.80</td>
      <td>7.70</td>
      <td>59.68</td>
      <td>11.50</td>
    </tr>
  </tbody>
</table>



All models are used with our ColGrep harness. Scores are F1 percentages, averaged across five seeds with 300 instances per benchmark per seed, using strict predictions without rescue. Evaluation uses a maximum of 10 turns, with a required final submission on turn 10; comparison models run without reasoning at temperature 0.6.

File F1 measures overlap between predicted and reference files; line F1 measures overlap at the line level. These are **code-localization scores**, not SWE-bench issue-resolution rates. The model returns locations for a developer or downstream agent to inspect and edit.