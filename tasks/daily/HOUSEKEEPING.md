Parse your persistent memory files and polish and condense them.

## Step 1: check that memory is up to date

For the following files check if their content and references are still up to date and relevant. If not, update them accordingly:

- `./.notes/MEMORY.md`
- `./.notes/GOTCHAS.md`
- `./.notes/RESEARCH.md`
- `./.notes/TIMELINE.md`
- `./.notes/HUMAN.md`

## Step 2: consider research and updates

Look at the memory system and ask yourself the following: "are there decisions that were made in the past that were based on information that might be outdated now?". For example, if we chose certain modules, apis, or versions because they were the best at the time but by now better versions or approaches might exist.

This step is explicitly NOT about updating dependency numbers or versions.

## Step 3: update system

- Check if any of the tools reference environment variables not documented in `.env.example`, add them if they are missing
- Check if git pull is clean, or whether you need to resolve the local state with remote
- Check if there are uncommitted files, if so, commit them with a clear message describing what changed

## Step 4: Summarize activity

Write a `timestamp.log` file in the log directory of this task documenting what you did. Send user a push notification with details.
