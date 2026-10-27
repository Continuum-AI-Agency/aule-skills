---
name: aule-adobe
description: Drive the person's own Adobe apps through Aulë — After Effects, Photoshop, Illustrator and Premiere Pro — to build and edit designs and motion with scripts, make them Forge-ready, send them to Aulë, and render their variants at scale in the Continuum cloud. Use when someone asks to make or change an After Effects comp, a Photoshop or Illustrator design, or a Premiere sequence, or to turn one into a template and render versions of it.
---

# Aulë for Adobe

Aulë is Continuum's desktop app. Its panels in After Effects, Photoshop, Illustrator and Premiere Pro let you run
scripts in what the person has open, one undo step per call. When the work is ready, Aulë sends it in, Forge turns it
into a template, and the Continuum cloud renders as many variants as the person needs. The Adobe apps only draw while
you author; every delivered file is rendered in the cloud (Photoshop, Illustrator and Premiere work is rebuilt as an
After Effects template there).

Every tool takes `app`: `"aftereffects"`, `"photoshop"`, `"illustrator"` or `"premiere"`.

## 1. Connect

1. Find the Aulë tools: `adobe_status`, `adobe_run`, `adobe_frame`, `adobe_send` (MCP server `aule`; names may carry
   a prefix such as `mcp__aule__`). If they are missing:
   - Aulë must be installed (`/Applications/Aule.app`); if not, ask the person to install it.
   - Add the MCP server: `claude mcp add aule -- /Applications/Aule.app/Contents/MacOS/aule mcp`, or
     `codex mcp add aule -- /Applications/Aule.app/Contents/MacOS/aule mcp`, then restart the session.
   - Or call it over the shell (it starts Aulë if needed):
     `echo '{"app":"aftereffects","script":"return app.version"}' | /Applications/Aule.app/Contents/MacOS/aule call adobe_run -`.
     Every tool below is `aule call <name> '<json>'`; `aule commands` lists them all.
2. `auth_status`. Nobody signed in to Continuum: `auth_sign_in` opens their browser; the person finishes it.
3. `adobe_status` lists which apps have the panel connected, and the selected `brandId`.
   - The app you need is not connected: `adobe_install` once, then ask the person to restart the app and open the
     panel: After Effects and Illustrator **Window → Extensions → Aulë**; Photoshop **Plugins → Aulë**; Premiere
     **Window → UXP Plugins → Aulë**. Aulë must be running. (The panels are unsigned for now: say so. After Effects and
     Illustrator need Adobe's CEP debug mode, which `adobe_install` turns on; Photoshop and Premiere may need the UXP
     Developer Tool if Creative Cloud refuses the unsigned plugin — `adobe_install` says which.)
   - `brandId: null`: `workspaces_list`, ask which workspace, `workspaces_select`.
   - `adobe_status {app}` reads what is open in that app. Nothing open, or never saved: ask the person to open or save
     it; `adobe_send` needs a saved file.

## 2. Edit

- Read before you write: `adobe_status {app}`, then scripts that return what you need.
- `adobe_run {app, script, label}`: one small edit per call with a human label (`"add title text"`). The person sees the
  label in Undo and can undo any of your steps. `return` plain values; a thrown error comes back with its message.
  The script language differs per app — read the reference first:
  - After Effects: ES3 ExtendScript function body — [references/after-effects.md](references/after-effects.md)
  - Illustrator: ES3 ExtendScript function body — [references/illustrator.md](references/illustrator.md)
  - Photoshop: modern async JavaScript with the `photoshop` UXP module — [references/photoshop.md](references/photoshop.md)
  - Premiere Pro: modern async JavaScript with the `premierepro` UXP module and `edit(...)` —
    [references/premiere.md](references/premiere.md)
- Never open, close, save or "save as" files yourself, and don't delete or rename what the person made unless they
  asked. `adobe_send` does the saving.
- Look at your work: `adobe_frame {app, …}` saves a PNG and answers its `path`; read the image. After Effects
  `{comp?, time?}`; Photoshop the document; Illustrator the active artboard. Premiere has no frame yet: ask the person
  to look, or use `preview_frame` once the template is in Aulë.

## 3. Make it Forge-ready

Build with Forge's rules in mind from the start ([references/forge-ready.md](references/forge-ready.md)):

- After Effects: deliverable comps named `RENDER <name>`; everything a variant changes is an **Essential Property**.
- Photoshop / Illustrator: one artboard per deliverable size; name every layer a person would change (text and
  images become fields by layer name).
- Premiere: top-level sequences are the deliverables; graphic text and clips become fields; effects, transitions and
  colour grading are not carried.
- Everywhere: installed fonts (not Adobe Fonts), footage and images that exist, no line breaks in text values.

## 4. Send to Aulë, forge, render

Full steps and what each answer means: [references/scale.md](references/scale.md).

1. `adobe_send {app}` saves a copy with its media, uploads it with every font it uses, and queues the Forge. Tell the
   person its `message`, and anything in `skipped` (media) and `fonts.left`.
   - `outcome: "named"`: a template already has this name. Ask whether this is its next revision or a new template,
     then `adobe_send {app, sameName: "revision" | "new"}`. A file sent from this computer before always becomes its
     template's next revision.
   - `state: "uploading" | "parsing"`: it stopped waiting; call what `next` says.
   - `state: "not_forged"`: `forge.refused` says why (missing fonts or media, a published template).
2. `library_wait {brandId, assetIds: [assetId], until: "review"}`. Answer a run's questions with `library_run` /
   `library_forge {action: "decisions"}`.
3. Publishing is the person's decision: they promote it in Aulë, or tell you to (`library_forge {action: "promote"}`).
4. Proofs first: `render_templates` (find the `templateKey` whose `sourceAssetId` is the asset) → `render_inputs` →
   `render_batch_preflight {templateKey, rows}` with 2–3 rows → `render_batch_submit {confirmationToken}` →
   `render_wait {batchId}`. Show the outputs; run the full batch (≤ 50 rows per batch) only when they agree.

Renders cost money. Every slow step has a `*_wait` tool: call it instead of polling.
