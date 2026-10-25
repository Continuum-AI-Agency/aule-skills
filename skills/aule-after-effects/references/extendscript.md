# ExtendScript for `ae_run`

`ae_run` runs your script as the body of a function inside After Effects, wrapped in one undo group:

```js
// ae_run {"label": "rename title", "script": "..."}
var comp = findComp("RENDER Promo 9:16");
if (!comp) throw new Error("no comp RENDER Promo 9:16");
comp.layer("Title").name = "Headline";
return { comp: comp.name, layers: comp.numLayers };
```

Each call is separate: define the helpers you need in every script.

## ES3, not modern JavaScript

- `var` only: no `let`, `const`, arrow functions, template literals, classes, destructuring, spread, `for…of`.
- No `Array.prototype.forEach / map / filter / indexOf`, no `Object.keys`, no `String.prototype.trim`. Write loops.
- `JSON.stringify` / `JSON.parse` are available (Aulë adds them when After Effects lacks them).
- Collections are **1-based**: `app.project.item(1)`, `comp.layer(1)`, `prop.keyValue(1)`.
- Colours are `[r, g, b]` (text) or `[r, g, b, a]` (shapes, effects), each 0–1.
- Times are seconds; `comp.frameDuration` is one frame.
- Return plain data. A returned layer or comp comes back as `"[TextLayer]"`, not its contents.

## Find things

```js
function findComp(name) {
  for (var i = 1; i <= app.project.numItems; i++) {
    var it = app.project.item(i);
    if (it instanceof CompItem && it.name === name) return it;
  }
  return null;
}
var layer = comp.layer("Title");            // by name (first match) or index
var active = app.project.activeItem;         // the comp open in the viewer, if any
```

## Make things

```js
var comp = app.project.items.addComp("RENDER Lower Third 9:16", 1080, 1920, 1, 6, 30); // name, w, h, aspect, sec, fps
comp.openInViewer();

var bg = comp.layers.addSolid([0.1, 0.1, 0.12], "Background", comp.width, comp.height, 1);

var title = comp.layers.addText("Ana Souza");
title.name = "Name";
var st = title.property("ADBE Text Properties").property("ADBE Text Document");
var doc = st.value;
doc.font = "Arial-BoldMT";                  // PostScript name; the font must be installed
doc.fontSize = 72;
doc.fillColor = [1, 1, 1];
doc.applyFill = true;
doc.justification = ParagraphJustification.LEFT_JUSTIFY;
st.setValue(doc);
title.property("ADBE Transform Group").property("ADBE Position").setValue([96, 1600]);

// Box text (fixed width, wraps) for values of unknown length:
var body = comp.layers.addBoxText([880, 200], "Head of Design");

var shape = comp.layers.addShape();
shape.name = "Bar";
var group = shape.property("ADBE Root Vectors Group").addProperty("ADBE Vector Group");
var contents = group.property("ADBE Vectors Group");
contents.addProperty("ADBE Vector Shape - Rect").property("ADBE Vector Rect Size").setValue([880, 12]);
contents.addProperty("ADBE Vector Graphic - Fill").property("ADBE Vector Fill Color").setValue([0.98, 0.36, 0.2, 1]);

var footage = app.project.importFile(new ImportOptions(new File("/path/to/logo.png")));
var logo = comp.layers.add(footage);
```

## Animate

```js
var pos = title.property("ADBE Transform Group").property("ADBE Position");
pos.setValueAtTime(0, [-600, 1600]);
pos.setValueAtTime(0.6, [96, 1600]);
var ease = new KeyframeEase(0, 75);
for (var k = 1; k <= pos.numKeys; k++) pos.setTemporalEaseAtKey(k, [ease], [ease]); // one ease for spatial props

var scale = title.property("ADBE Transform Group").property("ADBE Scale");
scale.setValueAtTime(0, [80, 80, 100]);
scale.setValueAtTime(0.6, [100, 100, 100]);
scale.setTemporalEaseAtKey(2, [ease, ease, ease], [ease, ease, ease]); // one ease per dimension otherwise

var opacity = title.property("ADBE Transform Group").property("ADBE Opacity");
opacity.expression = "linear(time, 0, 0.4, 0, 100)";
```

## Expose fields (Essential Properties)

```js
var text = comp.layer("Name").property("ADBE Text Properties").property("ADBE Text Document");
text.addToMotionGraphicsTemplateAs(comp, "Name");
var fill = comp.layer("Bar").property("ADBE Root Vectors Group").property(1)
  .property("ADBE Vectors Group").property("ADBE Vector Graphic - Fill").property("ADBE Vector Fill Color");
fill.addToMotionGraphicsTemplateAs(comp, "Accent Color");
return comp.motionGraphicsTemplateControllerCount;
```

`prop.canAddToMotionGraphicsTemplate(comp)` says whether a property can be exposed.

## Read what is there

```js
var out = [];
for (var i = 1; i <= comp.numLayers; i++) {
  var l = comp.layer(i);
  out.push({ index: i, name: l.name, type: l.reflect.name, enabled: l.enabled, solo: l.solo, in: l.inPoint, out: l.outPoint });
}
return out;
```

## Don't

- `app.project.save()`, `app.open()`, `app.newProject()`, `app.quit()`: `ae_send` saves, the person opens.
- `alert()`, `confirm()`, `prompt()` or anything else that shows a dialog: it blocks After Effects until someone
  clicks, and `ae_run` times out.
- `app.beginUndoGroup()` yourself: each `ae_run` already is one.
- Long loops over every frame: set keyframes or expressions instead.
