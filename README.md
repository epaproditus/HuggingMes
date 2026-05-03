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
    description: "Your LLM provider API key (e.g. Anthropic, OpenAI, Google, OpenRouter)."
  - name: LLM_MODEL
    description: "Model ID to use, e.g. google/gemini-2.0-flash or openai/gpt-4o."
  - name: GATEWAY_TOKEN
    description: "Strong token to secure your dashboard and API (generate: openssl rand -hex 32)."
  - name: TELEGRAM_BOT_TOKEN
    description: "Telegram bot token from @BotFather."
  - name: TELEGRAM_ALLOWED_USERS
    description: "Comma-separated list of numeric user IDs allowed to use the bot."
  - name: HF_TOKEN
    description: "Hugging Face token with write access. Used for automatic workspace backup."
  - name: CLOUDFLARE_WORKERS_TOKEN
    description: "Cloudflare API token for automatic Worker proxy and KeepAlive setup."
---

<!-- Badges -->
[![GitHub Stars](https://img.shields.io/github/stars/NousResearch/hermes-agent?style=flat-square)](https://github.com/NousResearch/hermes-agent)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://opensource.org/licenses/MIT)
[![HF Space](https://img.shields.io/badge/🤗%20HuggingFace-Space-blue?style=flat-square)](https://huggingface.co/spaces)
[![Hermes](https://img.shields.io/badge/Hermes-Agent-indigo?style=flat-square)](https://github.com/NousResearch/hermes-agent)

**Self-hosted Hermes AI agent gateway — free, no server needed.** HuggingMess runs [Nous Research Hermes Agent](https://github.com/NousResearch/hermes-agent) on HuggingFace Spaces, providing a 24/7 personal AI assistant. It includes a premium management dashboard, automatic persistent backup to HF Datasets, and built-in connectivity fixes to bypass platform restrictions. Deploy in minutes on the free HF Spaces tier with full data persistence.

## Table of Contents

- [✨ Features](#-features)
- [🚀 Quick Start](#-quick-start)
- [🔐 Access Control](#-access-control)
- [🤖 LLM Providers](#-llm-providers)
- [📱 Telegram Setup](#-telegram-setup)
- [🌐 Cloudflare Proxy](#-cloudflare-proxy)
- [💾 Backup & Persistence](#-backup--persistence)
- [💓 Staying Alive](#-staying-alive)
- [🔐 Security & Advanced](#-security--advanced)
- [💻 Local Development](#-local-development)
- [🏗️ Architecture](#️-architecture)
- [🐛 Troubleshooting](#-troubleshooting)
- [🌟 More Projects](#-more-projects)

## ✨ Features

- 🧠 **Hermes Core:** Runs the powerful Hermes agent framework for multi-modal chat and tool use.
- 🔐 **Secure by Default:** Adds a custom auth layer to protect the Hermes dashboard and API routes.
- 🌐 **Built-in Connectivity:** Includes transparent outbound proxying via Cloudflare Workers for Telegram, Google APIs, and more.
- 📊 **Premium Dashboard:** Beautiful Web UI at `/` for real-time monitoring of uptime, sync health, and agent status.
- 💾 **Persistent Backup:** Automatically syncs agent state, chats, and config to a private HF Dataset.
- ⏰ **Easy Keep-Alive:** Uses `CLOUDFLARE_WORKERS_TOKEN` to automatically set up a cron-triggered keep-awake worker at boot.
- 🐳 **Optimized Infrastructure:** Minimal resource usage with clean startup logs and production-ready proxying.

## 🚀 Quick Start

### Step 1: Duplicate this Space

[![Duplicate this Space](https://huggingface.co/datasets/huggingface/badges/resolve/main/duplicate-this-space-xl.svg)](https://huggingface.co/spaces/somratpro/HuggingMess?duplicate=true)

### Step 2: Add Your Secrets

Navigate to your new Space's **Settings → Variables and secrets**, and add the following three under **Secrets**:

- `LLM_API_KEY` – Your provider API key (e.g., Anthropic, OpenAI, OpenRouter).
- `LLM_MODEL` – The model ID string (e.g., `google/gemini-2.0-flash` or `openai/gpt-4o`).
- `GATEWAY_TOKEN` – A custom password to secure your dashboard.

### Step 3: Access Your Dashboard

Once the build is complete, visit your Space's public URL. You will see the HuggingMess management dashboard. Click **Open Hermes UI** and enter your `GATEWAY_TOKEN` to access the agent interface.

## 🔐 Access Control

Hermes' built-in dashboard is local-first. HuggingMess adds a secure wrapper:

- **Dashboard:** Opening `/app/` requires your `GATEWAY_TOKEN`.
- **API:** Routes under `/v1/*` (OpenAI-compatible) require `Authorization: Bearer <GATEWAY_TOKEN>`.

## 🤖 LLM Providers

HuggingMess automatically maps your `LLM_MODEL` and `LLM_API_KEY` to the correct Hermes configuration.

| Provider | Prefix | Example `LLM_MODEL` |
| :--- | :--- | :--- |
| **Google** | `google/` | `google/gemini-2.0-flash` |
| **OpenRouter** | `openrouter/` | `openrouter/anthropic/claude-3.5-sonnet` |
| **Anthropic** | `anthropic/` | `anthropic/claude-3-opus-latest` |
| **OpenAI** | `openai/` | `openai/gpt-4o` |
| **HuggingFace** | `huggingface/` | `huggingface/meta-llama/Llama-3.3-70B-Instruct` |

## 📱 Telegram Setup *(Optional)*

To use Hermes via Telegram:

1. Add `TELEGRAM_BOT_TOKEN` from [@BotFather](https://t.me/BotFather).
2. Add `TELEGRAM_ALLOWED_USERS` (comma-separated numeric IDs) to restrict access.
3. Add `CLOUDFLARE_WORKERS_TOKEN` to bypass HF network restrictions automatically.

## 🌐 Cloudflare Proxy

HuggingFace Spaces often block outbound connections to external APIs. HuggingMess handles this automatically:

1. Add `CLOUDFLARE_WORKERS_TOKEN` as a Space secret.
2. Restart the Space.

HuggingMess will auto-provision a Worker proxy for Telegram and other restricted traffic, and set up a keep-awake cron.

## 💾 Backup & Persistence

Set `HF_TOKEN` with **Write** access to enable backup. HuggingMess syncs all agent data to a private Dataset named `huggingmess-backup` every 180 seconds.

## 💓 Staying Alive *(Recommended on Free HF Spaces)*

Your Space will automatically be kept awake by a background Cloudflare Worker when you configure the `CLOUDFLARE_WORKERS_TOKEN` secret. The worker uses a cron trigger to regularly ping your Space's `/health` endpoint. The dashboard displays the current keep-alive worker status.

## 🔐 Security & Advanced

| Variable | Default | Description |
| :--- | :--- | :--- |
| `GATEWAY_TOKEN` | — | Token for dashboard and API auth |
| `HF_TOKEN` | — | HF token with write access for backups |
| `CLOUDFLARE_WORKERS_TOKEN` | — | Cloudflare API token for proxy & keep-awake |
| `SYNC_INTERVAL` | `180` | Backup frequency in seconds |
| `CLOUDFLARE_KEEPALIVE_ENABLED` | `true` | Set `false` to disable keep-awake worker |
| `TELEGRAM_MODE` | `webhook` | `webhook` or `polling` |

## 💻 Local Development

```bash
docker compose up --build
# Dashboard: http://localhost:7861
# Hermes App: http://localhost:7861/app/
```

## 🏗️ Architecture

- **Dashboard (`/`)**: Real-time management and monitoring.
- **Hermes App (`/app/`)**: Secure proxied access to the Hermes UI.
- **API (`/v1/*`)**: Proxied OpenAI-compatible agent API.
- **Health Check (`/health`)**: Readiness probe for HF and Keep-Alive.
- **Sync Engine**: Python background task for HF Dataset persistence.

## 🐛 Troubleshooting

- **Telegram bot not responding:** Ensure `CLOUDFLARE_WORKERS_TOKEN` is set. Check logs for "Setting up Cloudflare proxy".
- **Authentication failed:** Clear your browser cookies or use an incognito window if your `GATEWAY_TOKEN` has changed.
- **Data not persisting:** Ensure `HF_TOKEN` has **Write** permissions.
- **Space keeps sleeping:** Add `CLOUDFLARE_WORKERS_TOKEN` as a Space secret to enable automatic keep-awake monitoring via Cloudflare Workers.

## 🌟 More Projects

Similar projects by [@somratpro](https://github.com/somratpro) — all free, one-click deploy on HF Spaces:

| Project | What it runs | HF Space | GitHub |
| :--- | :--- | :--- | :--- |
| **Hugging8n** | n8n — workflow & automation platform | [Space](https://huggingface.co/spaces/somratpro/Hugging8n) | [Repo](https://github.com/somratpro/hugging8n) |
| **HuggingClaw** | OpenClaw — Claude Code in the browser | [Space](https://huggingface.co/spaces/somratpro/HuggingClaw) | [Repo](https://github.com/somratpro/huggingclaw) |
| **HuggingClip** | Paperclip — AI agent orchestration platform | [Space](https://huggingface.co/spaces/somratpro/HuggingClip) | [Repo](https://github.com/somratpro/huggingclip) |
| **HuggingPost** | Postiz — social-media scheduler | [Space](https://huggingface.co/spaces/somratpro/HuggingPost) | [Repo](https://github.com/somratpro/huggingpost) |

---
*Made with ❤️ by [@somratpro](https://github.com/somratpro)*
