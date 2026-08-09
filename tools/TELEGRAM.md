# Telegram Bot API

The `.env` file provides `TELEGRAM_BOT_TOKEN` and the comma-separated
`TELEGRAM_WHITELIST_USERNAMES`. Load them without printing them:

```bash
set -a
source .env
set +a

TELEGRAM_API="https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN"
```

Never print, log, commit, or paste the token or `TELEGRAM_API` value. The token is
part of every Bot API URL. If it is exposed, revoke it with
[@BotFather](https://t.me/BotFather) and replace the value in `.env`.

## Check the connection

`getMe` verifies the token and returns the bot's profile:

```bash
curl -fsS "$TELEGRAM_API/getMe" | jq
```

Bot API responses are JSON. A successful response has `"ok": true` and its
payload in `result`. Failed responses have `"ok": false`, an `error_code`,
and a human-readable `description`.

## Find a chat ID

Ask the recipient to open the bot in Telegram, press **Start**, and send it a
message. Then retrieve the update:

```bash
curl -fsS --get "$TELEGRAM_API/getUpdates" \
  --data-urlencode "timeout=30" |
  jq '.result[] | {
    update_id,
    chat_id: (.message.chat.id // .channel_post.chat.id),
    text: (.message.text // .channel_post.text)
  }'
```

Save the relevant `chat_id` separately if the bot sends to that chat often.
Group and channel IDs can be negative. A bot must be added to a group or channel
and granted the permissions required for what it needs to do.

## Whitelist senders

`TELEGRAM_WHITELIST_USERNAMES` contains usernames without the leading `@`,
separated by commas and without spaces:

```dotenv
TELEGRAM_WHITELIST_USERNAMES=actuallymentor,another_username
```

Check every incoming update before processing it or sending a response. Normalize
usernames by removing `@` and comparing them case-insensitively. If the username
is absent or not whitelisted, ignore the update:

```bash
is_whitelisted() {
  local username="${1#@}"
  local whitelisted_username
  local -a whitelist

  [[ -n "$username" ]] || return 1

  IFS=, read -ra whitelist <<< "$TELEGRAM_WHITELIST_USERNAMES"

  for whitelisted_username in "${whitelist[@]}"; do
    whitelisted_username="${whitelisted_username#@}"
    [[ -n "$whitelisted_username" ]] || continue

    if [[ "${whitelisted_username,,}" == "${username,,}" ]]; then
      return 0
    fi
  done

  return 1
}

username=$(jq -r '.message.from.username // empty' <<< "$UPDATE_JSON")

if ! is_whitelisted "$username"; then
  exit 0
fi
```

Telegram usernames can change. Update the whitelist if a trusted user renames
their account.

`getUpdates` and webhooks are mutually exclusive. If polling returns a webhook
conflict, inspect or remove the webhook:

```bash
curl -fsS "$TELEGRAM_API/getWebhookInfo" | jq
curl -fsS -X POST "$TELEGRAM_API/deleteWebhook" | jq
```

## Send a text message

Use form fields so shell metacharacters, Unicode, spaces, and line breaks are
encoded safely:

```bash
CHAT_ID="123456789"
MESSAGE=$'First line\n\nSecond line'

curl -fsS -X POST "$TELEGRAM_API/sendMessage" \
  --data-urlencode "chat_id=$CHAT_ID" \
  --data-urlencode "text=$MESSAGE" |
  jq
```

Text messages may be up to 4,096 characters after entity parsing. Telegram also
supports optional `parse_mode` values such as `HTML` and `MarkdownV2`; plain
text is safest unless formatting is needed.

## Send a photo or document

Upload a local file with multipart form data:

```bash
CHAT_ID="123456789"

curl -fsS -X POST "$TELEGRAM_API/sendPhoto" \
  -F "chat_id=$CHAT_ID" \
  -F "photo=@/absolute/path/to/photo.jpg" \
  -F "caption=Optional caption" |
  jq

curl -fsS -X POST "$TELEGRAM_API/sendDocument" \
  -F "chat_id=$CHAT_ID" \
  -F "document=@/absolute/path/to/file.pdf" |
  jq
```

## Receive updates

For a small local bot, use long polling. After processing a batch, request the
next batch with an offset one greater than the largest processed `update_id`;
otherwise Telegram will return the same updates again:

```bash
OFFSET="0"

curl -fsS --get "$TELEGRAM_API/getUpdates" \
  --data-urlencode "offset=$OFFSET" \
  --data-urlencode "timeout=30" |
  jq
```

For a deployed bot, register a public HTTPS webhook. Generate a separate random
webhook secret and verify the `X-Telegram-Bot-Api-Secret-Token` header on every
incoming request:

```bash
WEBHOOK_URL="https://example.com/telegram"
WEBHOOK_SECRET="replace-with-a-random-secret"

curl -fsS -X POST "$TELEGRAM_API/setWebhook" \
  --data-urlencode "url=$WEBHOOK_URL" \
  --data-urlencode "secret_token=$WEBHOOK_SECRET" |
  jq
```

Return an HTTP 2xx response promptly after accepting a webhook update. Deduplicate
updates by `update_id`, because delivery can be retried.

See the official [Telegram Bot API reference](https://core.telegram.org/bots/api)
for every available method, parameter, object, and current limit.
