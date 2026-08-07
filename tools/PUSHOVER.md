You can send the user messages on their phone using Pushover. For more info, you can read the [API docs](https://pushover.net/api) but you shouldn't need to.

You can send a pushover notification using the following command:

```bash
TITLE="Olivia: <message title>"

# Note the use of $'...' to allow for newlines in the message
MESSAGE=$'Message line 1\n\nMessage line 2\nMessage line with only a linebreak but no space\n\nMessage line 3'

# URL is optional, it allows the user to easily click through to a resource. If you don't want to include a URL, just leave it as an empty string.
URL=""

# Priority is optional, it can be -2, -1, 0, 1, or 2. Default is 0.
# -2: no notification, -1 no sound or vibration but shows notification, 0 regular notification that respects OS settings, 1: overrides do not disturb, 2: like 1 but repeated until acknowledged.
# use high priority only for calamity or when explicitly instructed, remember that the user sleeps at night (Timezone $TIMEZONE, assume Amsterdam/Europe is unset)
PRIORITY="0"

# Check for $PUSHOVER_TOKEN and $PUSHOVER_USER environment variables, if they are not set, check in .babysitrc file, if they are not set there, do not send a notification and log a warning
curl -f -X POST -d "token=$PUSHOVER_TOKEN&user=$PUSHOVER_USER&title=$TITLE&message=$MESSAGE&url=$URL&priority=$PRIORITY" https://api.pushover.net/1/messages.json
```