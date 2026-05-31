# TradingAgents Web UI

A FastAPI web interface for [TradingAgents](https://github.com/TauricResearch/TradingAgents) that streams live agent updates via Server-Sent Events and browses past analyses.

## What's included

```
2026-tradingagents-ui/
├── web/
│   ├── app.py              # FastAPI backend
│   └── static/
│       ├── index.html      # Single-page frontend
│       └── marked.min.js   # Bundled markdown renderer
├── scripts/
│   └── guard-server-restart.sh   # Claude Code hook — blocks server kill during active jobs
├── memory/
│   └── trading_memory.md   # Agent memory log (persists across runs)
├── main.py                 # Entry point with .env loading
└── WEBUI_SPEC.md           # Full spec — hand this to an AI to rebuild the UI from scratch
```

## Prerequisites

- Python 3.10+
- `uv` — `curl -LsSf https://astral.sh/uv/install.sh | sh`
- At least one LLM provider API key

---

## Installation

### 1. Clone TradingAgents (upstream core)

```bash
git clone https://github.com/TauricResearch/TradingAgents.git
cd TradingAgents
```

### 2. Install the package and all dependencies

```bash
uv venv
uv pip install -e .
uv pip install fastapi uvicorn[standard] mistune python-dotenv
```

### 3. Copy our web UI layer into the repo

From this folder (`2026-tradingagents-ui/`):

```bash
cp -r web/ scripts/ memory/ main.py /path/to/TradingAgents/
```

### 4. Configure `.env`

```bash
cp .env.example .env  # from the TradingAgents root
```

Add your API key and point memory to the repo-local folder:

```env
# LLM provider — set whichever you use
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
GOOGLE_API_KEY=...
XAI_API_KEY=...

# Point memory log to the repo-local file we copied in step 3
TRADINGAGENTS_MEMORY_LOG_PATH=./memory/trading_memory.md
```

### 5. Run

```bash
.venv/bin/uvicorn web.app:app --host 0.0.0.0 --port 8000
```

Open [http://localhost:8000](http://localhost:8000).

---

## API keys reference

| Provider | Env var |
|----------|---------|
| Anthropic | `ANTHROPIC_API_KEY` |
| OpenAI | `OPENAI_API_KEY` |
| Google | `GOOGLE_API_KEY` |
| xAI | `XAI_API_KEY` |
| DeepSeek | `DEEPSEEK_API_KEY` |
| Qwen | `DASHSCOPE_API_KEY` |
| GLM | `ZHIPU_API_KEY` |
| OpenRouter | `OPENROUTER_API_KEY` |
| Azure | `AZURE_OPENAI_API_KEY` + `AZURE_OPENAI_DEPLOYMENT_NAME` |
| Alpha Vantage (data) | `ALPHA_VANTAGE_API_KEY` |

Reddit and StockTwits require no API key.

---

## Runtime paths

| Path | Purpose |
|------|---------|
| `memory/trading_memory.md` | Agent memory — set via `TRADINGAGENTS_MEMORY_LOG_PATH` |
| `~/.tradingagents/logs/` | Analysis reports — auto-created on first run |
| `~/.tradingagents/cache/` | Data cache — auto-created |

---

## Optional: guard hook (Claude Code users)

`scripts/guard-server-restart.sh` blocks any shell command that would kill uvicorn while an analysis job is active. Add to `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "bash scripts/guard-server-restart.sh" }]
      }
    ]
  }
}
```
