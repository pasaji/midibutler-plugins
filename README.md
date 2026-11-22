# MIDI Butler plugins

Plugins that connect your own AI assistant to [MIDI Butler](https://midibutler.com) and teach it how to use it:
research your instrument's manual, save settings as versions, and ask your browser to send them.
Signing in uses your MIDI Butler account in the browser. There is no key to copy.

MIDI Butler has no device knowledge of its own. SysEx always needs your approval in the browser, and
a send only means your browser wrote to its MIDI output, not that the device accepted it.

Claude and Codex each come as an app and as a terminal tool. Pick the one you actually use.

## Apps

### Claude app (claude.ai and the Claude desktop app)

Open Customize > Plugins > Add > Add marketplace > Add from a repository, enter pasaji/midibutler-plugins,
then add the MIDI Butler plugin from Discover. On the plugin's Connectors tab, choose Connect and sign in.

Without the plugin: Settings > Connectors > Add custom connector, with the address
https://psaxiiqwtwrkomtkwtvv.supabase.co/functions/v1/mcp.

### ChatGPT app (the ChatGPT desktop app, Codex mode)

Open Plugins > Add > Add a marketplace and enter pasaji/midibutler-plugins. Install MIDI Butler from the
plugin list, then open the plugin, choose Authenticate and sign in.

If you already added the plugin with the Codex CLI commands below, the ChatGPT app shares those
settings and shows it too.

Without the plugin: Plugins > Add > Add custom MCP server, with the address
https://psaxiiqwtwrkomtkwtvv.supabase.co/functions/v1/mcp.

The ChatGPT app steps are not yet fully verified.

## Terminal tools

### Claude Code

```
claude plugin marketplace add pasaji/midibutler-plugins
claude plugin install midibutler@midibutler
```

Then run `/mcp`, choose the midibutler server and choose Authenticate.

### Codex CLI

```
codex plugin marketplace add pasaji/midibutler-plugins
codex plugin add midibutler@midibutler
```

Then sign in to MIDI Butler (your browser opens) and start a new Codex session:

```
codex mcp login midibutler
```

Installing the plugin works with Codex 0.135. The sign-in step has not been verified with every Codex
version.

## Licence

MIT. The MIDI Butler name and logo are not licensed.
