# MIDI Butler plugins

Plugins that connect your own AI assistant to [MIDI Butler](https://midibutler.com) and teach it how to use it:
research your instrument's manual, save settings as versions, and ask your browser to send them.
Signing in uses your MIDI Butler account in the browser. There is no key to copy.

MIDI Butler has no device knowledge of its own. SysEx always needs your approval in the browser, and
a send only means your browser wrote to its MIDI output, not that the device accepted it.

## Claude Code

```
claude plugin marketplace add pasaji/midibutler-plugins
claude plugin install midibutler@midibutler
```

Then run `/mcp`, choose the midibutler server and choose Authenticate.

## Claude app (claude.ai and desktop)

Download the zip from https://midibutler.com/plugins/midibutler.zip, then open Customize > Plugins > Add > Upload
plugin. On the plugin's Connectors tab, choose Connect and sign in.

## Codex

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
