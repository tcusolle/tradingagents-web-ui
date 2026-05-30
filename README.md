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
└── scripts/
    └── guard-server-restart.sh   # Claude Code hook — blocks server kill during active jobs
```

## Prerequisites

- Python 3.10+
- The `tradingagents` package installed (see below)
- At least one LLM provider API key

## Installation

### 1. Install the TradingAgents package

The web app imports `tradingagents` directly, so the core package must be installed.

**From PyPI:**
```bash
pip install tradingagents
```

**From source (if you have the repo):**
```bash
pip install -e /path/to/TradingAgents
```

### 2. Install web-specific dependencies

```bash
pip install fastapi uvicorn[standard] mistune python-dotenv
```

> `yfinance` is already a dependency of `tradingagents` — no need to install it separately.

### 3. Configure API keys

Create a `.env` file in the `2026-tradingagents-ui/` directory (i.e. next to `web/`):

```env
# ── LLM provider — set the key for whichever provider you use ──────────────
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
GOOGLE_API_KEY=...
XAI_API_KEY=...
DEEPSEEK_API_KEY=...
DASHSCOPE_API_KEY=...        # Qwen (international)
DASHSCOPE_CN_API_KEY=...     # Qwen (China region)
ZHIPU_API_KEY=...            # GLM (international)
ZHIPU_CN_API_KEY=...         # GLM (China region)
MINIMAX_API_KEY=...
MINIMAX_CN_API_KEY=...
OPENROUTER_API_KEY=...
AZURE_OPENAI_API_KEY=...
AZURE_OPENAI_DEPLOYMENT_NAME=...

# ── Data vendors (optional) ────────────────────────────────────────────────
ALPHA_VANTAGE_API_KEY=...    # Only needed if you switch data_vendors to alpha_vantage

# ── TradingAgents config overrides (optional) ──────────────────────────────
TRADINGAGENTS_LLM_PROVIDER=anthropic
TRADINGAGENTS_DEEP_THINK_LLM=claude-opus-4-7
TRADINGAGENTS_QUICK_THINK_LLM=claude-sonnet-4-6
TRADINGAGENTS_MAX_DEBATE_ROUNDS=1
TRADINGAGENTS_MAX_RISK_ROUNDS=1
TRADINGAGENTS_OUTPUT_LANGUAGE=English
```

You only need the key(s) for the provider(s) you intend to use. Reddit and StockTwits social data require no API key.

## Paths used at runtime

| Path | Purpose | Created automatically? |
|------|---------|----------------------|
| `~/.tradingagents/logs/` | Stores completed analysis reports (markdown files + `meta.json`) | Yes, on first analysis |
| `~/.tradingagents/cache/` | Data cache for market/news fetches | Yes |
| `~/.tradingagents/memory/trading_memory.md` | Agent memory log across runs | Yes |
| `web/static/` | Served as static files — must stay next to `web/app.py` | Already present |
| `.env` (in working directory) | API keys and config overrides | You create this |

All `~/.tradingagents/` paths are created automatically on first run. You can override them via env vars:

```env
TRADINGAGENTS_RESULTS_DIR=/your/path/logs
TRADINGAGENTS_CACHE_DIR=/your/path/cache
TRADINGAGENTS_MEMORY_LOG_PATH=/your/path/memory.md
```

## Running the server

From the `2026-tradingagents-ui/` directory:

```bash
uvicorn web.app:app --host 0.0.0.0 --port 8000
```

Then open [http://localhost:8000](http://localhost:8000).

For development with auto-reload:
```bash
uvicorn web.app:app --host 0.0.0.0 --port 8000 --reload
```

## Optional: guard hook (Claude Code users)

`scripts/guard-server-restart.sh` is a Claude Code pre-tool hook that blocks any shell command that would kill or restart the uvicorn process while an analysis job is active. To enable it, add it to your `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash /path/to/scripts/guard-server-restart.sh"
          }
        ]
      }
    ]
  }
}
```
