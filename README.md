# Free Claude Code (NVIDIA API + Claude Code CLI)

Use your NVIDIA NIM API key with Claude Code by running a local Anthropic-compatible proxy.

This repo is focused on:
- Connecting your own NVIDIA API key
- Routing Claude Code traffic to NVIDIA models
- Keeping CLI workflow familiar (`claude`, `/model`, etc.)

## Quick Start

### 1) Requirements

- Python 3.13+
- `uv`
- Claude Code CLI installed

Install `uv` (Linux/macOS):

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv self update
```

### 2) Clone and Configure

```bash
git clone https://github.com/MRajibH/free-claude-code.git
cd free-claude-code
cp .env.example .env
```

Set your NVIDIA API key in `.env`:

```dotenv
NVIDIA_NIM_API_KEY="nvapi-your-key"
MODEL="nvidia_nim/qwen/qwen3-next-80b-a3b-instruct"
ENABLE_MODEL_THINKING=false
ANTHROPIC_AUTH_TOKEN=""
```

Notes:
- `ANTHROPIC_AUTH_TOKEN=""` disables local proxy auth for easy local usage.
- Use `ENABLE_MODEL_THINKING=false` for faster, more stable first-token latency.

### 3) Start Proxy

```bash
uv sync
uv run uvicorn server:app --host 0.0.0.0 --port 8082
```

### 4) Run Claude Code Through Proxy

#### Recommended (explicit model)

```bash
ANTHROPIC_BASE_URL="http://localhost:8082" CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1 claude --model "claude-3-freecc-no-thinking/nvidia_nim/qwen/qwen3-next-80b-a3b-instruct"
```

#### Or default model from `.env`

```bash
ANTHROPIC_BASE_URL="http://localhost:8082" CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1 claude
```

## How API Connection Works

1. Claude Code sends Anthropic-style requests to `http://localhost:8082`.
2. Proxy converts/routs requests to NVIDIA NIM.
3. Proxy streams response back in Anthropic SSE format Claude Code expects.

So you use Claude Code UX, but model inference is served by NVIDIA API.

## Verify Setup

Check proxy health:

```bash
curl http://localhost:8082/health
```

Check active default model:

```bash
curl http://localhost:8082/
```

List discovered models:

```bash
curl http://localhost:8082/v1/models
```

## Troubleshooting

### CLI shows Opus/Sonnet in header

That label can be CLI profile text. Force exact model with `--model` as shown above.

### Request hangs

- Use a lighter model:
  - `nvidia_nim/qwen/qwen3-next-80b-a3b-instruct`
  - or `nvidia_nim/z-ai/glm5`
- Keep thinking disabled:
  - `ENABLE_MODEL_THINKING=false`
- Retry with no-thinking model id:
  - `claude-3-freecc-no-thinking/...`

### `Missing API key` on browser

If you want no local auth prompts, keep:

```dotenv
ANTHROPIC_AUTH_TOKEN=""
```

## Security

- Never commit `.env` or API keys.
- Rotate NVIDIA keys if exposed.
- Prefer enabling proxy auth (`ANTHROPIC_AUTH_TOKEN`) on shared machines.

## License

MIT. See `LICENSE`.
