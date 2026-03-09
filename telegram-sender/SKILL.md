---
name: telegram-sender
description: Send Telegram text messages and documents from the `telegram_sender` workspace. Use when Codex needs to notify someone through Telegram, send a local file with the bundled shell script, operate or test the Dockerized FastAPI sender service, or inspect/update this project's bot token, chat ID, API key, endpoints, and payload handling.
---

# Telegram Sender

Use this skill to work with the Telegram delivery tools in this repository. It covers both the local shell sender at `../../send_tg.sh` and the HTTP API defined by `../../api_server.py` and `../../docker-compose.yml`.

Read `references/usage.md` when you need exact commands, request examples, or troubleshooting steps.

## Choose the path

1. Use `../../send_tg.sh` for one-off local sends from the current machine.
2. Use the API service for integrations, health checks, or when another tool needs HTTP endpoints.
3. Inspect `../../api_server.py`, `../../send_tg.sh`, `../../docker-compose.yml`, and `../../.env.example` when changing behavior or debugging auth/config issues.

## Operating rules

- Work from the repository root containing `send_tg.sh` and `docker-compose.yml`.
- Do not echo bot tokens or API keys back to the user. Report whether they are configured, not their raw values.
- Prefer environment variables or `.env` over editing hardcoded defaults in the shell script.
- When the user wants content sent immediately, use the smallest viable path: shell script for direct sends, API only when HTTP is required.
- If the API is expected to be running, verify with `/health` before debugging deeper issues.

## Common workflows

### Send a text message

- Prefer `../../send_tg.sh -m "..."` for a direct local send.
- If the message comes from another command, pipe it into `../../send_tg.sh --stdin`.
- Use the API `/send/message` when the caller needs an HTTP interface or per-request `chat_id`.

### Send a document

- Use `../../send_tg.sh -d <path>` for a local file send.
- Use `/send/document` for multipart HTTP uploads.
- Fail early if the file path does not exist or the upload is empty.

### Run or verify the API service

- Configure `.env` from `../../.env.example`.
- Start the service with `docker compose up -d --build` from the repo root.
- Verify readiness with `curl http://localhost:${API_PORT:-8080}/health`.
- Open `/docs` only when interactive API docs are actually needed.

### Debug failures

- Check whether `TG_BOT_TOKEN`, `TG_CHAT_ID`, and `API_KEY` are set in the environment or `.env`.
- For HTTP failures, inspect the FastAPI route behavior in `../../api_server.py`.
- For shell-script failures, inspect CLI flags and env override logic in `../../send_tg.sh`.
- If Docker is involved, inspect container state and logs before changing code.
