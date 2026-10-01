# DevJock for Claude

Connect Claude to your DevJock workspace — your tasks, memories, assistants and skills — through a secure DevJock sign-in. Nothing to install on your computer: no terminal, no Node.js, no Python.

## What you need

- A DevJock account (the login you already use at devjock.ai).
- The Claude Desktop app, or Claude Code.

## Install in Claude Desktop

Claude Desktop has three tabs. **Cowork** and **Code** load plugins; **Chat** does not (see "Using the Chat tab" below).

1. Open Claude Desktop and switch to the **Cowork** or **Code** tab.
2. Open the plugins menu (**+** → **Plugins**) and choose **Add marketplace**.
3. Enter `Devjock-ai/devjock-claude-plugin` and add it.
4. Install the **DevJock** plugin from that marketplace.
5. The first time Claude uses DevJock, a browser window opens asking you to sign in to DevJock. Sign in and approve. Move through the screens promptly — if you take too long, Claude stops waiting and you will need to try again.
6. Ask Claude: **"Initialize DevJock."** Claude loads DevJock's operating context and asks what you want to work on.

## Install in Claude Code

```
/plugin marketplace add Devjock-ai/devjock-claude-plugin
/plugin install devjock@devjock-ai
```

Restart Claude Code, then type `/mcp`, choose **devjock**, and pick **Authenticate** to sign in. Then ask Claude to "initialize DevJock".

## Using the Chat tab

The Chat tab does not load plugins, but it can use DevJock through a **connector**:

1. In Claude Desktop or at claude.ai, open **Customize → Connectors**.
2. Click **+**, then **Add custom connector**.
3. Enter `https://tasks-mcp.devjock.com/mcp/`, name it **DevJock**, save, and sign in with your DevJock account.

**On a company Team or Enterprise plan,** only your organization's **Owner** can add a custom connector. They do it once at claude.ai under **Organization settings → Connectors** (**Add** → **Custom** → **Web**, same address). After that, everyone in the organization can switch it on for themselves under **Customize → Connectors**.

## Troubleshooting

- **Claude says DevJock isn't connected.** Sign in again: in Claude Code type `/mcp` → **devjock** → **Authenticate**; in Claude Desktop reopen the DevJock connector and sign in.
- **The browser says "can't connect" partway through sign-in.** You took a little too long on the approval screens. Try again and move a bit faster.
- **"Initialize DevJock" does nothing in the Chat tab.** The Chat tab doesn't load plugins. Use the connector steps above, or switch to Cowork.

Questions: support@devjock.ai
