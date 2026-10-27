# Aulë for Adobe

Let your agent work in your own Adobe apps — **After Effects, Photoshop, Illustrator and Premiere Pro** — then turn
the work into a Continuum template and render its variants at scale in the cloud.

- **Edit in your app.** Your agent runs scripts in what you have open, through the Aulë panel. Every change is one
  step you can undo (Illustrator: a few steps per change).
- **See it.** Your agent saves a frame as a PNG and looks at its work before sending (not in Premiere yet).
- **Send to Aulë.** One call (or the panel's button) saves a copy with its media and fonts, uploads it, and starts the
  Forge, which makes it a template. Send again and it becomes the template's next revision. Photoshop, Illustrator
  and Premiere work is rebuilt as an After Effects template in the cloud (Premiere: effects, transitions and colour
  grading are not carried).
- **Render at scale.** Proof a few variants, then render up to 50 per batch in the Continuum cloud.

## Requirements

- macOS, with [Aulë](https://trycontinuum.ai) installed in `/Applications`
- Any of: After Effects 2024 or later, Illustrator 2024 or later, Photoshop 2024 (25.0) or later, Premiere Pro 25.6
  or later
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
3. **Install the panels:** in Aulë's account menu, **Install Adobe panels** (or let your agent run `adobe_install`).
   The panels are unsigned for now: After Effects and Illustrator need Adobe's CEP debug mode, which this turns on;
   if Creative Cloud refuses the unsigned Photoshop or Premiere plugin, Aulë tells you how to load it with Adobe's
   UXP Developer Tool.
4. **In your app:** restart it and open the panel, with Aulë running: After Effects and Illustrator **Window →
   Extensions → Aulë** (After Effects also needs Settings → Scripting & Expressions → **Allow Scripts to Write Files
   and Access Network**); Photoshop **Plugins → Aulë**; Premiere **Window → UXP Plugins → Aulë**.
5. **Save your file** once: an untitled one can't be sent.

**Send to Aulë** in the panel (or **Send from** in Aulë's account menu) collects the media and fonts, uploads the
file, and starts the Forge. Adobe Fonts stay on your computer, because their licence keeps
them off servers: add a licensed copy in Aulë or pick a substitute there. Footage left linked to your computer
(After Effects: layered PSD/AI imports, image sequences) is listed when you send.

Something wrong? Aulë's logs are in `~/Library/Application Support/ai.trycontinuum.aule/logs/`; send them with a
note on what you did.

## Try it

> Build a 1080×1920 RENDER lower-third in After Effects with the name and title as fields, send it to Aulë, and
> render three proofs.

> In Photoshop, make a 1080×1350 artboard sale post with headline, price and product image layers, send it to Aulë,
> and render proofs for three products.

## How it works

```
agent ──MCP──▶ Aulë (desktop) ──local only──▶ Aulë panel in your Adobe app ──▶ your script in what you have open
                  │
                  └──▶ upload ▶ Forge ▶ template ▶ cloud renders
```

The panel talks only to the Aulë app on your computer (`127.0.0.1`, with a per-launch token). Nothing leaves your
computer until you or your agent send the project to Aulë, which uploads it to your Continuum brand.

## What's here

| Path | What |
| --- | --- |
| `skills/aule-adobe/` | The skill: connect, edit (per-app references), make Forge-ready, send and render |
| `mcp.json`, `.mcp.json` | Aulë's local MCP server, for Codex/ChatGPT and Claude Code |
| `plugin.json`, `.claude-plugin/`, `.agents/plugins/` | Plugin and marketplace manifests |

## License

MIT
