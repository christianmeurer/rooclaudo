# RooClaudo

> **Notice:** This project is a fork and adaptation of the original [claudo](https://github.com/digitalocean/claudo) repository by DigitalOcean. It has been modified to support the RooCode VS Code extension instead of the Claude Code CLI.

Run [RooCode](https://marketplace.visualstudio.com/items?itemName=RooCode.roo-cline) against [DigitalOcean Gradient AI](https://www.digitalocean.com/products/gradient-ai) instead of Anthropic's API.

`rooclaudo` spins up a local [LiteLLM](https://github.com/BerriAI/litellm) proxy that bridges RooCode's native Anthropic API format to DO's OpenAI-compatible endpoint — so you get the full RooCode experience billed through your DigitalOcean account.

```text
┌────────────┐         ┌────────────┐         ┌────────────┐
│  RooCode   │  ──▶    │  LiteLLM   │  ──▶    │  DO Grad.  │
│(VS Code Ext) Anthropic│   Proxy    │ OpenAI  │  AI API    │
│            │ format  │ (localhost) │ format  │            │
└────────────┘         └────────────┘         └────────────┘
```

## Prerequisites

- **Node.js** ≥ 18
- **Python 3** (for LiteLLM)
- **RooCode VS Code Extension** — installed in your editor
- **A DigitalOcean Gradient AI API key** — get one from the [DO control panel](https://cloud.digitalocean.com/gen-ai)

> **Note:** LiteLLM is installed automatically into a local virtualenv (`~/.config/rooclaudo/venv`) on first run. You do not need to install it manually.

## Installation

```bash
npm install -g rooclaudo
```

## Quick start

```bash
# First-time setup — enter your DO Gradient AI API key
rooclaudo setup

# Start the proxy for RooCode
rooclaudo
```

Once `rooclaudo` is running, configure RooCode in VS Code:
1. Open RooCode settings
2. Set API Provider to "OpenAI Compatible"
3. Set Base URL to the proxy URL (e.g., `http://localhost:4100`)
4. Set API Key to any placeholder value (the proxy handles the real authentication)
5. Select a supported model (e.g., `claude-3-5-sonnet`)

## Usage

```text
rooclaudo                    Start local proxy for RooCode
rooclaudo setup              Configure API key and discover models
rooclaudo status             Show running proxy instances
rooclaudo stop-all           Kill all proxy instances
rooclaudo models             Show discovered model mappings
rooclaudo version            Show version
rooclaudo help               Show help
```

### Examples

```bash
# Check what models are available on DO
rooclaudo models

# See running proxy instances
rooclaudo status

# Stop all running proxy instances
rooclaudo stop-all
```

## How it works

1. Loads your DO API key from `~/.config/rooclaudo/config.env`
2. Fetches available Claude models from DO's `/v1/models` endpoint (cached 24h)
3. Generates a LiteLLM config that maps Claude models to DO model IDs
4. Starts a LiteLLM proxy on a free port in the `4100–4200` range
5. RooCode connects to this local proxy, which translates and forwards requests to DO Gradient AI
6. Shuts down cleanly via `stop-all` or when terminated

Multiple `rooclaudo` sessions can run concurrently — each gets its own port.

## Configuration

| File | Purpose |
|------|---------|
| `~/.config/rooclaudo/config.env` | API key (chmod 600) |
| `~/.config/rooclaudo/models_cache.json` | Cached model list (24h TTL) |
| `~/.config/rooclaudo/litellm_config.yaml` | Auto-generated LiteLLM config |
| `~/.config/rooclaudo/venv/` | LiteLLM virtualenv |
| `~/.config/rooclaudo/logs/` | Per-instance proxy logs |

## Troubleshooting

**Proxy fails to start** — check `~/.config/rooclaudo/logs/proxy-<port>.log` for LiteLLM errors.

**Models not found** — run `rooclaudo setup` to refresh the model cache.

**Port range exhausted** — run `rooclaudo stop-all` to clean up stale instances.

**Python errors on 3.14+** — the script patches uvicorn's uvloop dependency automatically; if you hit issues, ensure your venv is up to date by deleting `~/.config/rooclaudo/venv` and re-running `rooclaudo setup`.

## Acknowledgements

This project is based on [claudo](https://github.com/digitalocean/claudo) by DigitalOcean. It has been adapted to function as a backend proxy for the RooCode VS Code extension.

## License

MIT
