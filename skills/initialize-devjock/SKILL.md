---
name: initialize-devjock
description: Start a DevJock session. Loads DevJock's live operating context from the cloud through the DevJock connector so Claude knows how DevJock works before you start. Use when the user says "initialize DevJock", "start DevJock", "load DevJock", or opens a conversation about their DevJock workspace, tasks, memories or assistants.
---

# Initialize DevJock

The instructions for starting a DevJock session live in the DevJock cloud, not in this plugin, so they are always current. Fetch them and follow them.

1. Confirm the DevJock connector's tools are available to you — in particular `read_single_prompt`, `list_prompts` and `list_agents` (they may appear with a prefix such as `mcp__plugin_devjock_devjock__`).
   - If they are missing or report that sign-in is needed, stop and tell the user: "DevJock isn't connected yet. Open the DevJock connector and sign in with your DevJock account, then ask me again." In Claude Code the user can type `/mcp`, choose **devjock**, and pick **Authenticate**. Do not invent DevJock context from memory.
2. Call `read_single_prompt` with `prompt_id` **776**. That prompt is the DevJock session-initialization workflow.
3. Follow it exactly as written. This plugin ships no local scripts, so take its **Path B (MCP only)** directly; do not try to run a shell command.
