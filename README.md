# Free Claude Code (NVIDIA API + Claude Code CLI)

Use your NVIDIA NIM API key with Claude Code by running a local Anthropic-compatible proxy.

This guide focuses on:
- Connecting your NVIDIA API key
- Running Claude Code through your own proxy
- Using a stable, fast model route without Claude API billing

## Quick Start

### 1) Prerequisites

- Python 3.13+
- `uv`
- Claude Code CLI installed

Install `uv` (Linux/macOS):

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv self update
```

### 2) Clone and configure

```bash
git clone https://github.com/MRajibH/free-claude-code.git
cd free-claude-code
cp .env.example .env
```

Edit `.env`:

```dotenv
NVIDIA_NIM_API_KEY="nvapi-your-key"
MODEL="nvidia_nim/qwen/qwen3-next-80b-a3b-instruct"
ENABLE_MODEL_THINKING=false
ANTHROPIC_AUTH_TOKEN=""
```

Why these values:
- `MODEL`: fast, reliable default for coding prompts.
- `ENABLE_MODEL_THINKING=false`: avoids slow/hanging first token on some large models.
- `ANTHROPIC_AUTH_TOKEN=""`: local no-auth mode for simple local usage.

### 3) Start proxy

```bash
uv sync
uv run uvicorn server:app --host 0.0.0.0 --port 8082
```

### 4) Run Claude Code via this proxy

Recommended (explicit model):

```bash
ANTHROPIC_BASE_URL="http://localhost:8082" CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1 claude --model "claude-3-freecc-no-thinking/nvidia_nim/qwen/qwen3-next-80b-a3b-instruct"
```

Or use `.env` default model:

```bash
ANTHROPIC_BASE_URL="http://localhost:8082" CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1 claude
```

## How API connection works

1. Claude Code sends Anthropic-style requests to `http://localhost:8082`.
2. Proxy maps the request to your configured NVIDIA model.
3. NVIDIA returns output; proxy streams it back in Anthropic SSE format.

You keep Claude Code UX while inference is served by NVIDIA API.

## Verify everything

Health:

```bash
curl http://localhost:8082/health
```

Active provider/model:

```bash
curl http://localhost:8082/
```

Discovered models:

```bash
curl http://localhost:8082/v1/models
```

## Troubleshooting

### Claude Code header still says Opus/Sonnet

That can be CLI profile text. Force model with `--model`:

```bash
claude --model "claude-3-freecc-no-thinking/nvidia_nim/qwen/qwen3-next-80b-a3b-instruct"
```

### Requests hang

- Keep `ENABLE_MODEL_THINKING=false`
- Use lighter/faster models:
  - `nvidia_nim/qwen/qwen3-next-80b-a3b-instruct`
  - `nvidia_nim/z-ai/glm5`

### Browser shows `Missing API key`

Set:

```dotenv
ANTHROPIC_AUTH_TOKEN=""
```

Then restart the proxy.

## Security notes

- Never commit `.env` or raw API keys.
- Rotate NVIDIA keys if exposed.
- On shared machines, set `ANTHROPIC_AUTH_TOKEN` to a non-empty secret.

## License

MIT. See `LICENSE`.
