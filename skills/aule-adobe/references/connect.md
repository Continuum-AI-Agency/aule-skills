# Connecting to Aulë

## The MCP server

Aulë's own binary serves MCP (`aule mcp`):

| OS | Binary |
| --- | --- |
| macOS | `/Applications/Aule.app/Contents/MacOS/aule` |
| Windows | `%LOCALAPPDATA%\Aule\aule.exe` |
| Linux | `~/.local/bin/aule` (or `/usr/bin/aule` from the `.deb`) |

Not there: ask the person to install Aulë. Then add the server and restart the session:
`claude mcp add aule -- <binary> mcp` or `codex mcp add aule -- <binary> mcp`.

Without MCP the CLI does the same (it starts Aulë if needed): `echo '<json input>' | <binary> call <tool> -`;
`<binary> commands` lists every tool.

## The panels

Adobe's apps run on macOS and Windows. When `adobe_status` doesn't show your app as `connected: true`:

1. `adobe_install {apps: ["<app>"]}` once; its `next` says what the person does for each app.
2. The person restarts the app and opens the panel, with Aulë running: After Effects and Illustrator
   **Window → Extensions → Aulë**; Photoshop **Plugins → Aulë**; Premiere **Window → UXP Plugins → Aulë**.
3. `adobe_status` again until the app is connected.

The panels are unsigned for now: say so. After Effects and Illustrator need Adobe's CEP debug mode, which
`adobe_install` turns on. Photoshop and Premiere may need the UXP Developer Tool when Creative Cloud refuses an unsigned
plugin; `adobe_install` gives the steps.
