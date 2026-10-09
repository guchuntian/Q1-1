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
license: apache-2.0
---

# ColGrep-Agent-2B

<div align="center">

<img src="media/banner.png" alt="ColGrep Agent 2B banner" width="600">

[![Website](https://img.shields.io/badge/LightOn-Website-blue?logo=google-chrome)](https://lighton.ai)
[![LinkedIn](https://img.shields.io/badge/LightOn-LinkedIn-0A66C2?logo=linkedin)](https://www.linkedin.com/company/lighton/)
[![X](https://img.shields.io/badge/@LightOnIO-X-black?logo=x)](https://x.com/LightOnIO)

📄 [Paper](https://arxiv.org/abs/2610.05099) | 📝 [Blog Post](https://huggingface.co/blog/lightonai/colgrep-agent/) | 💻 [GitHub](https://github.com/lightonai/next-plaid) | 🤗 [ColGrep-Agent-2B](https://huggingface.co/lightonai/colgrep-agent-2B)

</div>

**ColGrep-Agent-2B** is a lightweight agent designed to efficiently locate relevant code in a repository given a GitHub issue, bug report, feature request, or natural-language problem statement. It iteratively searches and reads the codebase using [ColGrep](https://github.com/lightonai/next-plaid) and read-only terminal commands, progressively refining its search over multiple turns based on retrieved results until it identifies the relevant files and line ranges needed to address the task.

The goal is to offload repository exploration from larger coding models. Instead of spending thousands of tokens searching through files before making a fix, a larger model can delegate this step to **ColGrep-Agent-2B** and receive a focused set of relevant code locations. Its small footprint enables fast, inexpensive, local inference, making it practical as a search sub-agent in coding workflows or as a standalone tool for developers navigating unfamiliar codebases.

## ColGREP

[ColGrep](https://github.com/lightonai/next-plaid) is a semantic code search tool for your terminal and coding agents, built on the NextPlaid search engine. It combines natural-language search with regex filtering to find code by both meaning and exact matches. ColGrep Agent adds an iterative search-and-read loop: it follows leads, inspects results, and selects the relevant lines.

## Quick start

Install ColGrep via Homebrew (macOS or Linux), download the [GGUF model](https://huggingface.co/lightonai/colgrep-agent-2B-GGUF), and start searching your codebase:

```bash
brew install lightonai/tap/colgrep  # Install ColGREP
colgrep --install-agent             # Download the 2B agent model
colgrep init                        # Index your repository
colgrep --agent "Where is the database connection pool configured?"
```

Inference runs locally through llama.cpp, with CPU support and GPU acceleration where available. See the [ColGrep documentation](https://github.com/lightonai/next-plaid/blob/main/colgrep/README.md#agent-mode---agent) for other installation methods and runtime settings.

## Training and harness

ColGREP-Agent-2B is built on MiniCPM5-2B and trained in two stages. We construct our training environments from **multilingual code repositories paired with issue descriptions and dedicated bug-fix commits**, using the pre-patch repository state as the search environment and the corresponding patches to derive gold file and line-level localization targets. We first apply a short cold-start supervised fine-tuning (SFT) phase to teach the model tool-call formatting, command ordering, argument and flag usage, and effective repository exploration strategies, with an emphasis on using ColGrep rather than traditional grep-based search. We then train the model for 500 steps using Group Relative Policy Optimization (GRPO) in a dedicated code-localization environment, rewarding accurate file and line identification while penalizing unnecessary exploration steps.

The harness provides three tools:

| Tool | Purpose |
|---|---|
| `colgrep` | Search code semantically, with optional regex and path filters. |
| `terminal` | Read files and inspect the repository through read-only commands. |
| `finish` | Submit relevant locations as `path/to/file:START-END`. |

The model searches and reads within a 10-turn budget, then returns the identified files and line numbers.

## Evaluation

We evaluate ColGREP-Agent-2B on [SWE-bench Lite](https://huggingface.co/datasets/SWE-bench/SWE-bench_Lite) and [Multi-SWE-bench Flash](https://huggingface.co/datasets/ByteDance-Seed/Multi-SWE-bench-flash), adapting both benchmarks into code-localization tasks following the methodology described in our [paper](https://arxiv.org/abs/2610.05099). Rather than generating patches or resolving issues end-to-end, the model is tasked with identifying the files and lines of code relevant to each issue. This allows us to evaluate localization accuracy independently of code generation.


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
      <td><b>ColGrep</b></td>
      <td><b>76.18</b></td>
      <td><b>19.92</b></td>
      <td><b>62.57</b></td>
      <td><b>15.93</b></td>
    </tr>
    <tr>
      <td>Qwen-3.5-27B</td>
      <td>ColGrep</td>
      <td>77.93</td>
      <td>15.71</td>
      <td>68.28</td>
      <td>16.63</td>
    </tr>
    <tr>
      <td>Gemma-4-26B-A4B</td>
      <td>ColGrep</td>
      <td>76.35</td>
      <td>13.34</td>
      <td>63.50</td>
      <td>14.75</td>
    </tr>
    <tr>
      <td>Qwen-3.8-27B</td>
      <td>ColGrep</td>
      <td>60.80</td>
      <td>7.70</td>
      <td>59.68</td>
      <td>11.50</td>
    </tr>
  </tbody>
</table>

All models use the ColGREP harness. File and line F1 measure overlap with reference files and lines, respectively. Scores are percentages averaged over five seeds (300 instances per benchmark per seed), using strict predictions without rescue, a 10-turn limit with mandatory final submission, and temperature 0.6 without reasoning for comparison models.

## Citation

```bibtex
@misc{colgrepagent_2026,
  title        = {ColGrep-Agent},
  author       = {Maxence Lasbordes and Raphael Sourty and Amélie Chatelain and Aarush Sinha},
  year         = {2026},
  howpublished = {\url{https://huggingface.co/blog/lightonai/colgrep-agent/}}
}
```
