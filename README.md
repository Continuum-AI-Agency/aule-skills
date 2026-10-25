# Aulë for After Effects

Let your agent work in your own Adobe After Effects, then turn the project into a Continuum template and render
its variants at scale in the cloud.

- **Edit in After Effects.** Your agent runs ExtendScript in the project you have open, through the Aulë panel.
  Every change is one step you can undo.
- **See it.** Your agent saves a frame as a PNG and looks at its work before sending.
- **Send to Aulë.** One call (or the panel's button) saves a copy of the project with its footage collected,
  uploads it with the fonts it uses, and starts the Forge, which makes it a template. Send again and it
  becomes the template's next revision.
- **Render at scale.** Proof a few variants, then render up to 50 per batch in the Continuum cloud.

## Requirements

- macOS, with [Aulë](https://trycontinuum.ai) installed in `/Applications`
- Adobe After Effects 2024 or later
- A Continuum account with a brand to render in
- An agent that runs local tools: Claude Code, Claude Desktop, Codex, or ChatGPT desktop. ChatGPT on the web and
  claude.ai cannot reach local tools.

## Install

Claude Code:

```sh
claude plugin marketplace add Continuum-AI-Agency/aule-skills
claude plugin install aule@aule
```

Codex or ChatGPT desktop:

```sh
codex plugin marketplace add Continuum-AI-Agency/aule-skills
```

then install **Aulë for After Effects** from the Plugins Directory.

The skill alone, for any agent that reads skills (connect Aulë's MCP server yourself:
`/Applications/Aule.app/Contents/MacOS/aule mcp`):

```sh
npx skills add Continuum-AI-Agency/aule-skills
```

## First run

1. **Open Aulë.** The pilot build isn't notarized yet: the first time, open it from Applications, then allow it in
   System Settings → Privacy & Security → **Open Anyway**.
2. **Sign in with Continuum** and pick the workspace (brand) to render in.
3. **Install the panel:** in Aulë's account menu, **Install After Effects panel** (or let your agent run
   `ae_install`). The panel is unsigned for now, so this turns on Adobe's CEP debug mode, which lets unsigned panels
   load.
4. **In After Effects:** restart it, turn on Settings → Scripting & Expressions → **Allow Scripts to Write Files
   and Access Network**, and open **Window → Extensions → Aulë** (each session, with Aulë running).
5. **Save your project** once: an untitled project can't be sent.

**Send to Aulë** in the panel (or **Send from After Effects** in Aulë's account menu) collects the footage and
fonts, uploads the project, and starts the Forge. Adobe Fonts stay on your computer, because their licence keeps
them off servers: add a licensed copy in Aulë or pick a substitute there. Footage left linked to your computer
(layered PSD/AI imports, image sequences) is listed when you send.

Something wrong? Aulë's logs are in `~/Library/Application Support/ai.trycontinuum.aule/logs/`; send them with a
note on what you did.

## Try it

> Build a 1080×1920 RENDER lower-third in After Effects with the name and title as fields, send it to Aulë, and
> render three proofs.

## How it works

```
agent ──MCP──▶ Aulë (desktop) ──local only──▶ Aulë panel in After Effects ──▶ ExtendScript in your project
                  │
                  └──▶ upload ▶ Forge ▶ template ▶ cloud renders
```

The panel talks only to the Aulë app on your computer (`127.0.0.1`, with a per-launch token). Nothing leaves your
computer until you or your agent send the project to Aulë, which uploads it to your Continuum brand.

## What's here

| Path | What |
| --- | --- |
| `skills/aule-after-effects/` | The skill: connect, edit, make Forge-ready, send and render |
| `mcp.json`, `.mcp.json` | Aulë's local MCP server, for Codex/ChatGPT and Claude Code |
| `plugin.json`, `.claude-plugin/`, `.agents/plugins/` | Plugin and marketplace manifests |

## License

MIT
