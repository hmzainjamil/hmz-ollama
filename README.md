# hmz-ollama
Local LLM inference via Ollama — always-on GPU-accelerated models for zero-cost AI operations

![Ollama](https://img.shields.io/badge/Ollama-v0.9%2B-blue?style=flat&labelColor=000) ![GPU](https://img.shields.io/badge/GPU-M1_Pro_Metal-silver?style=flat&labelColor=555) ![Models](https://img.shields.io/badge/models-7-orange?style=flat&labelColor=555) ![Cost](https://img.shields.io/badge/cost-free-brightgreen?style=flat&labelColor=555)

Part of the [HMZ AI Infrastructure](https://github.com/hmzainjamil) stack — local inference layer, zero API cost.

---

## 🧠 MODEL INVENTORY

| Model | Parameters | Context | Best For |
|-------|-----------|---------|----------|
| `llama3:latest` | 8B | 8K | General tasks, primary model |
| `codellama` | 7B | 16K | Code generation |
| `mistral` | 7B | 8K | Fast general purpose |
| `phi3` | 3.8B | 4K | Lightweight, low-RAM tasks |

## ⚙️ CONFIGURATION

```bash
# Check running models
ollama list

# Pull new model
ollama pull llama3:70b

# Run with specific context
ollama run llama3 --context-length 16384

# API endpoint (LaunchAgent keeps it always-on)
curl http://localhost:11434/api/generate -d '{"model": "llama3", "prompt": "...", "stream": false}'
```

LaunchAgent: always-on, auto-restarts on crash, available at `http://localhost:11434`

## 💡 INTEGRATION WITH HMZ STACK

| Component | How it uses Ollama |
|-----------|-------------------|
| G0DM0D3 burst | Tier 0 local inference |
| Paperclip CEO loop | Sub-task processing |
| Skill auto-activator | Prompt classification |
| `llm-burst` CLI | `--models ollama` flag |

■ **RAM Requirements**

| Model | Min RAM | Recommended |
|-------|---------|-------------|
| 3B-4B models | 4GB | 8GB |
| 7B-8B models | 8GB | 16GB |
| 13B models | 16GB | 32GB |
| 70B models | 40GB | 64GB |

---
Built by [HMZ](https://github.com/hmzainjamil) · [DigiMinds](https://digiminds.org)
