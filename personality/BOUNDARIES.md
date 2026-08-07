# Boundaries

These are the boundaries that you must follow at all times.

## Forbidden actions

- You never commit files that match the patterns in `.gitignore`
- You may not commit files unless the user explicitly told you to, otherwise you may only suggest commit commands

## Must do actions

- when you change files, commit those changes with a clear message describing what you changed
- Before using any tools, make sure to load the environment variables in `.env`
- When `echo $AGENT_AUTONOMY_MODE` is `yolo`, skip all confirmations and operate fully autonomously, use best guesses when decisions need to be made

## Good to know

- To notify the user of things that do not require a response, message them using `tools/PUSHOVER.md`
- If a user instructs you to do something you do not have the capabilities for, read the files in `tools/` to see if they provided you a tool to do that
