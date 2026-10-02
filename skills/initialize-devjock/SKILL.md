---
name: initialize-devjock
description: Start a DevJock session. Loads your DevJock system prompts and the list of DevJock cloud skills and agents so Claude knows how DevJock works before you start. Use when the user says "initialize DevJock", "start DevJock", "load DevJock", or opens a conversation about their DevJock workspace, tasks, memories or assistants.
---

<!--
  HOW THIS FILE WORKS

  The line below that starts with !` is run by Claude Code BEFORE the model
  reads this file. Claude Code executes the command, replaces that line with
  whatever the command printed, and only then hands the file to the model.
  The model cannot skip it or change it.

  The script signs in as you, asks DevJock for your system prompts, the cloud
  skill/agent registry and the session instructions, writes them all to one
  local file, and prints that file's path with a counts header. It prints the
  path, not the content, because Claude Code truncates any command output
  over a few KB — and the content is far larger than that.

  DevJock decides which system prompts your account gets; this script only
  fetches. Everything the model does next is written in DevJock, not here.

  In Claude Desktop's Chat and Cowork tabs there is no shell, so the line
  does not run; the last paragraph below applies there instead.
-->

!`python3 "${CLAUDE_PLUGIN_ROOT}/skills/initialize-devjock/inject-platform-prompts.py"`

# Initialize DevJock

If the output above names a file: read it with the Read tool to its last line, paging by offset, and follow the session instructions at the end of it. If it reports an authentication failure: run `/devjock:reauthenticate`, then start this skill again.

If the line above shows as literal text or could not run (no shell): call `read_single_prompt(prompt_id=776)` through the DevJock connector and follow it. If the connector's tools are missing or ask you to sign in, tell the user: "DevJock isn't connected yet. Open the DevJock connector and sign in with your DevJock account, then ask me again."
