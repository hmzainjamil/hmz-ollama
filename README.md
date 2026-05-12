# hmz-ollama
> Permanent GPU-accelerated local LLM server — Ollama on macOS Metal, LaunchAgent, zero-cost inference forever.

[![ollama](https://img.shields.io/badge/Ollama-GPU--Metal-blue?style=flat&labelColor=555)](https://ollama.ai)
[![models](https://img.shields.io/badge/models-llama3--llama3.1-green?style=flat&labelColor=555)](.)
[![launchagent](https://img.shields.io/badge/LaunchAgent-permanent-orange?style=flat&labelColor=555)](launchagents/)
[![tier0](https://img.shields.io/badge/tier0-zero--cost-purple?style=flat&labelColor=555)](.)
[![license](https://img.shields.io/badge/license-MIT-lightgrey?style=flat&labelColor=555)](LICENSE)

[concepts](#concepts) · [architecture](#architecture) · [tips](#tips) · [startups](#startups) · [star](#star)

---

## 🧠 CONCEPTS <a id="concepts"></a>

| Feature | Location | Description |
|---|---|---|
| [**Ollama LaunchAgent**](launchagents/ai.hmz.ollama.plist) | `launchagents/` | KeepAlive=true plist — Ollama stays running across reboots permanently |
| [**llama3:latest**](models/) | `models/` | Default model — 8B parameters, Metal GPU, ~1.5s first token on M-series |
| [**llm-burst integration**](.) | `~/.claude/bin/llm-burst` | `call_ollama()` — local model in burst pipeline after cloud wave |
| [**RAM guard**](.) | `~/.claude/bin/mae` | Ollama skipped in burst if RAM < 2GB — runs after cloud wave when RAM safe |
| [**Binary path**](.) | `/Applications/Ollama.app/Contents/Resources/ollama` | Absolute path used in all scripts — not dependent on PATH |
| [**Metal acceleration**](.) | `config/` | macOS Metal GPU backend — 4-5x faster than CPU-only |

### 🔥 Hot

| Feature | Location | Description |
|---|---|---|
| [**Multi-model pull script**](scripts/pull-models.sh) | `scripts/` | Pulls llama3, mistral, codellama, phi3 in one command |
| [**Ollama OpenAI API**](.) | `http://localhost:11434/v1` | OpenAI-compatible endpoint — drop-in for any openai client |
| [**GPT4All bridge**](.) | `~/.claude/bin/llm-burst` | GPT4All models run alongside Ollama in LOCAL_HEAVY tier |

---

## ⚙️ ARCHITECTURE <a id="architecture"></a>

```
macOS LaunchAgent (ai.hmz.ollama)
         │
    Ollama server (localhost:11434)
         │
    Metal GPU backend
         │
    ┌────┴───────────────────────┐
    │    Model Registry          │
    │  llama3:latest  (default)  │
    │  mistral:latest            │
    │  codellama:latest          │
    │  phi3:latest               │
    └────────────────────────────┘
         │
    llm-burst call_ollama()
    (runs after cloud wave, RAM-safe)
```

| Model | Size | Use case | Speed on M-series |
|---|---|---|---|
| llama3:latest | 4.7GB | General tasks, analysis | ~1.5s first token |
| llama3.1:latest | 4.9GB | Better reasoning | ~1.8s first token |
| mistral:latest | 4.1GB | Fast general purpose | ~1.2s first token |
| codellama:latest | 3.8GB | Code generation | ~1.3s first token |
| phi3:mini | 2.3GB | Ultra-fast lightweight | ~0.8s first token |

---

## 💡 TIPS AND TRICKS (12) <a id="tips"></a>

[setup](#tips-setup) · [performance](#tips-perf) · [integration](#tips-int)

<a id="tips-setup"></a>
■ **Setup (4)**

| Tip | Source |
|---|---|
| `ollama pull llama3` downloads model — check progress with `ollama list` | [Ollama](https://ollama.ai) |
| LaunchAgent binary path: `/Applications/Ollama.app/Contents/Resources/ollama` — not `ollama` | [hmzainjamil](https://github.com/hmzainjamil) |
| `launchctl load ~/Library/LaunchAgents/ai.hmz.ollama.plist` to start permanently | [hmzainjamil](https://github.com/hmzainjamil) |
| Verify running: `curl http://localhost:11434/api/generate -d '{"model":"llama3","prompt":"hi"}'` | [Ollama API](https://github.com/ollama/ollama/blob/main/docs/api.md) |

<a id="tips-perf"></a>
■ **Performance (4)**

| Tip | Source |
|---|---|
| macOS Metal: set `OLLAMA_METAL=1` env var for GPU acceleration (default on Apple Silicon) | [Ollama](https://ollama.ai/blog/ollama-on-apple-silicon) |
| Keep at least 2GB RAM free — Ollama needs headroom to not swap to disk | [hmzainjamil](https://github.com/hmzainjamil) |
| `OLLAMA_NUM_PARALLEL=2` allows 2 concurrent requests — good for burst pipelines | [Ollama env vars](https://github.com/ollama/ollama/blob/main/docs/faq.md) |
| Phi3-mini at 2.3GB is fastest for quick sub-tasks — use in burst when RAM is tight | [Microsoft](https://microsoft.com/phi) |

<a id="tips-int"></a>
■ **Integration (4)**

| Tip | Source |
|---|---|
| OpenAI-compatible endpoint: `base_url="http://localhost:11434/v1"` — works with any openai SDK | [Ollama](https://ollama.ai/blog/openai-compatibility) |
| Ollama in llm-burst runs AFTER cloud wave — ensures cloud APIs get priority on RAM | [hmzainjamil](https://github.com/hmzainjamil) |
| `OLLAMA_ORIGINS=*` env var enables CORS for browser-based tools | [Ollama](https://github.com/ollama/ollama) |
| Modelfile custom: `FROM llama3\nSYSTEM "You are DigiMinds AI assistant"` for branded agent | [Ollama Modelfile](https://github.com/ollama/ollama/blob/main/docs/modelfile.md) |

---

## ☠️ STARTUPS / BUSINESSES <a id="startups"></a>

| Feature | Replaced |
|---|---|
| **Permanent local LLM server** | [Replicate](https://replicate.com), [Together AI](https://together.ai), [Fireworks AI](https://fireworks.ai) |
| **Metal GPU inference** | [LM Studio](https://lmstudio.ai), [GPT4All](https://gpt4all.io), [Jan](https://jan.ai) |
| **Zero-cost inference** | [OpenAI API](https://openai.com/api), [Anthropic API](https://anthropic.com), [Cohere](https://cohere.com) |
| **OpenAI-compatible API** | [LocalAI](https://localai.io), [Oobabooga](https://github.com/oobabooga/text-generation-webui) |

---

## Star History <a id="star"></a>

[![Star History Chart](https://api.star-history.com/svg?repos=hmzainjamil/hmz-ollama&type=Date)](https://star-history.com/#hmzainjamil/hmz-ollama&Date)
