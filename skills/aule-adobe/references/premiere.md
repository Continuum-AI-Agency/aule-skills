# Premiere Pro: UXP JavaScript for `adobe_run`

`adobe_run {app: "premiere", script, label}` runs your script as the body of an **async** function. In scope:
`premierepro` (the UXP module), `uxp`, `project` (the open project, or null) and `edit(build)`.

Premiere changes things through **actions**: read with `await` first, then call `edit` **once** with a synchronous
`build(project)` that creates and returns the actions. Aulë applies them as one transaction — **one undo step** named by
your label.

```js
// adobe_run {"app": "premiere", "label": "rename sequence", "script": "..."}
if (!project) throw new Error("open a project first");
const seq = await project.getActiveSequence();
if (!seq) throw new Error("no active sequence");
const [size, item] = await Promise.all([seq.getFrameSize(), seq.getProjectItem()]);
// Forge names the deliverable `RENDER <sequence>` itself: name the sequence for its size.
edit(() => [item.createSetNameAction("Promo 9:16")]);
return { sequence: "Promo 9:16", width: size.width, height: size.height };
```

- Modern JavaScript. **Methods are async — `await` them**; properties are synchronous.
- Reading: `await project.getSequences()`, `await project.getActiveSequence()`, `await seq.getFrameSize()`,
  `await seq.getEndTime()` (`.seconds`), `await seq.getVideoTrack(i)` and its track items, `await seq.getProjectItem()`,
  and a clip's file: `const clip = premierepro.ClipProjectItem.cast(item); if (clip && !(await clip.isSequence())) path = await clip.getMediaFilePath();`.
- Actions (the `create…Action` methods on sequences, clips, components and parameters) must be created inside
  `build`; `edit` throws if Premiere does not apply them.
- Return plain data, not Premiere objects.
- Check a member before you rely on it: the
  [Premiere Pro UXP reference](https://developer.adobe.com/premiere-pro/uxp/ppro-reference/) and its
  [TypeScript declarations](https://github.com/adobe/premierepro-types) list each class and the version it needs.

Don't: save, close or open projects (`adobe_send` saves a copy); export with the encoder (renders happen in the
cloud).
