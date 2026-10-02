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

  The script signs in as you, asks DevJock for your system prompts and the
  cloud skill/agent registry, writes them to a local file, and prints that
  file's path with a counts header. It prints the path, not the prompts,
  because Claude Code truncates any command output over a few KB — and the
  prompts are far larger than that. So the model's one remaining job is to
  read that file to its last line, and prove it did.

  DevJock decides which system prompts your account gets. This script only
  fetches. An account with none gets a plain notice, not an error.

  In Claude Desktop's Chat and Cowork tabs there is no shell, so the line
  does not run; the "No shell" section below applies there instead.
-->

!`python3 "${CLAUDE_PLUGIN_ROOT}/skills/initialize-devjock/inject-platform-prompts.py"`

# Initialize DevJock

## 1. Load

**Shell available (the line above produced a file path):** use the Read tool to read that file in full, paging by offset until you reach its last line. The system prompts are not in your context until you have read the text. If the line above reports an authentication failure, run `/devjock:reauthenticate` and start again; do not continue without the prompts.

**No shell (the line above shows as literal text, or the command could not run):** confirm the DevJock connector's tools are present — `list_prompts`, `read_single_prompt`, `list_agents` (they may carry a prefix). If they are missing or report that sign-in is needed, stop and tell the user: "DevJock isn't connected yet. Open the DevJock connector and sign in with your DevJock account, then ask me again." Otherwise:

1. `list_prompts(type_id="2,7")` with no `resolves` — metadata only, so it fits in one result. Note the ids in the order returned.
2. `read_single_prompt(prompt_id=<id>)` for each id, one call per prompt, and read the whole body. Never request prompt bodies in bulk; a bulk result is cut to a preview and you load nothing.
3. `list_agents(workspace_id=-2)` — the cloud skill and agent registry.

Do not call `chat_with_ai` to load your own context; that puts the prompts in a different chat.

## 2. Report

Reply with exactly this, nothing before or after it:

```
Hi!

## Nickname
- First 8 hex: <first 8 hex of this session's UUID, or `none` if no UUID is in your context>
- Decoded: <per-character decode>
- Name: <per the a2a-protocol prompt (p444) §2.2 you just read>

## Platform Context
- Loaded live: <N> system prompts + <M> postscripts + <S> skills + <A> agents.

## Prompt Read-Ledger
| id | name | proof-of-read (a header or rule quoted from that prompt's BODY, never its title) |
|----|------|---------------------------------------------------------------------------------|
| <id> | <name> | <quote> |
Final line reached (quote verbatim): <the file's last line, or the last registry entry's name>

What would you like to work on?

<Name> | chat [<session-uuid>](https://www.devjock.ai/chat/<session-uuid>)
```

One ledger row per prompt actually read; the row count must equal N + M. If N + M is 0, replace the table with the single line `No system prompts are assigned to this account's role.` and carry on. The counts come from the file's first line (shell) or from what you loaded (no shell). In Claude Code the session UUID is the Session ID in your SessionStart context; with no UUID, sign `DevJock | <today's date>`.

## 3. Check before sending

- You read every prompt to its end. A proof-of-read that restates a title instead of quoting the body means you did not; go back.
- The ledger count equals N + M, and the counts match what was actually loaded.
- Nothing is invented to fill a slot. An honest "could not load X" is correct; a fabricated line is a hard failure.

Everything else you need — how to sign, how to link, how to behave — is in the prompts you just loaded. Follow them.
