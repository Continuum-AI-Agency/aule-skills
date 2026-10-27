# Forge-ready work

**After Effects.** Forge reads the exported project, decides which comps are deliverables and which properties are fields, and
checks the project's hygiene. A project built this way forges without questions.

## Deliverables

- Name every deliverable comp `RENDER <name>`. When any comp is named `RENDER …`, only those are deliverables.
  Without one, Forge guesses (comps with five or more layers, or with Essential Properties that no other comp uses)
  and may ask.
- Delivery comp names are unique.
- Sizes of one design share a name with a format suffix and become one template with variants:
  `RENDER Promo 9:16`, `RENDER Promo 1:1`, `RENDER Promo 16:9` (`story`, `square` and `feed` work too).
- Precomps are fine: build inside precomps, expose fields on the RENDER comp.

## Fields

- A field is an **Essential Property** of the RENDER comp (Essential Graphics panel, or
  `prop.addToMotionGraphicsTemplateAs(comp, "Name")`). Only Essential Properties become fields.
- Name fields for the person who fills them in: `Name`, `Title`, `Price`, `Accent Color`, `Product Image`.
- At most 40 fields per comp; more is flagged.
- An Essential Properties group whose name contains `ignore` is skipped.
- Expose source text, colours, sliders and checkboxes, and replaceable footage (Media Replacement) for images.

## Hygiene Forge checks

| Check | Level | Do |
| --- | --- | --- |
| Solo switch left on | error | turn every solo off |
| A field's layer disabled (eye off) | error | keep field layers enabled; animate opacity instead |
| Text overflows its box | error | size boxes for the longest value; use box text for long values |
| Missing font | warning | use installed fonts, not Adobe Fonts: `adobe_send` imports every other font the project uses |
| Missing footage | warning | import from files that exist; check `skipped` after `adobe_send` |
| Non-ASCII footage file name | warning | name files with plain ASCII |

Also:

- Values sent at render time must not contain line breaks: a newline in a text value breaks the render. Use
  separate fields for separate lines.
- Built-in effects and expressions render on the cloud's After Effects. Ask before using a third-party plug-in: the
  render fleet may not have it.
- Keep the project small: the exported folder must stay under 250 MB to travel with its footage.

## Photoshop and Illustrator

Forge rebuilds a design as an After Effects template:

- Each **artboard** becomes one delivery comp; sizes of one design share fields by layer name. No artboards: the
  canvas is the deliverable.
- **Text layers** become text fields and **smart objects** (Photoshop) or placed images become image fields, by layer
  name: name them for the person who fills them in (`Headline`, `Product Image`). Other layers keep a Show switch.
- Keep text live (not rasterised) in installed fonts; a flattened PSD is refused. Save Illustrator files with **PDF
  compatibility** (`adobe_send` does) — without it Forge cannot read them.
- Not carried (named in the import's `unmapped`): layer styles' exotic effects, adjustment layers, a group's blend
  mode, text cut by a group mask or clipping base.

## Premiere Pro

Forge rebuilds the project as an After Effects template:

- **Top-level sequences** are the deliverables (`RENDER <sequence>`); nested sequences become precomps. Name them for
  the size they deliver (`Promo 9:16`).
- **Graphic text** (Essential Graphics text layers) becomes text fields, and **clips** become media fields.
- Not carried (named in `unmapped`): effects, transitions, Lumetri colour, Motion Graphics template instances,
  captions, keyframed text, generators, Dynamic Link clips, reversed clips. Default Motion only for now (a clip's
  position and scale stay centred at 100%), and text keeps its font but not yet its size and colour.
- Media must be on this computer: `adobe_send` copies every clip's file into the package.
