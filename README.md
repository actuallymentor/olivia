# Olivia

Olivia is the **O**pen **L**LM **I**ngestion **V**ertically **I**ntegrated **A**ssistant.

Most of the tasks you need your "agent" to perform are glorified crontab's that need to read a markdown file and run some commands. Olivia is an example of how you could run an agent of arbitrary complexity without installing the agent suite du jour.

## Requirements

You are expected to have an authenticated coding agent like Codex or Claude installed. It is up to you whether you run them in a sandboxed environment or not.

Personally I use [babysit](https://github.com/actuallymentor/babysit) which also runs the loops for me.

## Usage

**Step 1: set up markdown files**

This repository contains the files as I use them. They are not per se designed to be used as-is. Edit as you see fit. I might keep this public or not. I recommend forking this repository and cloning that to the machine you want to run Olivia on.

You will need to edit the following files:

- `.env` - this file contains the environment variables that Olivia will use, unless you change her instructions
- `LOOP.md` - this file **IS** Olivia. This is what will be executed in a loop
- `personality/*` - files that contain how Olivia acts
- `tasks/*` - files that contain the tasks Olivia can perform
- `tools/*` - files that contain the tools Olivia can use

**Step 2: start the loop**

If you use `babysit`, you can use `babysit codex --yolo --loop --name "Olivia"`. If you use a vanilla coding agent, add something like this to your crontab:

```bash
0 * * * * cd ~/path-to-olivia-repo && claude -p "Read and execute the steps in LOOP.md"
```

If you run the agent sandboxed like I do, you could consider adding the `--dangerously-skip-permissions` flag. In `babysit` using `--yolo` already does that, and sets `AGENT_AUTONOMY_MODE=yolo` which you could reference in your markdown files.
