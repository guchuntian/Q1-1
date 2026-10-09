---
library_name: transformers
pipeline_tag: text-generation
tags:
- code
- code-search
- colgrep
- agent
---

# ColGrep Agent 2B

The model behind [`colgrep --agent`](https://github.com/lightonai/next-plaid/tree/main/colgrep#agent-mode---agent): a 2B-parameter code-localization agent. Given a question about a repository, it searches the code with [colgrep](https://github.com/lightonai/next-plaid/tree/main/colgrep) (semantic + regex search), reads candidate files through a read-only terminal, and returns the files and line ranges that answer the question. It is small enough to run locally and works as a search sub-agent for a larger coding agent.

These are the BF16 safetensors weights. For local use, colgrep runs the GGUF build from [lightonai/colgrep-agent-2B-GGUF](https://huggingface.co/lightonai/colgrep-agent-2B-GGUF) on llama.cpp.

## Quick start

```bash
# Install colgrep and download the model (Homebrew, macOS / Linux)
brew install lightonai/tap/colgrep && colgrep --install-agent

# or with the shell installer
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/lightonai/next-plaid/releases/latest/download/colgrep-installer.sh | sh && "${CARGO_HOME:-$HOME/.cargo}/bin/colgrep" --install-agent

# Index a repository, then ask
cd /path/to/project
colgrep init
colgrep --agent "where are retries configured"
```

The answer is a list of `file:start-end` locations on stdout. See the [GGUF model card](https://huggingface.co/lightonai/colgrep-agent-2B-GGUF) for Windows and Cargo installs, an example session, coding-agent integration and the available quantizations.

## Serving these weights

colgrep can send the agent's prompts to any OpenAI-compatible server through `/v1/completions`:

```bash
vllm serve lightonai/colgrep-agent-2B --served-model-name lightonai/colgrep-agent-2B-GGUF --port 8000
colgrep settings --agent-endpoint http://localhost:8000/v1
```

`--served-model-name` matches the model name colgrep sends by default.

The model was trained inside colgrep's agent harness: system prompt, tool schemas (`colgrep`, `terminal`, `finish`), chat template and observation format. colgrep reproduces that harness byte for byte, so using it through colgrep is the intended way to run the model.

## Training

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
