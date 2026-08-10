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

## Check access

List one available model to verify the account ID, token, and Workers AI
permissions without running inference:

```bash
curl -fsS --get \
  "https://api.cloudflare.com/client/v4/accounts/$CLOUDFLARE_ACCOUNT_ID/ai/models/search" \
  -H "Authorization: Bearer $CLOUDFLARE_WORKERS_AI_TOKEN" \
  --data-urlencode "per_page=1" |
  jq '{success, errors, messages, model: .result[0].name}'
```

A successful Cloudflare API response has `"success": true`. Errors are returned
in the `errors` array. The token needs Workers AI permissions for this account;
Cloudflare's Workers AI token template is the safest default.

## Run a text-generation model

The REST endpoint is
`POST /client/v4/accounts/{account_id}/ai/run/{model_name}`. Model names include
their `@cf/...` namespace:

```bash
MODEL='@cf/meta/llama-3.1-8b-instruct'

curl -fsS -X POST \
  "https://api.cloudflare.com/client/v4/accounts/$CLOUDFLARE_ACCOUNT_ID/ai/run/$MODEL" \
  -H "Authorization: Bearer $CLOUDFLARE_WORKERS_AI_TOKEN" \
  -H 'Content-Type: application/json' \
  --data '{"prompt":"Explain edge computing in one sentence."}' |
  jq
```

The response envelope contains `result`, `success`, `errors`, and `messages`.
Inputs and result shapes vary by model task, so check the selected model's page
before changing the JSON body. Inference can consume Workers AI usage or credits.

## Find models

Search the account-visible model catalog by name, description, or task:

```bash
curl -fsS --get \
  "https://api.cloudflare.com/client/v4/accounts/$CLOUDFLARE_ACCOUNT_ID/ai/models/search" \
  -H "Authorization: Bearer $CLOUDFLARE_WORKERS_AI_TOKEN" \
  --data-urlencode 'search=llama' \
  --data-urlencode 'per_page=20' |
  jq '.result[] | {name, task: .task.name}'
```

See Cloudflare's official [Workers AI REST guide](https://developers.cloudflare.com/workers-ai/get-started/rest-api/),
[model catalog](https://developers.cloudflare.com/workers-ai/models/), and
[Execute AI model API reference](https://developers.cloudflare.com/api/resources/ai/methods/run/).
