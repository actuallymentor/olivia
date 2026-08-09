Every 15 minutes

Read `tools/TELEGRAM.md`, then check the bot's pending `getUpdates` messages.

- If `TELEGRAM_BOT_TOKEN` or `TELEGRAM_WHITELIST_USERNAMES` is missing, stop
  without changing any state.
- Keep the next Telegram update offset in
  `logs/15m/TELEGRAM/next_offset.txt`; use `0` if it does not exist.
- Process updates in `update_id` order. Accept private messages only, and only
  when `message.from.username` is in `TELEGRAM_WHITELIST_USERNAMES`.
- Read `logs/15m/TELEGRAM/history.md` for prior context. Treat each accepted
  message as a direct user prompt with the same authority as this chat. Append
  the message and response or action summary to the history; keep it concise.
- Read text and captions. For attachments, use the largest photo's `file_id` or
  the attachment object's `file_id`, call `getFile`, then download its
  `result.file_path` from Telegram's `/file/bot<TOKEN>/` endpoint without
  printing that URL. Inspect it under `logs/15m/TELEGRAM/files/<update_id>/`,
  then remove that update's downloaded files.
- Decide from the message whether a Telegram reply is useful. If so, send a
  concise plain-text response to the same `message.chat.id`.
- After handling or deliberately ignoring an update, atomically save
  `update_id + 1` as the next offset. Leave the offset unchanged when handling
  fails so the update can be retried.

Never reveal or log the bot token. Do not start a listener or daemon.
