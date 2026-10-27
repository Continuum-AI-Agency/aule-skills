# Photoshop: UXP JavaScript for `adobe_run`

`adobe_run {app: "photoshop", script, label}` runs your script as the body of an **async** function, already inside
`executeAsModal` (Photoshop's lock for changing documents). With a label, everything it does is **one history step**,
and a script that throws leaves the document as it was. In scope: `photoshop` (the UXP module), `uxp`.

```js
// adobe_run {"app": "photoshop", "label": "rename title", "script": "..."}
const { app } = photoshop;
const doc = app.activeDocument;
if (!doc) throw new Error("no document open");
let title = null;
for (const layer of doc.layers) if (layer.name === "Title") title = layer; // collections iterate; use for…of
if (!title) throw new Error("no layer Title");
title.name = "Headline";
return { document: doc.name, layers: doc.layers.length };
```

- Modern JavaScript: `const`, arrow functions, `await`. **Properties are synchronous; methods are async** — `await`
  every method call (`await layer.scale(80, 80)`).
- Return plain data (strings, numbers, arrays and objects of those), not layers or documents.
- `doc.layers` is the top level; a group's children are its `layers`; `layer.kind` says what it is (`"text"`,
  `"group"`, `"smartObject"`, `"pixel"`…); `doc.activeLayers` is the selection.
- Text: a text layer's `layer.textItem` holds `contents` and its character style (font, size, colour).
- Changing things: `layer.opacity = 80`, `await layer.scale(x%, y%)`, `await layer.translate(dx, dy)`,
  `await doc.createLayer({ name })`, `layer.bringToFront()`.
- Anything the DOM lacks: `await photoshop.action.batchPlay([descriptor], {})` with a plain action descriptor.
- Check a member before you rely on it: the
  [Photoshop UXP reference](https://developer.adobe.com/photoshop/uxp/2022/ps_reference/) lists each class and the
  Photoshop version it needs.

Don't: save, close or open documents (`adobe_send` saves a copy); show dialogs; call `executeAsModal` yourself.
