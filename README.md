---
title: HuggingMess
emoji: 📚
colorFrom: blue
colorTo: indigo
sdk: docker
app_port: 7861
pinned: true
license: mit
secrets:
  - name: LLM_API_KEY
    description: "Your LLM provider API key. HuggingMess maps it to the right Hermes provider env var."
  - name: LLM_MODEL
    description: "Optional model ID override, e.g. openrouter/anthropic/claude-sonnet-4 or anthropic/claude-opus-4.6."
  - name: TELEGRAM_BOT_TOKEN
    description: "Telegram bot token from @BotFather."
  - name: TELEGRAM_ALLOWED_USERS
    description: "Comma-separated numeric Telegram user IDs allowed to use the bot."
  - name: GATEWAY_TOKEN
    description: "Bearer token for the proxied Hermes API routes."
  - name: HF_TOKEN
    description: "Hugging Face token with write access for private Dataset backup."
  - name: CLOUDFLARE_WORKERS_TOKEN
    description: "Cloudflare API token for automatic Worker proxy setup."
  - name: UPTIMEROBOT_API_KEY
    description: "UptimeRobot Main API key for automatic keep-awake monitor setup."
---

# HuggingMess

HuggingMess runs [Nous Research Hermes Agent](https://github.com/NousResearch/hermes-agent) as a Hugging Face Docker Space. It follows the same practical shape as HuggingClaw: one public Space port, Telegram gateway support, Cloudflare Worker proxy setup, UptimeRobot keep-awake, and private HF Dataset backup for Hermes state.

## Quick Start

1. Duplicate this Space or push this folder to a new Docker Space.
2. Add these secrets in Space Settings:

| Secret | Required | Notes |
| :--- | :--- | :--- |
| `LLM_MODEL` | Optional | Model override. If unset, HuggingMess leaves Hermes default/restored config alone. |
| `LLM_API_KEY` | Usually | Used to populate the provider-specific env var automatically |
| `TELEGRAM_BOT_TOKEN` | For Telegram | Bot token from BotFather |
| `TELEGRAM_ALLOWED_USERS` | Recommended | Comma-separated numeric Telegram user IDs |
| `GATEWAY_TOKEN` | Recommended | Bearer token for `/v1/*` API routes |
| `HF_TOKEN` | Optional | Enables private Dataset backup named `huggingmess-backup` |
| `CLOUDFLARE_WORKERS_TOKEN` | Optional | Auto-creates a Worker proxy for Telegram Bot API traffic |
| `UPTIMEROBOT_API_KEY` | Optional | Auto-creates a monitor for `/health` |

## Access Control

Hermes' built-in dashboard is local-first and does not provide its own public auth layer. HuggingMess adds wrapper-level auth for the exposed Space routes.

Set this Space secret:

```text
GATEWAY_TOKEN=your-strong-password-or-token
```

Then:

- Opening `/app/` shows a HuggingMess login page with one field.
- Paste `GATEWAY_TOKEN` into that field.
- HuggingMess stores an HTTP-only session cookie for the dashboard routes.
- API routes under `/v1/*` accept `Authorization: Bearer <GATEWAY_TOKEN>`.

## LLM Providers

HuggingMess supports two configuration styles:

1. **Simple wrapper style:** set `LLM_MODEL` and `LLM_API_KEY`; HuggingMess maps them into Hermes config and provider-specific API key variables.
2. **Native Hermes style:** set Hermes variables directly, such as `HERMES_MODEL`, `HERMES_INFERENCE_PROVIDER`, and the provider API key.

### Gemini

For Google Gemini, add these Space secrets:

| Secret | Value |
| :--- | :--- |
| `LLM_MODEL` | `google/gemini-2.5-flash` |
| `LLM_API_KEY` | Your Google AI Studio API key |

HuggingMess will convert that into Hermes config:

```yaml
model:
  default: gemini-2.5-flash
  provider: gemini
```

And it exports both `GOOGLE_API_KEY` and `GEMINI_API_KEY` for Hermes. You can also use `gemini/gemini-2.5-flash`; the prefix is stripped the same way.

Native Hermes equivalent:

| Secret | Value |
| :--- | :--- |
| `HERMES_MODEL` | `gemini-2.5-flash` |
| `HERMES_INFERENCE_PROVIDER` | `gemini` |
| `GOOGLE_API_KEY` or `GEMINI_API_KEY` | Your Google AI Studio API key |

### Common Examples

| Provider | Simple `LLM_MODEL` | API key secret |
| :--- | :--- | :--- |
| Gemini | `google/gemini-2.5-flash` | `LLM_API_KEY` |
| OpenRouter | `openrouter/anthropic/claude-sonnet-4` | `LLM_API_KEY` |
| Anthropic | `anthropic/claude-opus-4.6` | `LLM_API_KEY` |
| OpenAI | `openai/gpt-4o` | `LLM_API_KEY` |
| Hugging Face Router | `huggingface/meta-llama/Llama-3.3-70B-Instruct` | `LLM_API_KEY` |

## Telegram on HF Spaces

When `TELEGRAM_BOT_TOKEN` and `SPACE_HOST` are present, HuggingMess defaults Telegram to webhook mode:

```bash
TELEGRAM_WEBHOOK_URL=https://your-space.hf.space/telegram
TELEGRAM_WEBHOOK_PORT=8765
```

If you need polling instead, set:

```bash
TELEGRAM_MODE=polling
```

Hermes requires numeric Telegram IDs for allowlists. You can use either Hermes-native `TELEGRAM_ALLOWED_USERS` or the HuggingClaw-style aliases `TELEGRAM_USER_ID` / `TELEGRAM_USER_IDS`.

## Cloudflare Proxy

Hugging Face Spaces can be restrictive for outbound bot API traffic. Add `CLOUDFLARE_WORKERS_TOKEN`, and HuggingMess will:

1. create a Cloudflare Worker,
2. generate a shared proxy secret,
3. set Hermes Telegram `base_url` to `https://worker.example.workers.dev/bot`,
4. set `base_file_url` to `https://worker.example.workers.dev/file/bot`.

Manual mode is also supported:

```bash
CLOUDFLARE_PROXY_URL=https://your-worker.workers.dev
CLOUDFLARE_PROXY_SECRET=optional-shared-secret
```

The manual Worker source is included in `cloudflare-worker.js`.

## Backup

Set `HF_TOKEN` with write access to enable backup. HuggingMess syncs `/opt/data` to a private Dataset named `huggingmess-backup` every 180 seconds by default.

| Variable | Default | Description |
| :--- | :--- | :--- |
| `BACKUP_DATASET_NAME` | `huggingmess-backup` | Dataset name under your HF account |
| `SYNC_INTERVAL` | `180` | Backup interval in seconds |
| `SYNC_INCLUDE_ENV` | `false` | Include `/opt/data/.env` in backup |

By default `.env` is excluded from backups because HF Space secrets are already injected at runtime.

## Keep Awake

Add `UPTIMEROBOT_API_KEY`, and HuggingMess creates or reuses a monitor for:

```text
https://your-space.hf.space/health
```

Optional UptimeRobot variables:

| Variable | Default | Description |
| :--- | :--- | :--- |
| `UPTIMEROBOT_MONITOR_NAME` | `HuggingMess <space>` | Friendly monitor name |
| `UPTIMEROBOT_INTERVAL` | `300` | Monitor interval in seconds |
| `UPTIMEROBOT_ALERT_CONTACTS` | unset | Dash-separated alert contact IDs |

## Local Development

```bash
docker compose up --build
```

Then open:

```text
http://localhost:7861
```

## Useful Routes

| Route | Purpose |
| :--- | :--- |
| `/` | HuggingMess dashboard |
| `/health` | Health check for HF and UptimeRobot |
| `/status` | JSON status |
| `/app/` | Proxied Hermes dashboard/app |
| `/v1/models` | Proxied Hermes OpenAI-compatible API server |
| `/telegram` | Telegram webhook endpoint |

The `/v1/*` routes require:

```text
Authorization: Bearer <GATEWAY_TOKEN>
```

## Links

- [Hermes Agent GitHub](https://github.com/NousResearch/hermes-agent)
- [Hermes Agent Docs](https://hermes-agent.nousresearch.com/docs)
- [Hermes Docker Docs](https://hermes-agent.nousresearch.com/docs/user-guide/docker/)
- [Hermes Telegram Docs](https://hermes-agent.nousresearch.com/docs/user-guide/messaging/telegram)
