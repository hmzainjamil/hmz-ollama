# hmz-ollama
Local LLM infrastructure — Ollama server, model configs, and GPU-optimized inference on macOS.

![models](https://img.shields.io/badge/models-7%2B-blue?style=flat&labelColor=555)
![cost](https://img.shields.io/badge/API_cost-zero-brightgreen?style=flat&labelColor=555)
![gpu](https://img.shields.io/badge/backend-Metal%20GPU-orange?style=flat&labelColor=555)
![platform](https://img.shields.io/badge/platform-Apple%20Silicon-lightgrey?style=flat&labelColor=555)
![license](https://img.shields.io/badge/license-MIT-blue?style=flat&labelColor=555)

[Concepts](#-concepts) · [Architecture](#️-architecture) · [Tips](#-tips-and-tricks-20) · [Kills](#️-startups--businesses) · [Stars](#star-history)

## 🧠 CONCEPTS

| Feature | Location | Description |
|---------|----------|-------------|
| [**LaunchAgent Config**](launchagent/ai.hmz.ollama.plist) | `launchagent/ai.hmz.ollama.plist` | Permanent Ollama server — KeepAlive daemon, starts on login [![permanent](https://img.shields.io/badge/mode-permanent-green?style=flat&labelColor=555)] |
| [**Llama 3 8B**](models/llama3) | `models/llama3/Modelfile` | General tasks, research, analysis — 4.7GB, ~40 tok/s on M1 |
| [**Llama 3.1 8B**](models/llama3.1) | `models/llama3.1/Modelfile` | Improved context handling — better for long documents |
| [**Llama 3.2 3B**](models/llama3.2) | `models/llama3.2/Modelfile` | Fastest local model — 3B params, instant responses |
| [**Mistral 7B**](models/mistral) | `models/mistral/Modelfile` | Balanced, fast — coding and reasoning |
| [**DeepSeek Coder**](models/deepseek-coder) | `models/deepseek-coder/Modelfile` | Specialized code generation — better than GPT-3.5 for code |
| [**Qwen 2.5 Coder**](models/qwen-coder) | `models/qwen-coder/Modelfile` | 7B code model — top-tier local coding assistant |
| [**Phi-3 Mini**](models/phi3) | `models/phi3/Modelfile` | Lightweight 3.8B — fastest for simple tasks |
| [**Tier 0 Integration**](scripts/tier0-ollama.sh) | `scripts/tier0-ollama.sh` | G0DM0D3 integration — auto-routes qualifying tasks to local Ollama |
| [**Health Check**](scripts/health-check.sh) | `scripts/health-check.sh` | Verifies Ollama is running + logs model availability |

### 🔥 Hot

| Feature | Location | Description |
|---------|----------|-------------|
| [**GPU Optimization**](configs/gpu-config.sh) | `configs/gpu-config.sh` | Metal backend tuning — max GPU layers, context size for Apple Silicon |
| [**Model Preloader**](scripts/preload.sh) | `scripts/preload.sh` | Warms models on startup — zero cold start latency for tier 0 routing |
| [**Hermes Mistral**](models/hermes) | `models/hermes/Modelfile` | Chat-optimized Mistral — best for conversational tasks |

## ⚙️ ARCHITECTURE

```
macOS LaunchAgent
    │
    ▼ on login
Ollama Server (:11434)
    │
    ├─ llama3:latest     ← default, general purpose
    ├─ llama3.1:latest   ← long-context documents
    ├─ mistral:latest    ← fast balanced
    ├─ deepseek-coder:latest ← code generation
    └─ phi3:latest       ← lightweight tasks

G0DM0D3 Router → Ollama API → localhost:11434
Claude Code → llm-burst → Ollama (Tier 0A priority)
```

| Model | Size | Speed | Best Use | RAM Required |
|-------|------|-------|----------|-------------|
| Phi-3 mini | 2.3GB | ★★★★★ | Quick tasks, summaries | 4GB |
| Llama 3.2 3B | 2.0GB | ★★★★★ | General, fast | 4GB |
| Llama 3 8B | 4.7GB | ★★★★ | Balanced general | 8GB |
| Mistral 7B | 4.1GB | ★★★★ | Code + reasoning | 8GB |
| Qwen 2.5 Coder | 4.7GB | ★★★★ | Code generation | 8GB |
| DeepSeek Coder | 4.7GB | ★★★ | Advanced code | 8GB |

## 💡 TIPS AND TRICKS (20)

[setup](#tips-setup) · [performance](#tips-performance) · [models](#tips-models) · [integration](#tips-integration)

<a id="tips-setup"></a>■ **Setup (5)**

| Tip | Source |
|-----|--------|
| `brew install ollama` then `launchctl load` plist — permanent server in 2 commands | [Ollama](https://ollama.ai) |
| Check RAM first: `sysctl hw.memsize` — need 2x model size free to run | [HMZ](https://github.com/hmzainjamil) |
| `ollama pull llama3` before first use — 4.7GB download, cached permanently | [HMZ](https://github.com/hmzainjamil) |
| LaunchAgent `KeepAlive=true` ensures server restarts after crash or sleep | [HMZ](https://github.com/hmzainjamil) |
| `OLLAMA_NUM_GPU=1` env var — forces GPU acceleration on Apple Silicon | [Ollama](https://github.com/ollama/ollama/blob/main/docs/gpu.md) |

<a id="tips-performance"></a>■ **Performance (5)**

| Tip | Source |
|-----|--------|
| Metal GPU backend gives 3-5x speedup vs CPU — always enable on Apple Silicon | [Ollama](https://github.com/ollama/ollama) |
| `OLLAMA_NUM_CTX=4096` — sufficient for most tasks, higher degrades speed | [HMZ](https://github.com/hmzainjamil) |
| Keep only 1-2 models loaded simultaneously — each 7B model uses ~5GB VRAM | [HMZ](https://github.com/hmzainjamil) |
| Phi-3 mini at 2.3GB leaves headroom for system — use when RAM is tight | [HMZ](https://github.com/hmzainjamil) |
| Preload models at startup via script — avoids 2-3s cold start on first request | [HMZ](https://github.com/hmzainjamil) |

<a id="tips-models"></a>■ **Model Selection (5)**

| Tip | Source |
|-----|--------|
| Llama 3 8B beats GPT-3.5 on most benchmarks — strong default choice | [Meta AI](https://ai.meta.com/llama) |
| Qwen 2.5 Coder 7B = best local code model below 10B — better than DeepSeek Coder 6.7B | [Qwen](https://github.com/QwenLM/Qwen2.5-Coder) |
| Mistral 7B fastest for reasoning tasks — lower latency than Llama 3 8B | [Mistral AI](https://mistral.ai) |
| Hermes Mistral: add `SYSTEM` prompt for chat — base Mistral needs prompting | [HMZ](https://github.com/hmzainjamil) |
| `ollama list` shows all pulled models + size — check before routing decisions | [HMZ](https://github.com/hmzainjamil) |

<a id="tips-integration"></a>■ **G0DM0D3 Integration (5)**

| Tip | Source |
|-----|--------|
| Ollama is Tier 0A — always checked first by G0DM0D3 router before cloud | [HMZ](https://github.com/hmzainjamil) |
| `curl -s http://localhost:11434/api/tags` — check server health from any script | [Ollama](https://github.com/ollama/ollama/blob/main/docs/api.md) |
| If Ollama offline, G0DM0D3 falls through to Groq automatically | [HMZ](https://github.com/hmzainjamil) |
| `llm-burst --models ollama 'prompt'` — forces local-only even with cloud available | [HMZ](https://github.com/hmzainjamil) |
| Monitor token/s via `ollama ps` — drop to smaller model if below 20 tok/s | [HMZ](https://github.com/hmzainjamil) |

## ☠️ STARTUPS / BUSINESSES

| Feature | Replaced |
|-|-|
| **Local LLM Server** | [Jan.ai](https://jan.ai), [LM Studio](https://lmstudio.ai), [Msty](https://msty.app), [GPT4All Desktop](https://nomic.ai/gpt4all) |
| **GPU-Optimized Inference** | [Together AI](https://together.ai), [Fireworks AI](https://fireworks.ai) — cloud GPU vs local GPU at $0 |
| **Model Management** | [Hugging Face Hub](https://huggingface.co), [Civitai](https://civitai.com) |
| **Always-On Server** | [Replicate](https://replicate.com), [Banana](https://banana.dev) — eliminates cold start costs |

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=hmzainjamil/hmz-ollama&type=Date)](https://star-history.com/#hmzainjamil/hmz-ollama&Date)
