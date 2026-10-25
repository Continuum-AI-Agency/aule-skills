---
name: aule-after-effects
description: Drive the person's own Adobe After Effects through Aulë — build and edit comps with ExtendScript, make them Forge-ready, send the project to Aulë, and render its variants at scale in the Continuum cloud. Use when someone asks to make or change an After Effects project, motion graphics, lower thirds, titles or ad templates, or to turn an After Effects comp into a template and render versions of it.
---

# Aulë for After Effects

Aulë is Continuum's desktop app. Its After Effects panel lets you run ExtendScript in the project the person has
open, one undo step per call. When the comp is ready, Aulë exports the project, Forge turns it into a template, and
the Continuum cloud renders as many variants as the person needs. After Effects only composites while you author;
every delivered file is rendered in the cloud.

## 1. Connect

1. Find the Aulë tools: `ae_status`, `ae_run`, `ae_frame`, `ae_send` (MCP server `aule`; names may carry a prefix such as
   `mcp__aule__`). If they are missing:
   - Aulë must be installed (`/Applications/Aule.app`); if not, ask the person to install it.
   - Add the MCP server: `claude mcp add aule -- /Applications/Aule.app/Contents/MacOS/aule mcp`, or
     `codex mcp add aule -- /Applications/Aule.app/Contents/MacOS/aule mcp`, then restart the session.
   - Or call it over the shell (it starts Aulë if needed):
     `echo '{"script":"return app.version"}' | /Applications/Aule.app/Contents/MacOS/aule call ae_run -`.
     Every tool below is `aule call <name> '<json>'`; `aule commands` lists them all.
2. `auth_status`. Nobody signed in to Continuum: `auth_sign_in` opens their browser; the person finishes it.
3. `ae_status`.
   - `panel: "absent"`: call `ae_install` once, ask the person to restart After Effects and open
     **Window → Extensions → Aulë**, then `ae_status` again. (The panel is unsigned for now, so `ae_install` turns
     on Adobe's CEP debug mode; say so.)
   - `brandId: null`: `workspaces_list`, ask which workspace, `workspaces_select`.
   - No project open, or `projectPath: null`: ask the person to open or save one. `ae_send` needs a saved project.

## 2. Edit

- Read before you write: `ae_status` lists comps (id, size, fps, duration, layers, `fields` = Essential
  Properties). Read details with a script that returns them.
- `ae_run {script, label}`: the script is an **ES3 function body**; `return` plain values (strings, numbers,
  booleans, arrays and objects of those). A thrown error comes back with its line. See
  [references/extendscript.md](references/extendscript.md) for ES3 rules and recipes.
- One small edit per call, with a human label (`"add title text"`, `"animate title in"`). The person sees the label
  in Edit → Undo and can undo any of your steps.
- Return what you changed (comp and layer names, values) and check it with a read when it matters.
- Never open, close, save or "save as" projects yourself, and don't delete or rename what the person made unless
  they asked. `ae_send` does the saving.
- Look at your work: `ae_frame {comp?, time?}` saves one frame as a PNG and answers its `path`; read the image.
  Check layout, overflow and contrast at a few times before you send. After the project is in Aulë,
  `preview_frame` shows the cloud's exact frame.

## 3. Make it Forge-ready

Build with Forge's rules in mind from the first comp ([references/forge-ready.md](references/forge-ready.md)):

- Deliverable comps are named `RENDER <name>`; sizes of one design share the name with a format suffix
  (`RENDER Promo 9:16`, `RENDER Promo 1:1`).
- Everything a variant changes (texts, colours, images) is an **Essential Property** of the RENDER comp, with a
  readable name, at most 40 per comp.
- No solo switches left on, fonts installed, footage present with ASCII file names, text boxes that fit the longest
  value.

## 4. Send to Aulë, forge, render

Full steps and what each answer means: [references/scale.md](references/scale.md).

1. `ae_send` exports the project with its footage, uploads it with every font it uses, and queues the Forge.
   Tell the person its `message`, and anything in `skipped` (footage) and `fonts.left`.
   - `outcome: "named"`: a template already has this project's name. Ask whether this is its next revision or a
     new template, then `ae_send {sameName: "revision" | "new"}`. A project sent from this computer before always
     becomes its template's next revision.
   - `state: "uploading" | "parsing"`: it stopped waiting; call what `next` says.
   - `state: "not_forged"`: `forge.refused` says why (missing fonts or footage, a published template).
2. `library_wait {brandId, assetIds: [assetId], until: "review"}`. Answer a run's questions with `library_run` /
   `library_forge {action: "decisions"}`.
3. Publishing is the person's decision: they promote it in Aulë, or tell you to (`library_forge {action: "promote"}`).
4. Proofs first: `render_templates` (find the `templateKey` whose `sourceAssetId` is the asset) → `render_inputs` →
   `render_batch_preflight {templateKey, rows}` with 2–3 rows → `render_batch_submit {confirmationToken}` →
   `render_wait {batchId}`. Show the outputs; run the full batch (≤ 50 rows per batch) only when they agree.

Renders cost money. Every slow step has a `*_wait` tool: call it instead of polling.
