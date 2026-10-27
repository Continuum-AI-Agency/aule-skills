# Illustrator: ExtendScript for `adobe_run`

`adobe_run {app: "illustrator", script, label}` runs your script as the body of a function inside Illustrator.
Illustrator has **no undo-group API**: one call may be several undo steps, so keep each call to one small change.

```js
// adobe_run {"app": "illustrator", "label": "rename title", "script": "..."}
var doc = app.activeDocument;
var title = null;
for (var i = 0; i < doc.textFrames.length; i++) {
  if (doc.textFrames[i].name === "Title") title = doc.textFrames[i];
}
if (!title) throw new Error("no text frame named Title");
title.contents = "Spring Sale";
return { artboards: doc.artboards.length, text: title.contents };
```

ES3, as in After Effects ([after-effects.md](after-effects.md)): `var` only, write loops, collections are
**0-based** here (`doc.textFrames[0]`), return plain data.

## Find and change things

```js
var ab = doc.artboards[doc.artboards.getActiveArtboardIndex()];
var r = ab.artboardRect;                       // [left, top, right, bottom]; y grows upward

var t = doc.textFrames.add();
t.name = "Name";
t.contents = "Ana Souza";
t.position = [r[0] + 72, r[1] - 72];           // top-left of the frame
var attrs = t.textRange.characterAttributes;
attrs.size = 48;
attrs.textFont = app.textFonts.getByName("Arial-BoldMT"); // PostScript name; must be installed
var c = new RGBColor(); c.red = 250; c.green = 92; c.blue = 51;
attrs.fillColor = c;

var bar = doc.pathItems.rectangle(r[1] - 200, r[0] + 72, 400, 12); // top, left, width, height
bar.name = "Bar";

var img = doc.placedItems.add();
img.file = new File("/path/to/photo.jpg");
img.name = "Photo";
```

Name every item a variant changes (`Title`, `Photo`): Forge makes fields from text and images by name.

Don't: `doc.save()`, `saveAs`, `close`, `app.open` (`adobe_send` saves a copy); `alert()` or anything that shows a
dialog — it blocks Illustrator and the call times out.
