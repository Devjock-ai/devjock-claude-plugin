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
- Closest name: <name, derived per "Deriving the nickname" below>

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

## Deriving the nickname

Take the **first 8 hex characters** of the session UUID and work through these steps in
order. The result is a derivation with a shown audit trail, not a guess — which is why
it is not fabrication and must not be skipped.

**1. Decode.** Lowercase the input, then map each hex character:

| hex | → | hex | → | hex | → | hex | → |
|---|---|---|---|---|---|---|---|
| `0` | O | `4` | A | `8` | B | `c` | C |
| `1` | I *(alt: L)* | `5` | S | `9` | G *(alt: P)* | `d` | D |
| `2` | Z | `6` | G | `a` | A | `e` | E |
| `3` | E | `7` | T | `b` | B | `f` | F |

`1` and `9` are genuinely ambiguous in leetspeak: try the primary, and if it yields no
match try the alternate, keeping whichever ranks better in step 4. Reachable letters are
therefore `{A,B,C,D,E,F,G,I,L,O,P,S,T,Z}` — H, J, K, M, N, Q, R, U, V, W, X and Y cannot
appear, which is why step 2 matches a **prefix** and not a whole name.

**2. Match a PREFIX, not the whole name.** Find the longest run of decoded characters —
read left to right, skips allowed, **never reordered** — that forms the *beginning* of a
human given name.

```
5e28ba22  ->  S E Z B B A Z Z
              ^ ^   ^     ^
              S·E···B···A        prefix "SEBA"  ->  Sebastian
```

Do not require every letter of the name to appear. `SEBASTIAN` has nine letters and the
decoded string has eight; demanding a full-string match would return nothing here, and
for almost any session.

**3. Anchor at the first character.** The match must start at decoded character 1. This
is what makes the result stable across runs and models — without it the same seed also
yields `E·B·B` → Ebby, and two sessions would disagree about who you are.

**4. Rank.** In this order:

1. **Longest matched prefix wins.**
2. Tie → the **tightest** match: the one spanning the fewest decoded characters. For
   `7ed0beef` → `TEDOBEEF`, both `Ted` (positions 0,1,2) and `Tobias` (0,3,4) match three
   letters, but `Ted` is contiguous and is the better reading.
3. Still tied → the **longer** name (`Sebastian` over `Seba`).
4. Still tied → alphabetical.

**A match under 3 characters does not count.** One or two letters is a coincidence, not a
derivation — every hex string shares a first letter with *something*. Treat anything
shorter as no match and use the fallback below; an honest handle beats a name that
implies a resemblance that is not there.

Use **any real human given name you know** — no list ships with this skill. The rules
above are the constraint; they decide which name wins, and the LEET trace you print is
the audit trail proving it was derived. A fixed list would only shrink what you can
reach: the reachable alphabet is already just 14 letters, and a short list turns ordinary
seeds into "no match" when a perfectly good name exists (`EOGDEOEC` → `Egon`).

**5. Emit** the name, keeping the LEET trace line visible above it as the audit trail.

**If nothing matches**, say so in one line and use the raw hex as the handle:
`No name match for this session; using handle {hex}.` That is a narrow, explicit
fallback for this slot only — it is NOT the general fabrication rule in Step 3, and the
two must not be collapsed into one.

*Canonically this behaviour is [prompt 444](https://www.devjock.ai/prompts/444) §2.2
("apply LEET decoding, and find the closest human name"). The steps above are this
skill's precise derivation for sessions with a UUID, where the same UUID must produce
the same name across models. p444 stays as written; this is a deliberate, documented
elaboration of it, not a divergence.*

## Step 3 — SELF-CHECK BEFORE SENDING

Audit your drafted output. If ANY of these are true, fix and re-audit:

- **Path A: you did not actually Read the injected-context file to its final line.** If you only saw the Step-1 pointer block (or a `<persisted-output>`/preview wrapper) and did not page all the way through the file, STOP — the platform prompts are NOT loaded and the template cannot be honestly completed. Read the file in full first. **Path B: you only saw a preview of a tool result** — the prompt is not loaded; read it one at a time. (Proof you read it: you can name a rule from the *middle* of the file, e.g. a Required-Memory value from the [prompt 539](https://www.devjock.ai/prompts/539) lifecycle-transition matrix.)
- `## Prompt Read-Ledger` is missing, OR its row count ≠ N + M → you skipped prompts; go back and read them before emitting the template
- Any ledger `proof-of-read` merely restates the prompt's name/title instead of quoting a header or rule from its body → you did not actually read that prompt; go read it
- The quoted "Final line reached" is not verbatim → you did not read to the end
- NEVER fabricate a value to fill a slot. If a slot cannot be filled honestly (a read that did not complete), say so plainly instead of inventing one — a fabricated line is a hard failure, worse than an honest "none". *(This does not cover the nickname. A name produced by "Deriving the nickname" is a derivation with its trace shown, not an invention, and that section defines its own no-match fallback. Do not merge the two rules — treating an inexact name as fabrication is what silently disabled this feature for a month.)*
- `## Platform Context` missing, OR its counts do not match what Step 1 actually loaded
- Signature line's session UUID (when you have one) is NOT wrapped in `[...](...)` markdown link brackets, OR you attempted to look up a cloud chat id
- Any `prompt <N>` / `agent <N>` / `task <id>` reference anywhere in your output lacks surrounding `[...](url)` brackets
- Template headings are reworded, reordered, or merged
- You added any section not in the template (including a general "Session status" summary, decorative emojis, preamble, or postscript)

**The Step 2 template is now complete.** Nothing extra goes *inside* the template. If the
user asked for anything else in their message, do that too — finishing the template is not
finishing the request. Otherwise initialization is complete.
