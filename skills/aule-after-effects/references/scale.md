# From After Effects to renders at scale

Every slow step has a blocking `*_wait` tool that returns one line per item. Call it instead of polling; when it
answers `done: false` (its timeout), call it again.

## 1. Send

`ae_send` saves the open project, saves a copy with its footage collected into `footage/` beside it, and reopens
the original (its undo history starts over, so finish edits first). It uploads the copy as one ZIP, waits for
Aulë to read it, and queues the Forge. Every font the project uses goes up with it (all but Adobe Fonts). It
answers:

```json
{
  "outcome": "sent",
  "message": "Promo is in the Forge. Aulë shows it when it is ready for review.",
  "assetId": "...",
  "revision": true,
  "state": "forge_queued",
  "skipped": [{ "name": "bg.psd", "reason": "layered file, left linked to ..." }],
  "fonts": { "sent": ["Montserrat-Bold"], "left": [{ "name": "MinionPro-Regular", "reason": "it is from Adobe Fonts, ..." }] },
  "next": "library_wait {brandId: \"...\", assetIds: [\"...\"], until: \"review\"}"
}
```

- `skipped` footage stays linked to its original path and will be missing in the cloud: missing files, image
  sequences, and layered PSD/AI imports. Tell the person; flatten or replace it if it matters.
- `fonts.sent`: faces imported into the brand. `fonts.left`: faces that stayed behind, with why. Adobe Fonts'
  licence keeps them off servers, and a font not installed on this computer can't be sent: the person adds a licensed
  `.ttf`/`.otf` in Aulë, or accepts a substitute (`library_fonts` lists candidates,
  `library_substitute` picks one).
- `outcome: "named"`: a template already has this name and this project wasn't sent from this computer. Ask:
  `ae_send {sameName: "revision"}` (its next revision) or `"new"` (a separate template).
- `outcome: "same"`: these exact bytes are already the template `assetId`.
- `state`:
  - `"uploading"` or `"parsing"`: it stopped waiting (`timeoutSec`, default 600). Call what `next` says.
  - `"not_forged"`: `forge.refused[0].reason` says why. Fix it, then `library_forge_all {brandId, items: [{assetId}]}`.
  - `"forge_queued"`: go on below.
- `ae_send {forge: false}` stops once it is in the Library.

## 2. Forge

1. `library_wait {brandId, assetIds: [assetId], until: "review"}`. A run that stops with `needs_input` has
   questions: `library_run {assetId}` shows them; answer with `library_forge {action: "decisions", ...}`.
2. `library_get {assetId}` shows the parse: comps, fields, fonts and missing footage.
3. Publishing (`library_forge {action: "promote"}`) is the person's decision. Ask; don't promote on your own.

## 3. Uploading another way

`ae_export` alone answers `{aepPath, projectPath, collected, skipped, fonts}` without uploading. Then
`upload_files {brandId, paths: [aepPath], pack: true, linkTo: projectPath}` (`revisionOf: <assetId>` to make it a
revision) → `upload_wait {uploadIds}` → `library_wait {until: "parsed"}` → `library_forge_all`. For a revision,
`upload_wait` answers `revisionOf`: wait on that template, not the upload's `assetId`.

## 4. Proof, then batch

1. `render_templates {brandId}` → the `templateKey` whose `sourceAssetId` is your asset.
2. `render_inputs {templateKey}` → every field with its current value, and `problems` that block a render.
3. `render_batch_preflight {templateKey, rows: [{label, variables: {<field>: <value>}}]}` with 2–3 proof rows.
   Rows come back `READY`, `INCOMPLETE` or `BLOCKED` with findings; fix and preflight again.
4. `render_batch_submit {confirmationToken}` → `render_wait {batchId}` → each job's `outputs` (URLs) and the engine
   that rendered it.
5. Show the proofs. Only when the person agrees, run the full set: up to 50 rows per batch. A CSV or TSV of rows:
   `render_rows_read {path}`.

Delivery settings (frame rate, codec, audio) live on the template: `specs_get` / `specs_set`.

Renders cost money and take seconds to minutes each. Never loop renders to "try again" without the person.
