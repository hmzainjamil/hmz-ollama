<p align="center">
  <img src="https://img.shields.io/badge/HMZ-OLLAMA-black?style=for-the-badge&logoColor=white" alt="HMZ Ollama" height="60">
</p>

<h1 align="center">HMZ Ollama</h1>

<p align="center">
  <strong>Local AI infrastructure — Llama 3, Mistral, DeepSeek, CodeLlama running offline on Apple Silicon with GPU acceleration</strong>
</p>

<p align="center">
  <a href="https://github.com/hmzainjamil"><img src="https://img.shields.io/badge/By-HMZ-6C3EE8?style=for-the-badge" alt="By HMZ"></a>
  <a href="#models"><img src="https://img.shields.io/badge/Models-Llama3%20%2B%20More-20A34E?style=for-the-badge" alt="Llama3+"></a>
  <a href="#"><img src="https://img.shields.io/badge/GPU-Apple%20Silicon-F86606?style=for-the-badge" alt="Apple Silicon GPU"></a>
  <a href="#"><img src="https://img.shields.io/badge/Cost-Zero-246DFF?style=for-the-badge" alt="Zero Cost"></a>
  <a href="#"><img src="https://img.shields.io/badge/Offline-100%25-9D97F4?style=for-the-badge" alt="100% Offline"></a>
</p>

<p align="center">
  <a href="#overview">Overview</a> &bull;
  <a href="#quick-start">Quick Start</a> &bull;
  <a href="#models">Models</a> &bull;
  <a href="#integration">Integration</a> &bull;
  <a href="#use-cases">Use Cases</a> &bull;
  <a href="#installation">Installation</a>
</p>

---

## Overview

**HMZ Ollama** documents the local AI model infrastructure powering the offline tier of the HMZ AI system. Ollama runs on macOS with Apple Silicon GPU acceleration — models respond in milliseconds with zero API costs, zero rate limits, and zero internet dependency.

In the HMZ Tier 0 routing hierarchy, Ollama serves as the always-available fallback and the privacy-first option for sensitive tasks:
- **Zero cost** — runs entirely on local hardware
- **Zero rate limits** — no quotas, no throttling
- **Zero privacy risk** — data never leaves the machine
- **GPU accelerated** — Apple Silicon Metal acceleration, ~5x faster than CPU

---

## Quick Start

```bash
# Start Ollama (runs as LaunchAgent — starts at login automatically)
ollama serve

# Run llama3 interactively
ollama run llama3

# Run via API (OpenAI-compatible)
curl http://localhost:11434/api/generate -d '{
  "model": "llama3",
  "prompt": "Write a cold email for a digital marketing agency"
}'

# Use in llm-burst
~/.claude/bin/llm-burst --models ollama "your prompt"

# Pull additional models
ollama pull mistral
ollama pull codellama
ollama pull deepseek-coder
ollama pull phi3
```

---

## Models

### Installed (ready to use)

| Model | Size | Strength | Use for |
|---|---|---|---|
| `llama3:latest` | 4.7 GB | General purpose, strong reasoning | Everything — default model |

### Available to pull (tested)

| Model | Command | Strength |
|---|---|---|
| Mistral 7B | `ollama pull mistral` | Fast, reliable, multilingual |
| CodeLlama | `ollama pull codellama` | Code generation, debugging |
| DeepSeek-Coder | `ollama pull deepseek-coder` | Best local coding model |
| Phi-3 mini | `ollama pull phi3` | Lightweight, fast for simple tasks |
| Gemma 2 | `ollama pull gemma2` | Google architecture, efficient |
| Qwen2.5 | `ollama pull qwen2.5` | Strong multilingual + code |
| Neural Chat | `ollama pull neural-chat` | Conversational tasks |
| LLaVA (vision) | `ollama pull llava` | Image understanding, OCR |

### RAM requirements

| Model size | RAM needed | Fits on |
|---|---|---|
| 3B–7B | 4–8 GB | MacBook Air M1+ |
| 8B–13B | 8–16 GB | MacBook Pro M1+ |
| 30B–70B | 32–64 GB | Mac Studio / Mac Pro |

*Current system: 2.8 GB RAM available → 3B-7B models recommended*

---

## Integration

### With Claude Code (via llm-burst)

Ollama integrates into the HMZ multi-LLM burst system:

```bash
# Use Ollama specifically (when OpenRouter is down or for privacy)
~/.claude/bin/llm-burst --models ollama "your prompt"

# Include Ollama in the full burst
~/.claude/bin/llm-burst --models ollama,groq,gemini "your prompt"
```

### With n8n

Ollama works as a local AI node in n8n workflows:
- Use the **Ollama** n8n community node
- Base URL: `http://localhost:11434`
- No API key required

### With Open WebUI (optional)

```bash
# Install a ChatGPT-like UI for Ollama
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway   -v open-webui:/app/backend/data   --name open-webui ghcr.io/open-webui/open-webui:main
# Access at http://localhost:3000
```

### Direct Python integration

```python
import requests

def ollama(prompt, model="llama3"):
    r = requests.post("http://localhost:11434/api/generate",
        json={"model": model, "prompt": prompt, "stream": False})
    return r.json()["response"]

result = ollama("Write a cold email for digital agency services")
```

---

## Use Cases

| Task | Why use Ollama | Model |
|---|---|---|
| **Offline work** | No internet? Ollama still runs. | `llama3` |
| **Sensitive client data** | Data stays 100% local | `llama3` or `mistral` |
| **Code generation** | Fast, free, no quota | `deepseek-coder` or `codellama` |
| **Quick research** | Sub-second response for simple queries | `phi3` (smallest) |
| **Multilingual content** | Chinese, Arabic, Spanish support | `qwen2.5` |
| **Cost-free batches** | Process 1000s of items with zero API cost | `llama3` |
| **Image understanding** | OCR, image description, visual QA | `llava` |

---

## LaunchAgent Setup

Ollama runs permanently as a macOS LaunchAgent:

```bash
# Check if running
launchctl list | grep ollama

# If not installed as LaunchAgent:
cat > ~/Library/LaunchAgents/ai.hmz.ollama.plist << EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key><string>ai.hmz.ollama</string>
    <key>ProgramArguments</key>
    <array><string>/usr/local/bin/ollama</string><string>serve</string></array>
    <key>RunAtLoad</key><true/>
    <key>KeepAlive</key><true/>
</dict>
</plist>
EOF
launchctl load ~/Library/LaunchAgents/ai.hmz.ollama.plist
```

---

## Installation

```bash
# Install Ollama
brew install ollama

# Pull the default model
ollama pull llama3

# Verify API is running
curl http://localhost:11434/api/tags
```

---

## Resources

- **[Ollama.ai](https://ollama.ai)** — official documentation
- **[Ollama model library](https://ollama.ai/library)** — all available models
- **[Open WebUI](https://github.com/open-webui/open-webui)** — ChatGPT-like local UI
- **[hmz-g0dm0d3](https://github.com/hmzainjamil/hmz-g0dm0d3)** — model racing system (Ollama is Tier 0)
- **[claude-ai-automations](https://github.com/hmzainjamil/claude-ai-automations)** — llm-burst integrates Ollama

## License

MIT

---

<p align="center">Built by <a href="https://github.com/hmzainjamil">Hafiz Muhammad Zulqarnain</a> &mdash; HMZ AI Agency</p>
<p align="center"><sub>100% local. Zero cost. Zero rate limits. Zero privacy risk.</sub></p>