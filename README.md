# TradingAgents Web UI

> **This is only a UI layer.** All of the multi-agent trading analysis is done by
> [TradingAgents](https://github.com/TauricResearch/TradingAgents) from Tauric
> Research, which is a separate project with its own authors and its own
> license (Apache 2.0). This repo contains none of that code. It adds a small
> FastAPI backend and single-page frontend that run TradingAgents jobs and show
> the results.

## What this adds
- **Live progress:** a FastAPI backend (`web/app.py`) runs an analysis job in a
  background thread and streams agent updates to the browser over Server-Sent
  Events. A replay buffer lets a reconnecting client resume without losing events.
- **Report browser:** lists past analyses and renders them (parsing both the
  Markdown and JSON report formats TradingAgents produces).
- **Price charts and ticker info:** endpoints for chart data and basic ticker
  details, plus a rough per-run cost estimate in the frontend.
- **A Claude Code hook:** `scripts/guard-server-restart.sh` blocks shell commands
  that would kill the server while an analysis is running.

## What's included
```
web/app.py                   # FastAPI backend (job runner, SSE, report parsing)
web/static/index.html        # single-page frontend
web/static/marked.min.js     # bundled Markdown renderer
scripts/guard-server-restart.sh
main.py                      # small entry point that loads .env and runs the upstream graph
WEBUI_SPEC.md                # spec for the UI, detailed enough to rebuild it from scratch
```

## Not financial advice
TradingAgents is a research framework. Nothing here is investment advice, and
this UI does not place trades.

## Security
There is no authentication. Run it on localhost only (the command below does),
because the server uses your LLM API keys to run jobs.

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

From this repo:

```bash
mkdir -p memory
cp -r web/ scripts/ main.py /path/to/TradingAgents/
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

# Keep the memory log inside the project (create ./memory first)
TRADINGAGENTS_MEMORY_LOG_PATH=./memory/trading_memory.md
```

### 5. Run

```bash
.venv/bin/uvicorn web.app:app --host 127.0.0.1 --port 8000
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
| `memory/trading_memory.md` | Agent memory, set via `TRADINGAGENTS_MEMORY_LOG_PATH` (created at runtime) |
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

## License

MIT, covering only the UI code in this repository. The TradingAgents framework
it wraps is separate work by Tauric Research under its own license (Apache 2.0).
