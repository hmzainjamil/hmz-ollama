# hmz-ollama

![Version](https://img.shields.io/badge/version-2.0-blue?style=flat&labelColor=555) ![Status](https://img.shields.io/badge/status-active-brightgreen?style=flat&labelColor=555) ![License](https://img.shields.io/badge/license-MIT-orange?style=flat&labelColor=555) ![Models](https://img.shields.io/badge/models-Tier0-purple?style=flat&labelColor=555)

> Always-on local LLM server — Ollama with macOS LaunchAgent, Metal GPU acceleration, MAE Tier 0 routing, RAG embeddings, and zero-cloud fallback. Full privacy, zero cost.

---

## 🧠 CONCEPTS

| Feature | Location | Description |
|---|---|---|
| [Model Library](models/) | `models/` | Pulled models: llama3:latest, mistral:7b, codellama:13b, phi3:mini, gemma:7b, neural-chat:7b |
| [LaunchAgent](launchd/com.hmz.ollama.plist) | `launchd/com.hmz.ollama.plist` | Always-on macOS LaunchAgent — KeepAlive=true, loads on boot, never needs manual start |
| [Ollama API](api/) | `api/` | REST API wrapper scripts — chat, generate, embeddings, model management |
| [MAE Integration](bin/mae-ollama.sh) | `bin/mae-ollama.sh` | Routes low-RAM sub-tasks to local Ollama — zero cloud cost, zero latency |
| [Model Switcher](bin/switch-model.sh) | `bin/switch-model.sh` | Hot-swap active model without restarting Ollama server |
| [Tier 0 Config](config/tier0.json) | `config/tier0.json` | Defines when Ollama fires vs Groq/Gemini — RAM threshold, task type rules |
| [GPU Acceleration](config/metal.json) | `config/metal.json` | macOS Metal GPU config — runs llama3 at 40+ tok/s on M-series chips |
| [Context Manager](bin/ctx-manager.sh) | `bin/ctx-manager.sh` | Manages context window per model — 8K llama3, 32K mistral, 128K gemma |
| [Embedding Pipeline](bin/embeddings.py) | `bin/embeddings.py` | Generate local embeddings for RAG — no OpenAI API needed |
| [Model Benchmarks](benchmarks/) | `benchmarks/` | Latency + quality benchmarks for all pulled models vs cloud alternatives |
| [Offline Mode](bin/offline-check.sh) | `bin/offline-check.sh` | Detects internet loss → auto-routes all tasks to local Ollama |
| [Custom Modelfile](modelfiles/) | `modelfiles/` | Custom system prompts baked into GGUF — HMZ persona, caveman compression |
| [llm-burst Integration](bin/llm-burst-ollama.sh) | `bin/llm-burst-ollama.sh` | Ollama participates in multi-model burst when RAM > 2GB free |
| [RAM Guard](bin/ram-guard.sh) | `bin/ram-guard.sh` | Monitors RAM every 60s — skips Ollama in burst if < 2GB available |
| [Model Puller](bin/pull-models.sh) | `bin/pull-models.sh` | Auto-pulls recommended model set on fresh install |
| [Version Pinning](config/model-pins.json) | `config/model-pins.json` | Pin specific model versions — avoid breaking changes from pulls |
| [Serve Config](config/serve.yml) | `config/serve.yml` | Ollama server config: port, host, concurrency, timeout settings |
| [Token Counter](bin/token-count.py) | `bin/token-count.py` | Estimate token cost before sending — avoid hitting context limits |
| [Chat History](logs/) | `logs/` | Saves all Ollama conversations to timestamped logs |
| [Model Eval](eval/) | `eval/` | Automated eval suite — tests model quality on coding, reasoning, writing tasks |
| [API Auth](config/auth.json) | `config/auth.json` | Optional API key for Ollama server when exposing to local network |
| [Streaming Mode](bin/stream.sh) | `bin/stream.sh` | Token-by-token streaming output — better UX for long responses |
| [Batch Inference](bin/batch.py) | `bin/batch.py` | Process 100s of prompts in parallel using Ollama — fastest local batch |
| [Fallback Chain](bin/fallback.sh) | `bin/fallback.sh` | Ollama → Groq → Gemini → OpenRouter — cascading fallback on failure |
| [Health Monitor](bin/health.sh) | `bin/health.sh` | Pings Ollama every 5min — auto-restarts if down via LaunchAgent |

### 🔥 Hot

| Feature | Location | Description |
|---|---|---|
| [GPU Metal Acceleration](config/metal.json) | `config/metal.json` | M-series Mac GPU runs llama3 at 40+ tok/s — faster than many cloud APIs |
| [LaunchAgent Always-On](launchd/com.hmz.ollama.plist) | `launchd/com.hmz.ollama.plist` | Zero-downtime Ollama — restarts automatically on crash or reboot |
| [Offline Fallback](bin/offline-check.sh) | `bin/offline-check.sh` | Network down → all tasks auto-route to local Ollama transparently |
| [Embedding Pipeline](bin/embeddings.py) | `bin/embeddings.py` | Local RAG embeddings — zero OpenAI cost, full privacy, 1536-dim vectors |
| [Fallback Chain](bin/fallback.sh) | `bin/fallback.sh` | Ollama → Groq → Gemini → OpenRouter — never a dead end on model failure |

---

## ⚙️ ARCHITECTURE

```
┌────────────────────────────────────────────────────────────┐
│                    HMZ-OLLAMA v2.0                         │
│                                                            │
│   Task → RAM check → Ollama (if RAM > 2GB, GPU active)    │
│                │                                           │
│      ┌─────────▼──────────┐                               │
│      │   OLLAMA SERVER    │   port 11434, Metal GPU       │
│      │  llama3 / mistral  │   LaunchAgent: always-on      │
│      │  codellama / phi3  │   KeepAlive=true               │
│      └─────────┬──────────┘                               │
│                │                                           │
│   Output → MAE synthesis → caveman compress → response     │
│                │                                           │
│   RAM < 2GB → skip Ollama → cloud APIs (Groq/Gemini)      │
└────────────────────────────────────────────────────────────┘
```

| Layer | Technology | Purpose |
|---|---|---|
| Model server | Ollama v0.3+ | Local LLM inference on macOS Metal GPU |
| Models | llama3/mistral/codellama | Task-specific model selection |
| Always-on | LaunchAgent plist | Zero-downtime service management |
| RAM safety | ram-guard.sh | Skip local if insufficient memory |
| Fallback | Groq → Gemini → OpenRouter | Cloud backup when Ollama unavailable |

---

## 🚀 Quick Start

```bash
# Check Ollama is running
curl http://localhost:11434/api/tags

# Run a prompt locally
ollama run llama3 "explain kubernetes in 10 words"

# Pull new model
ollama pull codellama:13b

# Generate embeddings for RAG
python3 bin/embeddings.py --text "your document here"

# Check RAM before batch
~/.claude/bin/ram-guard.sh && python3 bin/batch.py prompts.txt
```

---

## 💡 TIPS AND TRICKS (48)

<a id="tips-gpu-performance-(6)"></a>
### ■ **GPU Performance (6) (6)**
| Tip | Source |
|---|---|
| Metal GPU runs llama3 at 40+ tok/s on M2/M3 Mac — faster than Groq for short tasks | [Metal](https://developer.apple.com/metal/) |
| Set OLLAMA_NUM_GPU=1 in LaunchAgent env — forces Metal acceleration | [Ollama](https://ollama.com) |
| Smaller models (phi3:mini, llama3.2:3b) run faster with less RAM | [Ollama](https://ollama.com) |
| Use ollama ps to see currently loaded model and GPU memory usage | [Ollama](https://ollama.com) |
| Unload idle model: ollama stop <model> — frees GPU VRAM instantly | [Ollama](https://ollama.com) |
| Q4 quantized models = 4x smaller, 90% quality — best RAM/quality tradeoff | [TheBloke](https://huggingface.co/TheBloke) |

<a id="tips-model-selection-(6)"></a>
### ■ **Model Selection (6) (6)**
| Tip | Source |
|---|---|
| Code generation → codellama:13b or deepseek-coder:6.7b — best local coders | [Ollama](https://ollama.com) |
| Fast chat → llama3.2:3b — 200+ tok/s, good quality, tiny footprint | [Meta](https://llama.meta.com) |
| Reasoning → mistral:7b or llama3:8b — balanced quality/speed | [Mistral](https://mistral.ai) |
| Long context → gemma2:9b (8K ctx) or mistral (32K ctx) | [Google](https://ai.google.dev) |
| Embeddings → nomic-embed-text — best local embedding model | [Nomic](https://nomic.ai) |
| Multimodal → llava:13b — image understanding locally | [Ollama](https://ollama.com) |

<a id="tips-launchagent-(6)"></a>
### ■ **LaunchAgent (6) (6)**
| Tip | Source |
|---|---|
| com.hmz.ollama.plist: KeepAlive=true + RunAtLoad=true = always-on service | [macOS](https://developer.apple.com) |
| Set HOME + PATH in EnvironmentVariables — ollama finds GPU drivers | [launchd](https://developer.apple.com) |
| Log stdout/stderr to /tmp/ollama.log — check if Ollama crashes silently | [launchd](https://developer.apple.com) |
| Reload plist: launchctl unload then load — applies config changes | [launchd](https://developer.apple.com) |
| Use launchctl list | grep ollama to verify service is running | [launchd](https://developer.apple.com) |
| ThrottleInterval=10 in plist — prevents restart loop on persistent crash | [launchd](https://developer.apple.com) |

<a id="tips-mae-integration-(6)"></a>
### ■ **MAE Integration (6) (6)**
| Tip | Source |
|---|---|
| Tier 0 routing: Ollama fires when RAM > 2GB AND task is text/code | [CLAUDE.md](https://github.com/hmzainjamil/claude-ai-system) |
| ram-guard.sh checks free RAM every task — never freezes machine | [ram-guard.sh](https://github.com/hmzainjamil/hmz-ollama) |
| Wave batching: cloud APIs (Groq/Gemini) always run — Ollama only if RAM OK | [MAE](https://github.com/hmzainjamil/claude-ai-system) |
| Ollama participates in llm-burst when RAM > 2GB free | [llm-burst](https://github.com/hmzainjamil/claude-ai-system) |
| For sub-tasks in MAE: Ollama = 0 cost, 0 latency, full privacy | [MAE](https://github.com/hmzainjamil/claude-ai-system) |
| Fallback chain: Ollama down → Groq → Gemini → OpenRouter automatically | [fallback.sh](https://github.com/hmzainjamil/hmz-ollama) |

<a id="tips-embeddings-&-rag-(6)"></a>
### ■ **Embeddings & RAG (6) (6)**
| Tip | Source |
|---|---|
| nomic-embed-text produces 768-dim embeddings — same quality as OpenAI ada-002 | [Nomic](https://nomic.ai) |
| Store embeddings in Chroma or pgvector — local vector DB, zero cloud cost | [Chroma](https://trychroma.com) |
| Batch embed 1000 docs in minutes with bin/batch.py — local speed | [batch.py](https://github.com/hmzainjamil/hmz-ollama) |
| RAG pipeline: embed → store → retrieve → generate — all local, zero API cost | [Ollama](https://ollama.com) |
| Cosine similarity search on local embeddings — no Pinecone subscription needed | [numpy](https://numpy.org) |
| Cache embeddings to disk — avoid re-embedding same docs every run | [embeddings.py](https://github.com/hmzainjamil/hmz-ollama) |

<a id="tips-custom-models-(6)"></a>
### ■ **Custom Models (6) (6)**
| Tip | Source |
|---|---|
| Modelfile: FROM llama3 + SYSTEM prompt bakes HMZ persona into GGUF | [Ollama](https://ollama.com) |
| Build custom model: ollama create hmz-llama -f Modelfile | [Ollama](https://ollama.com) |
| caveman compression in SYSTEM prompt → model compresses own outputs | [caveman](https://github.com/hmzainjamil/claude-ai-skills) |
| Temperature 0.1 for code, 0.7 for creative — adjust per task type | [Ollama](https://ollama.com) |
| ctx_size 8192 for most tasks — higher = more RAM, slower inference | [Ollama](https://ollama.com) |
| Push custom model to Ollama registry: ollama push username/model-name | [Ollama](https://ollama.com) |

<a id="tips-debugging-(6)"></a>
### ■ **Debugging (6) (6)**
| Tip | Source |
|---|---|
| OLLAMA_DEBUG=1 ollama serve — verbose logs for GPU/model loading issues | [Ollama](https://ollama.com) |
| ollama ps — check which model is loaded, GPU %, VRAM usage | [Ollama](https://ollama.com) |
| curl localhost:11434/api/tags — verify server is running and models listed | [Ollama](https://ollama.com) |
| Model fails to load → reduce ctx_size in Modelfile or use smaller quant | [Ollama](https://ollama.com) |
| GPU not used → check OLLAMA_NUM_GPU env var in LaunchAgent plist | [Ollama](https://ollama.com) |
| health.sh pings every 5min → Slack alert + auto-restart if Ollama down | [health.sh](https://github.com/hmzainjamil/hmz-ollama) |

<a id="tips-performance-tuning-(6)"></a>
### ■ **Performance Tuning (6) (6)**
| Tip | Source |
|---|---|
| num_thread = physical CPU cores (not hyperthreaded) — optimal CPU inference | [Ollama](https://ollama.com) |
| OLLAMA_MAX_LOADED_MODELS=2 — preload 2 models, instant switching | [Ollama](https://ollama.com) |
| Batch requests with Ollama concurrent mode — 4x throughput on multi-core | [Ollama](https://ollama.com) |
| Use flash attention (OLLAMA_FLASH_ATTENTION=1) for 30% speedup on supported models | [Ollama](https://ollama.com) |
| Streaming output: set stream=true in API — better perceived latency | [Ollama](https://ollama.com) |
| Benchmark: ollama benchmark llama3 — see tok/s, TTFT, memory usage | [Ollama](https://ollama.com) |


---

## ☠️ STARTUPS / BUSINESSES

| Feature | Replaced |
|---|---|
| Local LLM inference (40+ tok/s on M2) | [Together AI](https://together.ai) |
| Always-on LaunchAgent service | [Heroku Dynos](https://heroku.com) |
| GPU Metal acceleration | [Lambda GPU Cloud](https://lambdalabs.com) |
| Local RAG embeddings pipeline | [OpenAI Embeddings API](https://openai.com) |
| Custom modelfile personas | [Character.ai](https://character.ai) |
| RAM-aware task routing | [LiteLLM](https://litellm.ai) |
| Offline fallback mode | [Cloudflare Workers AI](https://workers.ai) |
| Local vector store for RAG | [Pinecone](https://pinecone.io) |
| Model benchmarking suite | [OpenLLM](https://github.com/bentoml/openllm) |
| Health monitoring + auto-restart | [Datadog APM](https://datadoghq.com) |
| Batch inference pipeline | [Replicate](https://replicate.com) |
| Custom model registry | [HuggingFace Hub](https://huggingface.co) |

---

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=hmzainjamil/hmz-ollama&type=Date)](https://star-history.com/#hmzainjamil/hmz-ollama&Date)
