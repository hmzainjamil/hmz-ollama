# hmz-ollama
Ollama local LLM setup — permanent LaunchAgent daemon serving llama3, CodeLlama, and Mistral locally at zero cost, zero latency, zero data exposure.

![models](https://img.shields.io/badge/models-3_loaded-blue?style=flat&labelColor=555) ![daemon](https://img.shields.io/badge/daemon-LaunchAgent_permanent-green?style=flat&labelColor=555) ![cost](https://img.shields.io/badge/cost-zero-brightgreen?style=flat&labelColor=555) ![privacy](https://img.shields.io/badge/privacy-fully_local-orange?style=flat&labelColor=555)

[Concepts](#-concepts) · [Hot](#-hot) · [Models](#️-model-inventory) · [Tips](#-tips-and-tricks-20) · [Replaced](#️-startups--businesses) · [Stars](#star-history)

---

## 🧠 CONCEPTS

| Feature | Location | Description |
|---------|----------|-------------|
| [**LaunchAgent daemon**](launchagent/ai.hmz.ollama.plist) | `~/Library/LaunchAgents/ai.hmz.ollama.plist` | `RunAtLoad=true` `KeepAlive=true` — Ollama always running on GPU, survives crashes and reboots |
| [**REST API**](http://localhost:11434) | `http://localhost:11434` | OpenAI-compatible API — any tool supporting OpenAI format works with Ollama |
| [**llama3:latest**](https://ollama.com/library/llama3) | `ollama pull llama3` | Default general model — 8B parameters, best balance of speed and quality for Tier 0 |
| [**codellama:7b**](https://ollama.com/library/codellama) | `ollama pull codellama:7b` | Code generation and debugging — Meta's code-specialized model |
| [**G0DM0D3 integration**](https://github.com/hmzainjamil/hmz-g0dm0d3) | `CLAUDE.md` | Ollama = first in Tier 0 routing — called before any cloud API for suitable tasks |
| [**llm-burst integration**](https://github.com/hmzainjamil/claude-ai-system) | `~/.claude/bin/llm-burst` | Ollama included in 8-model parallel burst — local response always in the mix |

### 🔥 Hot

| Feature | Location | Description |
|---------|----------|-------------|
| [**GPU acceleration**](launchagent/) | `OLLAMA_GPU_OVERHEAD=0` | Metal GPU inference on macOS — 30-60 tok/sec vs 8-12 tok/sec CPU |
| [**Low-RAM fallback**](CLAUDE.md) | `CLAUDE.md` | When RAM < 2GB free, skip Ollama in llm-burst — use Groq cloud instead |
| [**Model hot-swap**](http://localhost:11434/api/tags) | `GET /api/tags` | Switch models without restarting Ollama — `ollama pull <new-model>` while running |

---

## ⚙️ MODEL INVENTORY

| Model | Size | Strength | When to Use |
|-------|------|----------|-------------|
| `llama3:latest` | 8B | General reasoning, chat, Q&A | Default for all general tasks |
| `codellama:7b` | 7B | Code generation, debugging | Any code generation task |
| `mistral:7b` | 7B | Fast general purpose | Speed-critical general tasks |
| `phi3:mini` | 3.8B | Lightweight, fast | Low-RAM fallback |
| `llama3.1:8b` | 8B | Improved context handling | Long-context tasks |
| `deepseek-coder:6.7b` | 6.7B | Code specialist | Complex code tasks (pull when needed) |

---

## 💡 TIPS AND TRICKS (20)

[Setup](#tips-setup) · [Performance](#tips-perf) · [Models](#tips-models) · [Integration](#tips-int) · [Debug](#tips-debug)

<a id="tips-setup"></a>■ **Setup (4)**

| Tip | Source |
|-----|--------|
| LaunchAgent needs both `KeepAlive=true` AND `RunAtLoad=true` to be permanent | [macOS SOP](launchagent/) |
| Set `OLLAMA_HOST=0.0.0.0:11434` in plist env to allow LAN access | [Networking](launchagent/) |
| `OLLAMA_KEEP_ALIVE=24h` keeps models in GPU memory — avoids reload delay | [Performance](launchagent/) |
| `OLLAMA_GPU_OVERHEAD=0` on M-series Macs for maximum GPU allocation | [Apple Silicon](launchagent/) |

<a id="tips-perf"></a>■ **Performance (5)**

| Tip | Source |
|-----|--------|
| 8B models on M2 Pro with 32GB RAM = 40-60 tok/sec — matches cloud latency | [Benchmark](https://ollama.com) |
| `ollama ps` to see loaded models and GPU/RAM usage in real-time | [Monitoring](https://ollama.com/docs) |
| First token delay (cold start) = model not in GPU memory — use `OLLAMA_KEEP_ALIVE` | [Latency fix](launchagent/) |
| Low RAM (<2GB)? Only run `phi3:mini` — other models thrash swap and freeze | [RAM rule](CLAUDE.md) |
| Concurrent requests: Ollama handles one at a time — `llm-burst` routes Ollama as one of 8 | [Architecture](../hmz-g0dm0d3/) |

<a id="tips-models"></a>■ **Model Selection (4)**

| Tip | Source |
|-----|--------|
| General tasks → `llama3:latest` (best default) | [G0DM0D3 routing](../hmz-g0dm0d3/) |
| Code → `codellama:7b` or `deepseek-coder:6.7b` (pull on demand) | [G0DM0D3 routing](../hmz-g0dm0d3/) |
| Offline/private → any local model (nothing leaves the machine) | [Privacy principle](CLAUDE.md) |
| `ollama pull <model>` is instant if already cached, ~minutes if new | [CLI](https://ollama.com) |

<a id="tips-int"></a>■ **Integration (4)**

| Tip | Source |
|-----|--------|
| Any OpenAI-compatible tool works: point base_url to `http://localhost:11434/v1` | [Compatibility](https://ollama.com/docs) |
| `llm-burst` calls Ollama as one parallel model — skip with `--no-ollama` on low RAM | [llm-burst](../claude-ai-system/automations/bin/llm-burst) |
| G0DM0D3 routes suitable tasks to Ollama first — cloud only as fallback | [G0DM0D3](../hmz-g0dm0d3/) |
| GPT4All and Ollama can run simultaneously — different ports, no conflict | [Multi-local](../hmz-personal-ai-infrastructure/) |

<a id="tips-debug"></a>■ **Debug (3)**

| Tip | Source |
|-----|--------|
| Ollama not responding: `launchctl restart ai.hmz.ollama` | [Runbook](launchagent/) |
| `curl http://localhost:11434/api/tags` — health check from CLI | [API ref](https://ollama.com/docs) |
| Model not loading: check `~/Library/Logs/ollama-error.log` for OOM errors | [Debug](launchagent/) |

---

## ☠️ STARTUPS / BUSINESSES

| Feature | Replaced |
|-|-|
| **Local LLM inference** | [Replicate](https://replicate.com), [Together AI](https://together.ai), [Fireworks AI](https://fireworks.ai) — cloud billing for local-capable work |
| **Privacy-first inference** | [ChatGPT API](https://openai.com/api) — sends all data to OpenAI |
| **Always-on daemon** | Manual `ollama serve` before each session |
| **Zero-cost inference** | Groq free tier limits · Gemini rate limits — Ollama has none |
| **OpenAI-compatible API** | Custom API adapters for each local model runtime |

---

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=hmzainjamil/hmz-ollama&type=Date)](https://star-history.com/#hmzainjamil/hmz-ollama&Date)