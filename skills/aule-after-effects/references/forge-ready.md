# Forge-ready comps

Forge reads the exported project, decides which comps are deliverables and which properties are fields, and
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
| Missing font | warning | use installed fonts, not Adobe Fonts: `ae_send` imports every other font the project uses |
| Missing footage | warning | import from files that exist; check `skipped` after `ae_send` |
| Non-ASCII footage file name | warning | name files with plain ASCII |

Also:

- Values sent at render time must not contain line breaks: a newline in a text value breaks the render. Use
  separate fields for separate lines.
- Built-in effects and expressions render on the cloud's After Effects. Ask before using a third-party plug-in: the
  render fleet may not have it.
- Keep the project small: the exported folder must stay under 250 MB to travel with its footage.
