# Telegram Sender Reference

## Repository files

- `../../send_tg.sh`: local shell entry point for sending text or documents
- `../../api_server.py`: FastAPI server that proxies requests to the Telegram Bot API
- `../../docker-compose.yml`: local service wrapper
- `../../.env.example`: required environment variable names

## Local shell commands

Send text:

```bash
./send_tg.sh -m "hello telegram"
echo "multi-line message" | ./send_tg.sh --stdin
```

Send a document:

```bash
./send_tg.sh -d ./report.txt --caption "daily report"
```

Override config without editing the script:

```bash
TG_BOT_TOKEN=... TG_CHAT_ID=... ./send_tg.sh -m "hello"
./send_tg.sh -t "123456:ABCDEF" -c "123456789" -m "hello"
```

## API service

Create local config:

```bash
cp .env.example .env
```

Required variables:

```dotenv
TG_BOT_TOKEN=your_bot_token
TG_CHAT_ID=your_default_chat_id
API_KEY=your_api_key
API_PORT=8080
```

Start the service:

```bash
docker compose up -d --build
```

Health check:

```bash
curl http://localhost:8080/health
```

Send a text message:

```bash
curl -X POST http://localhost:8080/send/message \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your_api_key" \
  -d '{
    "message": "hello telegram",
    "parse_mode": "Markdown"
  }'
```

Send a document:

```bash
curl -X POST http://localhost:8080/send/document \
  -H "X-API-Key: your_api_key" \
  -F "file=@./report.txt" \
  -F "caption=nightly report"
```

## Behavior notes

- `/send/message` accepts JSON with `message`, optional `chat_id`, and optional `parse_mode`.
- `/send/document` accepts multipart form data with `file`, plus optional `caption`, `chat_id`, and `parse_mode`.
- If `API_KEY` is empty, the API does not enforce authentication.
- If `TG_CHAT_ID` is unset, callers must provide `chat_id`.
- The API returns `502` when Telegram rejects the request or cannot be reached.

## Troubleshooting

- `401 Invalid API key`: the `X-API-Key` header does not match `API_KEY`.
- `400 chat_id is required`: set `TG_CHAT_ID` or pass `chat_id` with the request.
- `500 TG_BOT_TOKEN is not configured`: set the bot token before sending.
- Empty upload: the API rejects zero-byte files.
- Shell script failures: verify `curl` exists and the target file path is valid.
