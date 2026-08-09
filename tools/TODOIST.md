# Todoist

The `.env` file provides `TODOIST_API_TOKEN`. Load it without printing it:

```bash
set -a
source .env
set +a
```

Use the current API with a bearer token:

```bash
# List active tasks or projects
curl -fsS "https://api.todoist.com/api/v1/tasks?limit=200" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN"
curl -fsS "https://api.todoist.com/api/v1/projects?limit=200" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN"

# Create an Inbox task
curl -fsS -X POST "https://api.todoist.com/api/v1/tasks" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content":"Task name"}'

# Complete a task
curl -fsS -X POST "https://api.todoist.com/api/v1/tasks/$TASK_ID/close" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN"
```

Responses are JSON. List endpoints return `results` and `next_cursor`; for complete lists, repeat with `cursor=<next_cursor>` until it is `null`. See the [API reference](https://developer.todoist.com/api/v1/). Never print or commit the token.
