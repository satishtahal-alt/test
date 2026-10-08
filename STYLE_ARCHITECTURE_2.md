# Style system, pass 2: mechanisms, parameters, recipes (2026-10-08)

The second pass of the Style system work (the plan: audit → review → foundation → new looks → Style Lab → generator).
Baseline: `docs/STYLE_AUDIT_1.md`. This pass makes the current system **describe itself** without changing what it
draws. It does not add a look, a mechanism the engine did not have, a Style Lab, or free mixing of mechanisms.

Code: `js/styleMechanisms.js` (new, the registry), `js/styles.js` (the looks as recipes, everything derived),
`js/watercolor.js` (two binding names), `js/stylePanel.js` (tooltips). Tests: `test/unit/style_mech.test.js`,
`test/support/styleSamples.js`, `test/fixtures/style_baseline.json`, `test/probes/style_equiv.js`,
`test/probes/style_arch2.js`.

---

## 0. The audit against the code

Every count of the audit was measured again by script on the code of commit `0125e23`.

| Claim of the audit | Measured | State |
|---|---|---|
| 13 kinds in code | 13 | holds |
| 12 built-in styles, `flat` has none | 12, `flat` | holds |
| 632 catalog entries, 9 aliases, 623 variants | 632, 9, 623 | holds |
| 420 variants differ only by palette, 203 change a setting their kind reads | 420 and 203 by that rule | holds, and is refined below |
| 105 of 396 written settings are ignored by the entry's kind, in 99 entries | 105 of 396, in 99 | holds |
| 11 distinct setting values, 29 hybrids | 11, 29 | holds |
| 20 settings (16 sliders, 3 overlay, `washRecipe`), 25 roles, 42 materials + auto | the same | holds |
| 13 geometry modifiers with 106 declared parameters | 13, 106 | holds |
| Line & Wash fails validation on its own export | `settings.washRecipe: unknown setting` | holds (fixed in this pass) |
| an unknown setting is rejected on import | rejected | holds |

Three corrections. None changes a conclusion.

1. **423 colourways, not 420.** Three of the 203 "variants" write a setting their kind reads, but with the value the
   built-in already has. They change nothing. The numbers used from here on: 9 aliases, **423** palette-only, **200**
   that change how the look draws.
2. **Blueprint draws no dashed guides.** The audit lists "dashed guides" for the blueprint kind. The line is in the
   code, but an object whose material is a guide leaves `resolve` before any kind runs. The dash is never drawn. It is
   left as it is and is not declared as a mechanism of the blueprint look.
3. **`flat` never read `depthFade`.** Its hand-written list of settings named it. No style can have the flat look (an
   imported definition of kind `flat` is stored on the first built-in), so nothing visible depended on it.

Nothing in the audit was stale because of the workflow commits between it and this pass: they did not touch the
Style system.

---

## 1. The three decisions

### A. What the 632 catalog entries are

**Recommendation: keep every entry, stop calling all of them styles, and present three levels.**

| Level | What it is | Today |
|---|---|---|
| **Look** | a recipe of mechanisms: what makes a rendering technique | 13 (12 with a built-in style) |
| **Variant** | a look with different parameter values: it draws differently | the 12 built-ins, 200 catalog entries, a scene's own variants |
| **Colourway** | a palette on a look or a variant: the same drawing in other colours | 423 catalog entries |

And two **tags** on an entry, which are not levels:

| Tag | Meaning | Today |
|---|---|---|
| **other name** | another name for a built-in (a search word, not a second style) | 9 |
| **intended as** | the source asked for something its look cannot do yet: a hybrid of two looks (29), or settings its look ignores (99 entries, 105 settings) | 124 entries carry one or both |

How this should appear, when the Style Manager is built:

* The first list is **looks** (13 rows), not 632 rows. A look opens to its variants. A variant opens to its colourways.
* A colourway is offered as what it is: *"Paper Diorama in these colours"*. Its 25 role colours come from the
  entry's seed colours, which do not depend on the look. So a colourway can be applied to **any** look or variant.
  That makes the 423 more useful than they are now, not less: 423 palettes times any look.
* The entries tagged *intended as* are the **rebuild list**. When a mechanism arrives that their name asks for (a
  stitch, a frame pass, two looks at once), they are re-derived and move from colourway to variant. Until then they
  say plainly that they are a colourway of look X.
* Provenance ("Game Design Manual, category, source") stays a field of the entry. It is not a class.

What exists now, without touching the catalog: `Styles.Catalog.classify(entry)` and `Styles.Catalog.census()` work
this out from each entry's data and its look's recipe. The table below is their output.

| Look | Other names | Colourways | Variants | Entries |
|---|---|---|---|---|
| paper | 0 | 70 | 62 | 132 |
| woodblock | 2 | 21 | 12 | 35 |
| neon | 1 | 70 | 36 | 107 |
| blueprint | 1 | 59 | 5 | 65 |
| silhouette | 0 | 43 | 17 | 60 |
| biolume | 0 | 37 | 6 | 43 |
| pixel | 1 | 18 | 8 | 27 |
| comic | 1 | 21 | 15 | 37 |
| chalk | 1 | 13 | 7 | 21 |
| sketch | 1 | 8 | 4 | 13 |
| wash | 0 | 45 | 20 | 65 |
| impasto | 1 | 18 | 8 | 27 |
| **all** | **9** | **423** | **200** | **632** |

Tagged *intended as*: 29 hybrids, 99 entries with 105 ignored settings, 124 entries in all.

Not done, on purpose: no entry was moved, renamed, deleted or re-derived, and the panel still lists the catalog as
it did. A class is computed, not stored, so it stays right when a look gains a parameter.

### B. `texture` and `jitter`: split, into three things each

What each slider did, by look, read from the code:

| Slider | What it drove | In the looks |
|---|---|---|
| `texture` | the **scene surface**: the tile and the vignette over the whole picture | all 13 |
| `texture` | a **tile inside shapes**: halftone dots, chalk dust, pigment grain | comic, chalk, wash |
| `texture` | **shading marks** drawn as strokes: hatching, brush dabs | sketch, impasto |
| `jitter` | **corner points** of polygons moved | paper, chalk |
| `jitter` | the **edge of the fill** roughened (and, in the wash, moved off the line) | impasto, wash |
| `jitter` | **strokes** that overshoot, wobble and drift | sketch, wash (its pen) |

**Decision: yes, split. Six concepts, each the dial of one mechanism. No look loses its slider.**

| New parameter | Mechanism | Follows | Used by |
|---|---|---|---|
| `texture` (unchanged) | Scene surface | | all |
| `shapeTexture.strength` | Texture inside shapes | `texture` | comic, chalk, wash |
| `hatchShading.strength` | Hatching | `texture` | sketch |
| `brushDabs.strength` | Brush dabs | `texture` | impasto |
| `jitter` (unchanged) | Corner jitter | | paper, chalk |
| `edgeRoughness.amount` | Rough edge | `jitter` | impasto, wash |
| `looseStroke.amount` | Loose strokes | `jitter` | sketch, wash |

How it keeps every style where it is: a parameter may **follow** another. While it is not set, it has the value of the
one it follows. No built-in, no catalog entry and no saved scene sets the new ones, so each reads `texture` or
`jitter` exactly as before. A style that sets one moves that mechanism alone. Hatching and brush dabs are two
mechanisms and so have two dials; both are in the category *Marks*, which is the concept a person sees.

What was deliberately not split: the scale and blend of a tile, the wash's offset against its roughness, the three
numbers of a loose stroke (overshoot, wobble, drift). Each stays behind its one dial.

Implemented in this pass (it is the proof that the split costs nothing): the four mechanisms exist, the looks read
them, and a definition may set them. The Styles panel does not show them yet; the old two sliders are still the
controls a person has.

### C. How the registry works

```
Mechanism   id, name, stage (where it acts), slot (what it excludes), description, its parameters
Parameter   id, label, description, type (number | choice | color), default, range and step,
            owner = its mechanism;  control (the mechanism's own dial);  follows (another parameter)
Look        a recipe: [ mechanism + the values this look gives its parameters ]
Style       a look + a palette + materials + settings (any parameter, by id)

value of a parameter in a style = the style's setting, else the look's recipe value,
                                  else the parameter it follows, else its default
```

* **One list.** `Draw5.StyleMech` is the only place a mechanism or a parameter is declared. The panel's sliders
  (`Styles.SETTINGS`), the defaults (`DEFAULT_SETTINGS`), what a definition may set (`validateDefinition`), the
  sliders a look shows (`Kinds[k].uses`), its overlay and its low-resolution flag are all **derived**.
* **Stable ids.** A parameter that had a setting keeps its plain id (`lineWeight`, `texture`): scenes, variants, the
  catalog and `.d5style.json` files name them. A parameter lifted out of a look is `<mechanism>.<key>`
  (`dropShadow.blur`).
* **Usage is derived, and checked.** A look uses the dials of its mechanisms, the parameters its recipe gives a
  value, and what those follow. A test runs every look over 162 sample shapes with a recording reader and fails if
  the code reads a parameter the recipe does not name, or the recipe names one the code never reads.
* **Stages** say where a mechanism acts: colour, fill, edge, stroke, geometry, marks, shape surface, shadow and
  glow, scene surface, frame.
* **Slots** say what cannot coexist. There is one today: `halo`, the single blurred copy the renderer has per shape.
  Drop shadow and Glow both take it. `StyleMech.conflict('dropShadow', 'glow')` says so, and a recipe with both is
  refused. Giving the renderer a second blurred copy would remove the slot; that is renderer work for a later pass.
* **Different from the geometry modifier registry** in three ways. A mechanism has no `apply` function yet: the
  drawing arithmetic is still in each look (the adaptor). A parameter id is global, not local to its mechanism,
  because style files name it. And a recipe is checked when the look is defined, not when it runs.

What is still code: each look's `adjust` function. It is the **adaptor**. It reads every number it takes from the
recipe through `c.p(id)`, so the number is visible, can be set by a style and is counted in the usage. The step
after this one is to move the arithmetic itself into the mechanisms; only then can a style add or remove one.

### D. `washRecipe`

It is a **choice parameter** of the mechanism *Watercolour wash*: which data recipe (`watercolor.js`) builds each
shape. Its options are the recipes that are registered (`lineAndWash`, `wetInWet`, `boldInk`), read from
`Draw5.Watercolor.RECIPES` when asked, so a new recipe is a new option with no second list. Not set means the
look's own (`lineAndWash`).

The defect was that the validator had its own list of settings and `washRecipe` was not in it. The validator has no
list now: a setting is valid when a registered parameter has that id, and its value is checked by the parameter's
own type, range or options. `washRecipe` is valid because the mechanism declares it. The three hand-written special
cases for `overlay`, `overlayBlend` and `overlayScale` went the same way.

---

## 2. What was built

### Mechanisms (22) and parameters (84)

| Mechanism | Id | Acts on | Slot | Parameters | Looks that use it |
|---|---|---|---|---|---|
| Line weight | `line` | Stroke |  | `lineWeight` | flat, paper, woodblock, neon, blueprint, biolume, pixel, comic, chalk, sketch, wash, impasto |
| Outline | `outline` | Edge |  | `outline`, `width`, `darken`, `floor`, `nearGain` | flat, paper, woodblock, blueprint, silhouette, pixel, comic, chalk, sketch, wash, impasto |
| Drop shadow | `dropShadow` | Shadow and glow | halo | `shadow`, `color`, `opacity`, `blur`, `blurNear`, `offsetX`, `offsetXNear`, `offsetY`, `offsetYNear` | paper, comic, impasto |
| Glow | `glow` | Shadow and glow | halo | `glow`, `blur`, `heroBlur`, `textBlur`, `minDepth` | neon, silhouette, biolume |
| Scene surface | `sceneOverlay` | Scene surface |  | `texture`, `overlay`, `overlayBlend`, `overlayScale`, `opacity`, `vignette` | all |
| Depth fade | `depth` | Colour |  | `depthFade`, `reach`, `curve`, `cut` | paper, woodblock, neon, silhouette, biolume, pixel, comic, chalk, sketch, wash, impasto |
| Corner jitter | `cornerJitter` | Geometry |  | `jitter`, `reach` | paper, chalk |
| Colour grade | `colourGrade` | Colour |  | `saturation`, `brightness`, `contrast` | all |
| Low resolution | `lowRes` | Frame |  | `pixelSize` | pixel |
| Watercolour wash | `watercolour` | Marks |  | `wetness`, `bloom`, `pooling`, `coverage`, `penVariation`, `washRecipe`, `pale` | wash |
| Tube line | `rim` | Edge |  | `width` | neon, biolume |
| Fill strength | `fillStrength` | Fill |  | `amount`, `nearGain`, `backdrop` | neon, blueprint, biolume, chalk |
| Translucent bodies | `translucency` | Fill |  | `base`, `nearGain` | biolume |
| Hard bands | `banding` | Fill |  | `minBands`, `perStop` | pixel |
| Two cel tones | `celTones` | Fill |  | `split` | comic |
| Broken line | `dashes` | Edge |  | `base`, `perWeight`, `openPerWeight`, `gap` | chalk |
| Rough edge | `edgeRoughness` | Geometry |  | `amount`, `base`, `gain` | wash, impasto |
| Loose strokes | `looseStroke` | Stroke |  | `amount`, `length`, `gaps`, `secondLength`, `secondGaps` | sketch, wash |
| Texture inside shapes | `shapeTexture` | Shape surface |  | `strength`, `tile`, `opacity`, `backdropOpacity`, `minArea` | comic, chalk, wash |
| Hatching | `hatchShading` | Marks |  | `strength`, `angle`, `crossAbove`, `spacing`, `spacingRange`, `width` | sketch |
| Brush dabs | `brushDabs` | Marks |  | `strength`, `angle`, `angleRandom`, `widthLight`, `widthDark` | impasto |
| Lit ridge | `paintRidge` | Marks |  | `lighten`, `width`, `dx`, `dy` | impasto |

20 parameters existed as settings. 64 are new ids: 6 dials from the split above and 58 numbers and choices that were
fixed inside a look.

### The looks as recipes

Every number below was a constant in the look's code and is now the value its recipe gives a declared parameter. A
style of that look may set any of them in its `settings`.

| Look | Its recipe: mechanism (the values it gives) |
|---|---|
| flat | colourGrade · line · outline (width 1.5, darken 0.35) · sceneOverlay (overlay none) |
| paper | colourGrade · depth (reach 0.25) · line · outline (width 1.3, darken 0.22) · cornerJitter (reach 2.6) · dropShadow (color #2a1a10, opacity 0.28, blur 2, blurNear 5, offsetX 1, offsetXNear 4, offsetY 1.5, offsetYNear 6) · sceneOverlay (overlay paper) |
| woodblock | colourGrade · depth (reach 0.3) · line · outline (width 2.6) · sceneOverlay (overlay fiber) |
| neon | colourGrade · depth (reach 0.55, cut 0.3) · line · rim (width 2) · fillStrength (amount 0.2) · glow (blur 14, heroBlur 24, textBlur 12) · sceneOverlay (overlay scan, vignette rgba(0,0,0,0.55)) |
| blueprint | colourGrade · line · outline (width 1.4, floor 0.3) · fillStrength (amount 0.14) · sceneOverlay (overlay grid) |
| silhouette | colourGrade · depth (reach 0.75, curve 2) · outline (width 0.8) · glow (blur 22) · sceneOverlay (overlay paper, opacity 0.5, vignette rgba(60,10,40,0.5)) |
| biolume | colourGrade · depth (reach 0.6) · line · rim (width 1.5) · fillStrength (amount 0.42) · translucency (base 0.62, nearGain 0.3) · glow (blur 18, heroBlur 30, textBlur 10, minDepth 0.3) · sceneOverlay (overlay motes, vignette rgba(0,8,18,0.7)) |
| pixel | colourGrade · banding (minBands 4, perStop 2) · depth (reach 0.3) · line · outline (width 2) · lowRes · sceneOverlay (overlay pixelgrid, opacity 0.6) |
| comic | colourGrade · celTones (split 0.5) · depth (reach 0.35) · line · outline (width 1.6, nearGain 1.6) · dropShadow (opacity 0.9, blur 0, offsetX 3.5, offsetY 3.5) · shapeTexture (tile halftone, opacity 0.45, backdropOpacity 0.55) · sceneOverlay (overlay grain, opacity 0.7) |
| chalk | colourGrade · depth (reach 0.5) · line · outline (width 2.2, floor 0.3) · fillStrength (amount 0.2, nearGain 0.14, backdrop 0.35) · dashes (base 3, perWeight 10, openPerWeight 8, gap 1.8) · cornerJitter (reach 2.2) · shapeTexture (tile chalk, opacity 0.5, backdropOpacity 0.6, minArea 3000) · sceneOverlay (overlay chalk, opacity 0.85, vignette rgba(0,0,0,0.35)) |
| sketch | colourGrade · depth (reach 0.6) · line · outline (width 1.05) · hatchShading (angle 52, crossAbove 0.55, spacing 2.4, spacingRange 8, width 0.65) · looseStroke (length 70, gaps 0.08, secondLength 45, secondGaps 0.35) · sceneOverlay (overlay paper, opacity 0.55) |
| wash | colourGrade · depth (reach 0.4) · line · outline (floor 0.3) · watercolour (pale 0.1) · edgeRoughness · looseStroke · shapeTexture (tile watercolor, opacity 0.55, backdropOpacity 0.7, minArea 1200) · sceneOverlay (overlay coldpress, opacity 0.9) |
| impasto | colourGrade · depth (reach 0.35) · line · outline (width 2, darken 0.3) · edgeRoughness (base 1.6, gain 2.8) · brushDabs (angle 20, angleRandom 35, widthLight 4.5, widthDark 3.5) · paintRidge (lighten 0.55, width 1.4, dx -1.1, dy -1.3) · dropShadow (color #1a1008, opacity 0.3, blur 1.5, offsetX 1.2, offsetXNear 1.5, offsetY 1.6, offsetYNear 2) · sceneOverlay (overlay canvas, opacity 0.9) |

### Compatibility and adaptors

* A style **stores only what it sets**. A lifted parameter is not written into `DEFAULT_SETTINGS`, so an exported
  built-in holds the same 17 or 18 settings as before and no file grew.
* `Kinds[k].uses`, `.overlay` and `.resolution` are still there for the code that reads them (the panel, the schema),
  now computed from the recipe.
* `watercolor.js`: the `lineAndWash` recipe bound `jitter` in two places. The wash body now binds `roughness`, the pen
  binds `looseness`. A caller that passes only the sliders gets `jitter` for both, so a recipe object kept in an old
  scene and the existing tests build what they built.
* Old scenes: a setting no mechanism owns, or a recipe name that is not known, is kept and ignored as before. Only
  **import** refuses.

### Import and export

* The file format is unchanged: `D5STY`, `settings` is a flat map of parameter id to value.
* Validation is by the registry (see D). Messages keep their form: `settings.wetness: number 0..1`,
  `settings.zzz: unknown setting`. New: `settings.washRecipe: one of lineAndWash, wetInWet, boldInk`.
* The exported vocabulary (`Styles.vocabulary()`, `Styles.schema()`) now lists every parameter with its mechanism,
  description, type, range and default, every mechanism with its stage, slot and the looks that use it, and each
  look's recipe.
* One change a diff will show: the **order of keys** inside `settings` in an exported file follows the registry
  (`overlay`, `overlayBlend`, `overlayScale` come after `texture`, not after `pixelSize`). The values are the same.

### The Styles panel

Unchanged in what it shows. Two additions, both read from the registry: every slider and the three overlay controls
say what they do on hover (*"Scene surface: How strongly the scene surface shows…"*), and the Definition box names
the mechanisms the look is made of.

---

## 3. How to see it (the UI and the console)

1. **Line & Wash survives its own file.** Look ▸ Styles ▸ pick *Line & Wash* ▸ Definition ▸ **Export…**. Then
   **Open…** the downloaded `style_wash.d5style.json`. It is added as a new style and draws the same picture. Before
   this pass it was refused.
2. **What a slider does.** Look ▸ Styles ▸ hover any slider under Settings.
3. **A number that was fixed.** Look ▸ Styles ▸ *Paper Diorama* ▸ Customise ▸ **Save as variant**. In its Definition
   box add `"dropShadow.blurNear": 14, "dropShadow.offsetYNear": 16` to `settings` ▸ **Apply definition**. The paper
   shadows become long and soft. No other style changes.
4. **One of the split dials.** On a variant of *Comic Halftone* add `"shapeTexture.strength": 0.2` ▸ Apply. The dots
   inside shapes fade; the paper grain over the scene stays at the Texture slider.
5. **Who uses what** (the console): `Draw5.Styles.looksUsing('glow')`, `Draw5.Styles.looksWith('dropShadow')`,
   `Draw5.Styles.parameterMechanisms('texture')`, `draw5.styles.recipe('style_diorama')`,
   `draw5.styles.usage()`, `Draw5.Styles.Catalog.census()`.

---

## 4. Evidence

| Check | Result |
|---|---|
| Unit tests (`node test/run_all.js`) | 1286 passed, 0 failed (1266 before) |
| New unit tests (`style_mech.test.js`) | 20 |
| Appearance records, old code against new: 12 built-ins, each 162 shapes at 4 depths, plus overlay, low-resolution request, effect language, 2.5D tones and the exported definition | identical (hash per style) |
| The same for all 632 catalog entries, installed | identical (hash per look) |
| Pictures in the desktop app, old code against new (`style_equiv.js`): 12 built-ins, 4 catalog entries, Line & Wash with `boldInk` | 17 of 17 pixel-identical |
| `style_arch2.js` in the desktop app | 19 of 19 |
| Existing probes that use the Styles panel or the style switcher: `workflow_examples.js`, `panel_search.js`, `p13_0_examples.js` | 43 of 43, all rows ok, 14 of 14 |
| `passRegression` | PASS: dojo, stepyard, watchpost identical to the base `0125e23`, both runs agree |

How the equivalence was made strict: `test/fixtures/style_baseline.json` was written by running
`test/support/styleSamples.js` on the code of `0125e23`, in a separate checkout. The pixel comparison ran the same
probe on the old files and on the new ones, at one stage size, with the clock held while the level was built (object
ids carry the clock, and torn edges and stroke passes are seeded by the id).

Differences found: **none in what is drawn.** Two things that are not the same as before:

* the order of keys in an exported `settings` object (above);
* `resolve` costs about 5 microseconds more per shape (15 to 20 in Node, measured over 10 872 calls): the reader
  that looks a parameter up. For the 1 330 objects of the showcase that is about 7 ms on a full rebuild.

---

## 5. Not done, on purpose

| Not done | Why it waits |
|---|---|
| Full composability: a style that adds, removes or repeats a mechanism | the arithmetic is still in each look's adaptor; a mechanism has no `apply` yet. A look may use a mechanism once |
| Style Manager and Style Lab | the vocabulary is to be reviewed first |
| A UI that edits every parameter | the 64 new parameters can be set in a definition only |
| Creating a style from nothing | needs composability |
| Random and novelty generation | needs the Lab |
| 16-bit and retro looks | need a frame stage: palette restriction, dithering, display effects |
| Embroidery and thread | needs marks placed as shapes along and across a shape |
| New frame-level mechanisms | none was added; `lowRes` is the only frame mechanism and is unchanged |
| Mark and stitch placement | not started |
| Reclassifying or rebuilding the catalog | the classes are computed and shown in section 1A; nothing was moved |
| A second blurred copy per shape (shadow and glow together) | renderer work; the `halo` slot states the limit |
| Lists of materials as parameters (which materials glow in the silhouette look, which get dots in the comic look) | they are still in code; a list type was not added |
| The remaining fixed numbers | lifted where they define a look. Still in code: the line colours of blueprint and chalk, the hatch dash lengths, the dab spacings, open-line widths, text handling, the 2.5D tone steps (`DIM_LOOK`), every value inside the three watercolour recipes (already data, not yet parameters) |
| Relaxed import (keep an unknown parameter and report it) | the audit proposed it; an unknown setting is still refused |

### Seen while testing, not from this pass

The first picture of *Line & Wash* in the showcase level is not the picture every later look at it gives: its
content layer differs once, also against the built-in itself, and the old code does the same (identical hashes).
Recorded in `GRAND_ROADMAP.md` ▸ Recorded gaps.

`test/probes/panel_search.js` has one row that is not ok, *Engines (popped out)*: the probe reads the count of the
popped-out Engines panel and does not find what it expects. The old code gives the same row. The six other rows,
the Styles list and the catalog among them, are ok. Recorded in the same place.

---

## 6. Open for the review

1. Is the three-level model (look, variant, colourway) the one to build the Style Manager on?
2. Are the six dials of the split the right vocabulary, and should the panel show them before the Lab exists?
3. Are the 22 mechanisms cut at the right size? Two candidates to merge or split: *Tube line* against *Outline*
   (two edge mechanisms that differ only in whether the Outline slider scales them), and *Hatching* against *Brush
   dabs* (both are rows of strokes inside a shape).
4. Which family comes first after this: the frame stage (pixel and retro) or marks as shapes (embroidery)?
