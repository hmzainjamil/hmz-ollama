# hmz-ollama
local AI that costs nothing and goes offline with you

![Ollama](https://img.shields.io/badge/Ollama-v0.9%2B-blue?style=flat&labelColor=000) ![GPU](https://img.shields.io/badge/GPU-M1_Pro_Metal-silver?style=flat&labelColor=555) ![Models](https://img.shields.io/badge/models-7_active-orange?style=flat&labelColor=555) ![Cost](https://img.shields.io/badge/cost-free_forever-brightgreen?style=flat&labelColor=555) ![LaunchAgent](https://img.shields.io/badge/LaunchAgent-permanent-green?style=flat&labelColor=555)

Always-on local LLM inference via Ollama — GPU-accelerated on Apple Silicon, zero API cost, zero latency on cold starts, zero data sent to any server. The backbone of the Tier 0 model routing mandate: every sub-task hits Ollama first before touching any cloud model.

## 🧠 MODELS

| Model | Size | Strength | When to Use |
|---|---|---|---|
| `llama3:latest` | 8B | General reasoning, chat, Q&A | Default for all general tasks |
| `llama3.1:8b` | 8B | Better instruction following | Tasks needing structured output |
| `llama3.2:3b` | 3B | Ultra-fast, lightweight | Simple lookups, quick transforms |
| `mistral:7b` | 7B | Balanced, reliable | Coding, structured tasks |
| `codellama:7b` | 7B | Code generation, debugging | Any code task |
| `phi3:mini` | 3.8B | Microsoft reasoning model | Math, logic, step-by-step |
| `deepseek-coder:6.7b` | 6.7B | Code-first, multilingual | Complex code generation |

**Pull new models:**
```bash
ollama pull llama3          # fastest general model
ollama pull codellama       # best for code
ollama pull phi3            # best for reasoning
ollama pull mistral         # balanced
```

## ⚙️ CONFIGURATION

**LaunchAgent** (`~/Library/LaunchAgents/ai.hmz.ollama.plist`):
```xml
<key>Label</key><string>ai.hmz.ollama</string>
<key>ProgramArguments</key><array><string>ollama</string><string>serve</string></array>
<key>KeepAlive</key><true/>
<key>RunAtLoad</key><true/>
<key>EnvironmentVariables</key><dict>
  <key>OLLAMA_NUM_GPU</key><string>99</string>
  <key>OLLAMA_FLASH_ATTENTION</key><string>1</string>
  <key>OLLAMA_MAX_LOADED_MODELS</key><string>2</string>
</dict>
```

**Key env vars:**
| Variable | Value | Effect |
|---|---|---|
| `OLLAMA_NUM_GPU` | `99` | Use all GPU layers (max Metal offload) |
| `OLLAMA_FLASH_ATTENTION` | `1` | Flash attention — 30% faster on long contexts |
| `OLLAMA_MAX_LOADED_MODELS` | `2` | Keep 2 models hot in VRAM |
| `OLLAMA_HOST` | `0.0.0.0:11434` | Expose to network (for multi-device) |
| `OLLAMA_CONTEXT_LENGTH` | `8192` | Default context per model |

## 💡 TIPS AND TRICKS

■ **Performance (6)**

| Tip | Note |
|---|---|
| Set `OLLAMA_NUM_GPU=99` — without it Ollama defaults to partial GPU offload, leaving performance on the table | KeepAlive plist only |
| Use `llama3.2:3b` for all skill-routing and classification tasks — it's 3x faster than 8B with no quality loss for binary decisions | Speed wins over quality here |
| Pre-pull models during machine setup — first pull is slow (download), subsequent runs are instant | `ollama pull` in setup script |
| `OLLAMA_MAX_LOADED_MODELS=2` keeps the two most-used models in VRAM — avoids cold-load penalty between tasks | Costs 8-16GB VRAM |
| Use `/api/generate` not `/api/chat` for one-shot completions — 15% faster, no conversation overhead | |
| Flash attention (`OLLAMA_FLASH_ATTENTION=1`) is especially impactful on 4K+ token prompts | |

■ **Integration with Claude Code (5)**

| Tip | Note |
|---|---|
| All Tier 0 sub-tasks route to `http://localhost:11434/api/generate` before hitting any cloud API | `~/.claude/bin/tier0-prompt-inject` |
| `~/.claude/bin/llm-burst` checks Ollama first — if running and RAM is available, routes there before Groq/Gemini | Low RAM (<2GB free) skips Ollama |
| `~/.claude/bin/tier0-check` pings `localhost:11434` every session start — if down, auto-falls through to Groq | |
| `~/.claude/bin/skill-auto-activate` uses llama3.2:3b for keyword matching (fastest) | |
| All `UserPromptSubmit` hooks use Ollama for non-Claude computations within the hook lifecycle | |

■ **Model Selection (4)**

| Tip | Note |
|---|---|
| `llama3` > `mistral` for English instruction-following; `mistral` > `llama3` for code with non-English comments | |
| `codellama` is purpose-trained on code — significantly better than llama3 for multi-file refactors | |
| `phi3:mini` beats llama3 8B on math and logical step-by-step tasks despite being smaller | Microsoft trained on math-heavy data |
| `deepseek-coder` is the best local model for Python and TypeScript specifically | |

■ **Gotchas (5)**

| Gotcha | Fix |
|---|---|
| Ollama crashes silently when VRAM exhausted — LaunchAgent restarts it but you lose the in-flight request | Set `OLLAMA_MAX_LOADED_MODELS=1` if models are large |
| First request after sleep takes 2-4s even with LaunchAgent — this is Metal GPU re-init, not Ollama startup | Unavoidable on macOS |
| Context length defaults to 2048 on some models — outputs truncate without warning | Explicitly set `num_ctx: 8192` in Modelfile or request |
| `ollama list` shows models but they're corrupt if disk filled during download | `ollama rm <model> && ollama pull <model>` |
| Ollama ignores `OLLAMA_*` env vars if set after launchctl load — must be in plist EnvironmentVariables | Modify plist, then `launchctl unload` + `launchctl load` |

## 🔧 OPERATIONS

```bash
# Status
ollama list                          # all pulled models
ollama ps                            # currently loaded (VRAM)
curl -s localhost:11434/api/tags     # JSON model list

# Quick inference
ollama run llama3 "explain OODA loop in 2 sentences"
echo "write a bash function to check disk usage" | ollama run codellama

# Via API
curl -s localhost:11434/api/generate -d '{
  "model": "llama3",
  "prompt": "Your task here",
  "stream": false,
  "options": {"num_ctx": 8192, "temperature": 0.1}
}' | jq '.response'

# LaunchAgent management
launchctl list | grep ai.hmz.ollama     # check status
launchctl stop ai.hmz.ollama            # stop
launchctl start ai.hmz.ollama           # start

# RAM/GPU stats during inference
sudo powermetrics --samplers gpu_power -i 1000 -n 5
```

## ☠️ WHY NOT JUST CLOUD MODELS

| Local (Ollama) | Cloud (OpenAI/Anthropic) |
|---|---|
| $0 forever | $0.002-$0.06 per 1K tokens |
| <100ms latency (warm) | 300-2000ms network latency |
| No rate limits | Hard rate limits at scale |
| Offline capable | Requires internet |
| Private — data never leaves machine | Data sent to third-party servers |
| Fixed quality | Constantly updated (can regress) |
| 7B-70B models | 1T+ parameter models |

**When to use cloud anyway:** Tasks requiring reasoning beyond 70B quality, long context (>32K tokens), vision, or when Ollama RAM is constrained. The Tier 0 routing handles this automatically.

## 📊 PERFORMANCE BENCHMARKS (M1 Pro, 16GB)

| Model | Tokens/sec (warm) | RAM Usage | VRAM Usage |
|---|---|---|---|
| llama3.2:3b | ~85 t/s | 2.1GB | 2.1GB |
| llama3:8b | ~45 t/s | 4.7GB | 4.7GB |
| mistral:7b | ~42 t/s | 4.3GB | 4.3GB |
| codellama:7b | ~40 t/s | 4.2GB | 4.2GB |
| phi3:mini | ~70 t/s | 2.3GB | 2.3GB |

## 📁 RELATED REPOS

| Repo | Role |
|---|---|
| `hmz-personal-ai-infrastructure` | Full AI stack overview |
| `hmz-g0dm0d3` | Multi-LLM routing that includes Ollama as Tier 0 |
| `claude-ai-system` | CLAUDE.md mandate requiring Ollama first |
| `claude-ai-automations` | LaunchAgent configs for always-on services |
