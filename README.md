
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

📄 [Paper](https://arxiv.org/abs/2610.05099) | 📝 [Blog Post](https://huggingface.co/blog/lightonai/colgrep-agent/) | 💻 [GitHub](https://github.com/lightonai/next-plaid) | 🤗 [Model](https://huggingface.co/lightonai/colgrep-agent-2B) | 📦 [GGUF](https://huggingface.co/lightonai/colgrep-agent-2B-GGUF)

</div>

**ColGrep-Agent-2B** is a lightweight code-localization agent designed to identify the files and line ranges relevant to a GitHub issue, bug report, feature request, or natural-language question about a repository.

Given a question such as *"Where are retries configured?"* or *"Why does the export crash on empty files?"*, the agent iteratively searches the repository using [ColGrep](https://github.com/lightonai/next-plaid/tree/main/colgrep), reads candidate files through a read-only terminal, and returns the precise code locations relevant to the task.

The model powers [`colgrep --agent`](https://github.com/lightonai/next-plaid/tree/main/colgrep#agent-mode---agent) and is designed to **offload repository exploration from larger coding agents**. Rather than consuming thousands of tokens searching through files before making a fix, a larger model can delegate code localization to ColGrep-Agent-2B and receive a focused list of relevant locations.

Its small footprint enables practical local inference, with GPU acceleration when available, without requiring Python, a separate inference server, or an API key.

<p align="center">
  <img width="820" src="media/colgrep-demo.gif" alt="colgrep: a semantic search, then colgrep --agent"/>
</p>


## 🚀 Quick start

### 1. Install ColGrep and the model

**Homebrew (macOS / Linux)**

```bash
brew install lightonai/tap/colgrep && colgrep --install-agent
```

**Shell installer (macOS / Linux)**

```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/lightonai/next-plaid/releases/latest/download/colgrep-installer.sh | sh && "${CARGO_HOME:-$HOME/.cargo}/bin/colgrep" --install-agent
```

**Windows (PowerShell)**

```powershell
irm https://github.com/lightonai/next-plaid/releases/latest/download/colgrep-installer.ps1 | iex
colgrep --install-agent
```

**Cargo (from source)**

```bash
cargo install colgrep && colgrep --install-agent
```

The `colgrep --install-agent` command downloads the default `colgrep-agent-2B-Q8_0.gguf` model (approximately 2.5 GB), verifies that it loads, and prepares the prompt cache to accelerate the first inference.

Installation is required only once. No Python environment, external inference server, or API key is necessary.

### 2. Index a repository

```bash
cd /path/to/project
colgrep init
```

After the initial indexing, subsequent searches automatically detect modified files and update the index before answering.

### 3. Ask a question

```bash
colgrep --agent "where does the agent decide which GPU backend llama.cpp uses"
```

The following example shows a real session on ColGrep's own source code, running on an M3 Pro with Metal acceleration. Individual tool results have been omitted for readability.

```text
$ colgrep --agent "where does the agent decide which GPU backend llama.cpp uses"
◆ colgrep agent  · colgrep-agent-2B-Q8_0.gguf (GPU)
❯ where does the agent decide which GPU backend llama.cpp uses

  ● colgrep   llama.cpp GPU backend
  ● terminal  cat -n agent/src/llm/llama.rs
  ● terminal  cat -n agent/src/llm/server.rs
  ● terminal  cat -n agent/src/engine.rs
  ● terminal  cat -n agent/src/llm/llama.rs | sed -n '80,150p'
  ● terminal  cat -n agent/src/llm/server.rs | sed -n '65,130p'
  ● colgrep   find_gpu
  ● terminal  cat -n agent/src/llm/server.rs | sed -n '1,100p'
  ● finish    agent/src/llm/server.rs:103-105, agent/src/llm/server.rs:65-69, …

✓ 5 locations · 9 turns · 2 searches · 6 terminal commands · 21.7s

agent/src/llm/server.rs:103-105
agent/src/llm/server.rs:65-69
agent/src/llm/llama.rs:101-101
agent/src/llm/llama.rs:115-116
agent/src/engine.rs:57-67
```

The output follows the `path/to/file:START-END` format, making it directly usable by developers, scripts, and other coding agents.

Agent progress is printed to **stderr only when running in a terminal**, while stdout contains only the resulting code locations. This separation makes the command easy to compose with other tools.

**Additional usage examples**

Scope the search to a directory and display the matching code lines:

```bash
colgrep --agent "crash when the config file is empty" ./backend -c
```

Return structured JSON for downstream applications:

```bash
colgrep --agent --json "where are retries configured"
```

## 🤖 Integration with coding agents

ColGrep can be integrated into existing coding-agent workflows, allowing larger models to delegate repository exploration to a smaller specialized model.

Install the corresponding hooks and skills:

```bash
colgrep --install-claude-code   # Claude Code
colgrep --install-codex         # Codex
colgrep --install-opencode      # OpenCode
```

For example, once configured, Claude Code can delegate open-ended questions such as *"Where is this behavior implemented?"* to `colgrep --agent`, receiving exact file paths and line ranges instead of performing every exploratory search itself.

This division of labor allows larger coding agents to reserve more of their context and computation for understanding issues, planning changes, and generating patches.

## 🔎 ColGrep

[ColGrep](https://github.com/lightonai/next-plaid) is a semantic code search tool for terminals and coding agents, built on the NextPlaid search engine. It combines natural-language search with regex filtering to find code by both meaning and exact matches.

**ColGrep-Agent-2B extends this search functionality with an iterative search-and-read loop.** Rather than relying on a single search query, the agent follows leads, inspects candidate files, refines its searches, and identifies relevant code locations over multiple turns.


## 🧠 Training and harness

**ColGREP-Agent-2B** is built on MiniCPM5-2B and trained in two stages: a short supervised fine-tuning (SFT) cold start followed by reinforcement learning (RL) using DAPO.

We construct multilingual code-localization environments from [nebius/SWE-rebench](https://huggingface.co/datasets/nebius/SWE-rebench), [internlm/SWE-Fixer-Train-110K](https://huggingface.co/datasets/internlm/SWE-Fixer-Train-110K), and additional scraped repositories covering multiple programming languages. Each training instance pairs a repository with an issue description and a dedicated bug-fix commit. We use the pre-patch repository state as the search environment and derive gold file- and line-level localization targets from the corresponding patch. These data sources are used across both the SFT and RL stages.

We first perform a short cold-start SFT phase using traces generated and filtered with **Qwen3.5-27B as a teacher model**. This stage teaches tool-call formatting, command ordering, argument and flag usage, and effective repository exploration strategies, with an emphasis on semantic search through ColGrep rather than traditional grep-based search. We then train for **500 steps using DAPO** in a dedicated code-localization environment, rewarding accurate file- and line-level identification while penalizing unnecessary exploration steps.

The model operates in a repository exploration environment with access to three tools:

| Tool | Purpose |
|---|---|
| `colgrep` | Search code semantically, with optional regex and path filters. |
| `terminal` | Read files and inspect the repository through read-only commands. |
| `finish` | Submit relevant locations as `path/to/file:START-END`. |

The agent has a **10-turn budget** to search and inspect the repository before submitting its final prediction as a set of relevant file paths and line ranges.


## 📊 Evaluation

![ColGREP performance vs. model size](media/colgrep_performance_vs_model_size.png)

We evaluate ColGREP-Agent-2B on [SWE-bench Lite](https://huggingface.co/datasets/SWE-bench/SWE-bench_Lite) and [Multi-SWE-bench Flash](https://huggingface.co/datasets/ByteDance-Seed/Multi-SWE-bench-flash), adapting both benchmarks into code-localization tasks following the methodology described in our [paper](https://arxiv.org/abs/2610.05099). Rather than generating patches or resolving issues end-to-end, the model is tasked with identifying the files and lines of code relevant to each issue. This allows us to evaluate localization accuracy independently of code generation.

<table>
  <thead>
    <tr>
      <th rowspan="2">Model</th>
      <th rowspan="2">Harness</th>
      <th colspan="2">SWE-bench Lite</th>
      <th colspan="2">Multi-SWE-bench Flash</th>
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
      <td><b>lightonai/colgrep-agent-2B</b></td>
      <td><b>ColGrep</b></td>
      <td><b>76.18</b></td>
      <td><b>19.92</b></td>
      <td><b>62.57</b></td>
      <td><b>15.93</b></td>
    </tr>
    <tr>
      <td>openbmb/MiniCPM5-2B</td>
      <td>ColGrep</td>
      <td>66.72</td>
      <td>9.62</td>
      <td>52.01</td>
      <td>11.31</td>
    </tr>
    <tr>
      <td>google/gemma-4-31B-it</td>
      <td>ColGrep</td>
      <td>79.93</td>
      <td>17.08</td>
      <td>68.30</td>
      <td>19.14</td>
    </tr>
    <tr>
      <td>Qwen/Qwen3.5-27B</td>
      <td>ColGrep</td>
      <td>77.93</td>
      <td>15.71</td>
      <td>68.28</td>
      <td>16.63</td>
    </tr>
    <tr>
      <td>google/gemma-4-26B-A4B-it</td>
      <td>ColGrep</td>
      <td>76.35</td>
      <td>13.34</td>
      <td>63.50</td>
      <td>14.75</td>
    </tr>
    <tr>
      <td>Qwen/Qwen3.8-27B</td>
      <td>ColGrep</td>
      <td>60.80</td>
      <td>7.70</td>
      <td>59.68</td>
      <td>11.50</td>
    </tr>
    <tr>
      <td>deepseek-ai/DeepSeek-V4-Flash</td>
      <td>ColGrep</td>
      <td>74.04</td>
      <td>11.36</td>
      <td>64.26</td>
      <td>13.64</td>
    </tr>
  </tbody>
</table>

All models use the ColGREP harness. File and line F1 measure overlap with reference files and lines, respectively. Scores are percentages averaged over five seeds (300 instances per benchmark per seed), using strict predictions without rescue, a 10-turn limit with mandatory final submission, and temperature 0.6 without reasoning for comparison models.


## 📚 Citation

```bibtex
@misc{colgrepagent_2026,
  title        = {ColGrep-Agent},
  author       = {Lasbordes, Maxence and Sourty, Raphael and Sinha, Aarush and Chatelain, Amélie},
  year         = {2026},
  howpublished = {\url{https://huggingface.co/blog/lightonai/colgrep-agent/}}
}
```
