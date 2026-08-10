# Cloudflare Workers AI

The `.env` file provides `CLOUDFLARE_ACCOUNT_ID` and
`CLOUDFLARE_WORKERS_AI_TOKEN`. Load them without printing them:

```bash
set -a
source .env
set +a
```

Never print, log, commit, or paste the token. Use it only in the
`Authorization` header. If it is exposed, revoke it in the Cloudflare dashboard
and replace the value in `.env`.

Workers AI calls use account-scoped endpoints under:

```text
https://api.cloudflare.com/client/v4/accounts/$CLOUDFLARE_ACCOUNT_ID/ai/
```

Model input and output formats differ. Check the selected model's documentation
before changing its body, query parameters, or content type. Inference consumes
Workers AI usage; see [pricing](https://developers.cloudflare.com/workers-ai/platform/pricing/)
and [limits](https://developers.cloudflare.com/workers-ai/platform/limits/).

## Check access

Model search verifies the account ID, token, and Workers AI permissions without
running inference:

```bash
curl -fsS --get \
  "https://api.cloudflare.com/client/v4/accounts/$CLOUDFLARE_ACCOUNT_ID/ai/models/search" \
  -H "Authorization: Bearer $CLOUDFLARE_WORKERS_AI_TOKEN" \
  --data-urlencode 'per_page=1' |
  jq '{success, errors, messages, model: .result[0].name}'
```

A successful Cloudflare API response has `"success": true`; failures are
described in the `errors` array. A custom token normally needs Workers AI Read
and Edit permissions. Write operations such as uploading a LoRA require Edit.

## Audio

### Speech to text

This sends an MP3 directly to Deepgram Nova-3 and prints its best transcript:

```bash
AUDIO_FILE='/absolute/path/to/recording.mp3'
MODEL='@cf/deepgram/nova-3'

curl -fsS -X POST \
  "https://api.cloudflare.com/client/v4/accounts/$CLOUDFLARE_ACCOUNT_ID/ai/run/$MODEL?language=en" \
  -H "Authorization: Bearer $CLOUDFLARE_WORKERS_AI_TOKEN" \
  -H 'Content-Type: audio/mpeg' \
  --data-binary "@$AUDIO_FILE" |
  jq -r '.result.results.channels[0].alternatives[0].transcript'
```

Set `Content-Type` to the actual audio type. Nova-3 defaults to US English;
choose a supported language code or replace `language=en` with
`detect_language=true`. See the [Nova-3 model page](https://developers.cloudflare.com/workers-ai/models/nova-3/)
for languages, diarization, punctuation, utterances, and other options.

For transcription plus speech-to-English translation, timestamps, and VTT
output, use [`@cf/openai/whisper-large-v3-turbo`](https://developers.cloudflare.com/workers-ai/models/whisper-large-v3-turbo/).
Cloudflare's [large-audio tutorial](https://developers.cloudflare.com/workers-ai/guides/tutorials/build-a-workers-ai-whisper-with-chunking/)
shows how to Base64-encode and chunk recordings before sending them to Whisper.

### Text to speech

Aura returns raw audio rather than a JSON envelope. Save the response directly
to a file:

```bash
OUTPUT_FILE='/absolute/path/to/speech.mp3'
MODEL='@cf/deepgram/aura-1'

curl -fsS -X POST \
  "https://api.cloudflare.com/client/v4/accounts/$CLOUDFLARE_ACCOUNT_ID/ai/run/$MODEL" \
  -H "Authorization: Bearer $CLOUDFLARE_WORKERS_AI_TOKEN" \
  -H 'Content-Type: application/json' \
  --data '{
    "text": "Hello from Cloudflare Workers AI.",
    "speaker": "asteria",
    "encoding": "mp3"
  }' \
  --output "$OUTPUT_FILE"

test -s "$OUTPUT_FILE" && file "$OUTPUT_FILE"
```

The [Aura model page](https://developers.cloudflare.com/workers-ai/models/aura-1/)
lists speakers, encodings, containers, sample rates, and bit rates. For
multilingual speech, see [`@cf/myshell-ai/melotts`](https://developers.cloudflare.com/workers-ai/models/melotts/).

### Real-time audio

Cloudflare also offers WebSocket-capable models for streaming transcription,
speech generation, and voice-turn detection. Start with the
[audio models in the catalog](https://developers.cloudflare.com/workers-ai/models/)
and the [`smart-turn-v2` model](https://developers.cloudflare.com/workers-ai/models/smart-turn-v2/).

## Other capabilities

Actual access depends on the token's permissions and each model's availability.

- [Text generation, reasoning, vision, translation, classification, summarization, embeddings, reranking, image generation, image understanding, and object detection](https://developers.cloudflare.com/workers-ai/models/)
- [Function calling](https://developers.cloudflare.com/workers-ai/features/function-calling/) and [structured JSON output](https://developers.cloudflare.com/workers-ai/features/json-mode/)
- [Asynchronous batch inference](https://developers.cloudflare.com/workers-ai/features/batch-api/rest-api/)
- [Document and image conversion to Markdown](https://developers.cloudflare.com/workers-ai/features/markdown-conversion/usage/rest-api/)
- [Fine-tuned inference with LoRA adapters](https://developers.cloudflare.com/workers-ai/features/fine-tunes/loras/) — requires Workers AI Edit
- [Model and task discovery through the API](https://developers.cloudflare.com/api/resources/ai/subresources/models/methods/list/)

See the official [Workers AI REST guide](https://developers.cloudflare.com/workers-ai/get-started/rest-api/)
and [Execute AI model API reference](https://developers.cloudflare.com/api/resources/ai/methods/run/)
for the shared authentication and response conventions.
