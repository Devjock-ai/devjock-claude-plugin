---
name: initialize-devjock
description: Start a DevJock session. Loads DevJock's live operating context — the system prompts for your role, plus the cloud skill and agent registry — so Claude knows how DevJock works before you start. Use when the user says "initialize DevJock", "start DevJock", "load DevJock", or opens a conversation about their DevJock workspace, tasks, memories or assistants.
---

# DEVJOCK SESSION INITIALIZATION

**Initialization is THREE steps.** Read all three before you start:

1. Load the platform prompts.
2. Emit the strict output template.
3. Self-check the template.

## Step 1 — LOAD PROMPTS

DevJock serves each account the **system prompt set for its role**. There are three sets, kept as agent templates: `/load-workspace-user-system-prompts` (477), `/load-workspace-admin-system-prompts` (190) and `/load-platform-admin-system-prompts` (213). The server decides which prompts your account may read; this skill only fetches. An account whose role has no system prompts loads none — that is a valid result, not a failure.

Pick the loader by the tools you actually have. Do not guess from branding.

### Path A — you have a shell (Bash) tool AND a file Read tool

This is Claude Code, or Claude Desktop's Code tab. Run the injector:

!`python3 "${CLAUDE_PLUGIN_ROOT}/skills/initialize-devjock/inject-platform-prompts.py"`

**The injector does NOT inline the platform prompts.** The full context can be ~50k tokens — far larger than a `!`command`` result can inline. If it tried, the harness would SILENTLY truncate the output to a ~2KB preview, and you would proceed having loaded almost nothing. So the injector **writes the full context to a file and prints only that file's path plus the counts header**.

**Before doing Step 2, you MUST use the Read tool to read that file IN FULL.** It can be larger than the Read tool's per-call cap (~25k tokens), so page through it — offset by offset — until you reach its final line (the type-7 postscript, prompt 692, followed by the Skill & Agent Registry). The prompts are **NOT in your context window until you have actually read the file.** Do not call `list_prompts` / `list_agents` to re-read them.

The counts for the Step 2 template come from the injector's header comment (`N platform + M postscript + S skills + A agents`) — also the first line of the file. If the header says 0 platform prompts, the account's role has no system prompts: say so in the template and carry on.

**If the injector printed an auth-failure line instead of a file path** (token missing or expired), do NOT fabricate a load — run `/devjock:reauthenticate` and re-run Step 1. If the shell command itself cannot run (no `python3`, the `!` line shows as literal text, or `CLAUDE_PLUGIN_ROOT` is unset), fall back to Path B.

### Path B — DevJock connector only (no shell)

This is Claude Desktop's Chat or Cowork tab, or a cloud session. Confirm the DevJock connector's tools are present — `read_agent`, `read_single_prompt`, `list_agents` (they may carry a prefix such as `mcp__plugin_devjock_devjock__`). If they are missing or report that sign-in is needed, stop and tell the user: "DevJock isn't connected yet. Open the DevJock connector and sign in with your DevJock account, then ask me again." Do not invent DevJock context from memory.

Do NOT page `list_prompts` with bodies: one page is larger than a tool result may be, so it is silently cut to a preview and you load nothing. Load **one prompt per call** instead:

1. Call `read_agent(agent_id=213)`, `read_agent(agent_id=190)` and `read_agent(agent_id=477)`. Take the union of their `prompt_ids`, keeping the order in which ids first appear (213's list first). Append `692`, the postscript.
2. For each id, call `read_single_prompt(prompt_id=<id>)` and read the whole body. **An "access denied" or "not found" answer is the server saying this prompt is not for your role — skip it silently and go on.** Do not retry it, and do not try another route to it.
3. Call `list_agents(workspace_id=-2)`. That is the Skill & Agent Registry: the DevJock cloud skills and agents you can invoke via `chat_with_ai` (`skill_ids=[id]` for a skill, `agent_id=id` or `task_id` for an assistant).

Your counts for Step 2 are what you actually loaded: N = prompts read whose `type_id` is 2, M = prompts read whose `type_id` is 7, S and A = registry items whose `category` is `skill` / `agent`. If N is 0, say so in the template and carry on.

**Do NOT call `chat_with_ai` to load your own context.** That spawns a separate chat; the prompts land in *its* context, not yours.

## Step 2 — OUTPUT (STRICT TEMPLATE)

Your response to the user MUST match this template exactly. Fill in every `<…>` placeholder. Do not add sections. Do not omit sections. Do not reword the headings.

```
Hi!

## Nickname (prompt 444 §2)
- First 8 hex: <first 8 hex of this session's UUID. If no session UUID is available to you (Path B), write `none` and skip the two lines below>
- LEET trace: <per-char decode, e.g. "0→O, 5→S, 8→B, f→F, 5→S, c→C, e→E, f→F">
- Closest name: <name, derived per the a2a-protocol prompt (p444) §2.2, which is in the system prompts you just read>

## Platform Context
- Loaded live (Path <A or B>): <N> platform prompts + <M> postscripts + <S> skills + <A> agents.

## Prompt Read-Ledger
(one row per prompt actually read, in reading order; the row count MUST equal N + M. If N + M is 0, write one line: `No system prompts are assigned to this account's role.` instead of the table.)
| id | name | proof-of-read (a header or rule quoted from that prompt's BODY, never its title) |
|----|------|---------------------------------------------------------------------------------|
| <id> | <name> | <a header or specific rule quoted from that prompt's body> |
Final line reached (quote verbatim): <Path A: the injected-context file's actual last line. Path B: the last registry entry's name>

What would you like to work on?

<Name> | chat [<claude-code-session-uuid>](https://www.devjock.ai/chat/<claude-code-session-uuid>)
```

**`<claude-code-session-uuid>` = this session's UUID** (in Claude Code, the Session ID from the SessionStart context — the same UUID the nickname is derived from). When no session UUID is available (Path B), sign `DevJock | <today's date>` instead. Do NOT call `list_chat_sessions`, `chat_with_ai`, or any other lookup to find or create a chat id during init.

## Step 3 — SELF-CHECK BEFORE SENDING

Audit your drafted output. If ANY of these are true, fix and re-audit:

- **Path A: you did not actually Read the injected-context file to its final line.** If you only saw the Step-1 pointer block (or a `<persisted-output>`/preview wrapper) and did not page all the way through the file, STOP — the platform prompts are NOT loaded and the template cannot be honestly completed. Read the file in full first. **Path B: you only saw a preview of a tool result** — the prompt is not loaded; read it one at a time. (Proof you read it: you can name a rule from the *middle* of the file, e.g. a Required-Memory value from the [prompt 539](https://www.devjock.ai/prompts/539) lifecycle-transition matrix.)
- `## Prompt Read-Ledger` is missing, OR its row count ≠ N + M → you skipped prompts; go back and read them before emitting the template
- Any ledger `proof-of-read` merely restates the prompt's name/title instead of quoting a header or rule from its body → you did not actually read that prompt; go read it
- The quoted "Final line reached" is not verbatim → you did not read to the end
- NEVER fabricate a value to fill a slot. If a slot cannot be filled honestly (a read that did not complete), say so plainly instead of inventing one — a fabricated line is a hard failure, worse than an honest "none".
- `## Platform Context` missing, OR its counts do not match what Step 1 actually loaded
- Signature line's session UUID (when you have one) is NOT wrapped in `[...](...)` markdown link brackets, OR you attempted to look up a cloud chat id
- Any `prompt <N>` / `agent <N>` / `task <id>` reference anywhere in your output lacks surrounding `[...](url)` brackets
- Template headings are reworded, reordered, or merged
- You added any section not in the template (including a general "Session status" summary, decorative emojis, preamble, or postscript)

**The Step 2 template is now complete.** Nothing extra goes *inside* the template. If the
user asked for anything else in their message, do that too — finishing the template is not
finishing the request. Otherwise initialization is complete.
