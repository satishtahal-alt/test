# Style system audit, phase 1: what Draw5 has today (2026-10-08)

Audit only. Nothing in the Style system was changed. Source of truth: `js/styles.js`, `js/renderer.js`,
`js/textures.js`, `js/watercolor.js`, `js/geomod.js`, `js/geomodifiers.js`, `js/geomodPaint.js`, `js/stylePanel.js`,
`js/styleCatalog.js`, `js/definitions.js`, `tools/importManualStyles.js`. Counts were measured by script on the data,
not taken from older documents. What each parameter does was read from the code; the styles were **not** rendered
side by side for this audit.

## The answer to the first question: Model C, leaning to B

Draw5 is a **hybrid**. There is one small shared vocabulary of settings and one shared output record, but each
rendering look is a hand-written function that reads the shared settings with its own fixed numbers.

* **Shared (the A part).** 16 numeric settings, 3 overlay choices and 1 recipe choice. 25 palette roles. 42
  materials mapped to roles. One appearance record that every look fills in: `fill, gradient, stroke, strokeWidth,
  opacity, shadow, glow, dash, jitter, texture, passes, fillMods`.
* **Bespoke (the B part).** 13 **kinds** in code (`Styles.Kinds`). Each has an `adjust()` function that turns a
  material colour into that record. The same slider means different things in different kinds, and most of what
  makes a kind look like itself is a constant inside its function, not a parameter.
* **Consequence.** A style definition can only pick one kind and set values. It cannot combine two kinds, cannot add
  a setting, and cannot reach a mechanism its kind does not call.

Two registries of the right shape already exist beside the styles: the geometry modifiers (`GeoMod.register`, 13
modifiers, 106 declared parameters with label, range, step, default, category, description) and the watercolour
recipes (data that binds sliders to modifier parameters as `{base, setting: factor}`). The Line & Wash kind is the
only kind built that way.

---

## A. Current core styles

13 kinds. 12 have a built-in style. `flat` has none (it is the fallback).

| Kind (id) | Built-in style | What makes it distinct | Own drawing technique |
|---|---|---|---|
| `paper` | Paper Diorama | matte fill, thin darker edge, soft offset shadow that grows toward the front, corner jitter | no (fill, stroke, shadow) |
| `woodblock` | Japanese Woodblock | flat fields, ink contour 2.6 px on every shape, far layers fade to paper | no |
| `neon` | Neon Vector | fills dimmed 80 % to the background, coloured stroke, blur glow, far layers lose alpha | no |
| `blueprint` | Blueprint | no gradients, fill 14 % of colour, light line, dashed guides | no |
| `silhouette` | Backlit Silhouette | shapes hazed to the horizon colour by distance squared, named light materials glow | no |
| `biolume` | Bioluminescent Deep-Sea | translucent bodies (opacity 0.62 – 0.92), rim stroke, glow, far layers sink into the background | no |
| `pixel` | 8-bit Pixel | gradients cut into hard bands, ink edge, **the frame drawn at low resolution in Play** | yes: resolution |
| `comic` | Comic Halftone | two hard cel tones, ink contour thicker near, hard offset shadow, halftone tile on some materials | no (uses a tile) |
| `chalk` | Chalkboard | fill rubbed in at 20 – 34 %, dashed outline, chalk tile, far layers erased | no (uses dash and a tile) |
| `sketch` | Graphite Sketch | two loose stroke passes, hatching whose density follows the darkness of the colour | yes: geometry passes |
| `wash` | Line & Wash | built from a data recipe: bleed, body, glaze, blooms, gaps, pooled edge, pen ribbon | yes: geometry passes from a recipe |
| `impasto` | Thick Impasto | roughened fill edge, two layers of brush dabs, lit ridge, thickness shadow | yes: geometry passes |
| `flat` | (none) | colour and a darker outline | no |

So: **4 kinds have a drawing technique of their own** (pixel, sketch, wash, impasto). The other 9 are different
arithmetic over the same four primitives: fill colour, stroke, one blur-shadow slot, a tiled texture.

## B. Current variations

| What | Count |
|---|---|
| Kinds (rendering looks in code) | 13 |
| Built-in styles | 12 |
| Catalog entries (`js/styleCatalog.js`, imported from the Game Design Manual) | 632 |
| of those, aliases that just select a built-in | 9 |
| of those, variants | 623 |
| Variants that differ from their built-in **only by palette** (no setting, or only settings their kind ignores) | 420 |
| Variants that change at least one setting their kind reads | 203 |
| Distinct setting values used across the whole catalog | 11 values over 7 settings |
| Settings written by the importer that the entry's kind never reads | 105 of 396, in 99 entries |
| Entries whose palette came from the manual's demo code | 40 |
| Entries whose palette came from colour words in the text | 191 |
| Entries whose palette is a category default shifted in hue | 401 |
| "Hybrid" entries (two looks named); only the first is used | 29 |
| Pairs identical in kind, settings and colours | 3 |
| Variants defined by demo levels | 2 (Petal Paper on `paper`; Crimson Moon 16-bit on `pixel`) |

The 632 are **not** 632 technologies. They are 12 looks × a palette, and for a third of them one or two preset
slider values. The importer chose the kind by scoring keywords (paper 132, neon 107, wash 65, blueprint 65,
silhouette 60, biolume 43, comic 37, woodblock 35, pixel 27, impasto 27, chalk 21, sketch 13). Nothing is installed
until *Use* is pressed; then the entry becomes a variant in the scene.

Examples that show the limit: **Embroidery** is `paper` with a palette. **16-bit SNES-era** is `pixel` with a palette,
`texture 0.75` and `shadow 0.6`, and the pixel kind does not read `shadow`. Four `paper` entries carry `glow 0.85`,
which the paper kind ignores.

## C. Current parameter inventory (existing parameters only)

### C1. Style settings (`Styles.SETTINGS`, `DEFAULT_SETTINGS`)

| Id | Label in the UI | Range, step | Default | Category | Read by | Affects |
|---|---|---|---|---|---|---|
| `lineWeight` | Line weight | 0 – 3, 0.05 | 1 | Stroke | all kinds but silhouette | stroke width (a multiplier) |
| `outline` | Outline | 0 – 1, 0.05 | 0.5 | Stroke | flat, paper, woodblock, blueprint, silhouette, pixel, comic, chalk, sketch, wash, impasto | closed-shape edge width (a multiplier) |
| `shadow` | Depth shadow | 0 – 1, 0.05 | 0 | Shadow | paper, comic, impasto; also the particle effects | the shape's drop shadow |
| `glow` | Glow | 0 – 1, 0.05 | 0 | Lighting | neon, silhouette, biolume; also the particle effects | blur halo around the shape |
| `texture` | Texture | 0 – 1, 0.05 | 0 | Texture | every kind (the scene overlay); plus comic, chalk, sketch, wash, impasto in their own way | see E |
| `depthFade` | Depth fade | 0 – 1, 0.05 | 0.5 | Layering | all but flat and blueprint | colour or alpha by layer depth |
| `jitter` | Cut-edge jitter | 0 – 1, 0.05 | 0 | Edge / geometry | paper, chalk, sketch, wash, impasto | see E |
| `saturation` | Saturation | -1 – 1, 0.05 | 0 | Colour | every kind (applied to each palette colour) | fills and strokes |
| `brightness` | Brightness | -1 – 1, 0.05 | 0 | Colour | every kind | fills and strokes |
| `contrast` | Contrast | -1 – 1, 0.05 | 0 | Colour | every kind | fills and strokes |
| `pixelSize` | Pixel size (screen px) | 1 – 12, 1 | 4 | Pixel / display | pixel | frame resolution, Play only |
| `wetness` | Wetness (bleeding) | 0 – 1, 0.05 | 0 | Material (watercolour) | wash | geometry passes |
| `bloom` | Blooms / backruns | 0 – 1, 0.05 | 0 | Material (watercolour) | wash | geometry passes |
| `pooling` | Edge pooling | 0 – 1, 0.05 | 0 | Edge (watercolour) | wash | a stroke pass |
| `coverage` | Coverage (1 = no paper gaps) | 0 – 1, 0.05 | 1 | Fill (watercolour) | wash | fill passes |
| `penVariation` | Pen weight variation | 0 – 1, 0.05 | 0 | Stroke (watercolour) | wash | the pen ribbon pass |
| `overlay` | Texture type | one of 17 tiles, or none | the kind's | Texture | every kind | the scene overlay |
| `overlayBlend` | Blend | source-over, multiply, overlay, soft-light, screen | source-over | Compositing | every kind | the scene overlay |
| `overlayScale` | Texture scale | 0.25 – 4, 0.05 | 1 | Texture | every kind | the scene overlay |
| `washRecipe` | (no control) | lineAndWash, wetInWet, boldInk | lineAndWash | Material (watercolour) | wash | which recipe builds each shape |

None of these has a description in the UI. The label is all there is.

### C2. Other style data

* **Palette roles (25):** bg, sky, skyHigh, sun, cloud, far, hill, foliage, trunk, water, waterDeep, foam, sand,
  sandWet, stone, wood, wall, roof, sail, accent, hazard, pickup, creature, light, ink.
* **Materials (42 + auto):** each maps to a role, optionally a gradient of roles. Per material and style: `fill`
  (role or hex), `gradient`, `texture {type, scale, opacity, blend}`. Flags `backdrop`, `hero`, `lit`, `guide` are
  read by kinds. The flag `glow: true` is set on 8 materials and **no kind reads it** (silhouette lists material
  names in code instead).
* **Per object:** `material`, `styleMode: 'authored'`, `styleOverrides[styleId] {fill, stroke, strokeWidth, stops}`,
  `texture {type, scale, rotation, opacity, blend}`.

### C3. Texture tiles (17)

none, paper, fiber, grain, noise, canvas, hatch, dots, stripes, scan, grid, motes, pixelgrid, halftone, chalk,
coldpress, watercolor. Each is a 256 px procedural tile. Usable as the scene overlay, per material, per object.

### C4. Geometry modifiers (13, with 106 declared parameters)

roughen (Edge), wave (Edge), protrude (Growth), taper, bend, smooth, offset, symmetry (Form), shift (Transform),
sketchStroke, hatch, inkRibbon (Stroke), blooms (Growth). These carry full metadata already. Styles reach them only
through kind code (`ap.passes`, `ap.fillMods`); a style definition cannot name one.

### C5. Hidden parameters (fixed numbers inside the kinds)

These exist in the engine and are **not exposed**. A sample of the ones that define each look:

| Kind | Fixed in code |
|---|---|
| paper | edge = colour darkened 22 %, width 1.3; shadow colour `#2a1a10`, opacity 0.28, blur 2 + 5 × depth, offset up to 5 / 7.5 px; jitter × 2.6 px; far fade 25 % |
| woodblock | contour width 2.6; far fade 30 % |
| neon | interior dimmed 80 %; stroke 2; glow blur 14 (hero 24); far cut-off at depth 0.3 |
| blueprint | fill 14 % of colour; line = 65 % white; widths 1.4 / 1.5; dash 8, 6 |
| silhouette | haze = (1 − depth)² × 0.75; rim 0.8 px; glow blur 22; which materials are lights |
| biolume | body 42 % of colour; opacity 0.62 + 0.3 × depth; glow blur 18 (hero 30) |
| pixel | band count = max(4, 2 × stops); edge 2 px |
| comic | cel split at 50 %; contour 1.6 + 1.6 × depth; shadow offset 3.5, blur 0, opacity 0.9; halftone opacity 0.45 / 0.55; which materials get dots |
| chalk | fill 20 % + 14 % × depth; dash 10 + 3, gap 1.8; texture only above 3 000 px² |
| sketch | hatch angle 52°, spacing 2.4 – 10.4, cross-hatch above tone 0.55; stroke length 70 and 45; gaps 0.08 and 0.35 |
| wash | everything in the recipe (data): offsets, roughness, alphas, bloom count, pen taper |
| impasto | dab angle 20° ± 35°, widths 4.5 and 3.5, ridge offset −1.1 / −1.3; shadow blur 1.5 |
| every kind | overlay opacity and vignette colour; the per-kind tone steps for 2.5D faces (`DIM_LOOK`) |

## D. Parameter → style matrix

● = the kind reads it. ○ = shown as a slider but not read by the kind's own function.

| Kind | lineWeight | outline | shadow | glow | texture | depthFade | jitter | pixelSize | wetness, bloom, pooling, coverage, penVariation |
|---|---|---|---|---|---|---|---|---|---|
| flat | ● | ● | | | ○ (overlay is none) | ○ | | | |
| paper | ● | ● | ● | | ● overlay | ● | ● | | |
| woodblock | ● | ● | | | ● overlay | ● | | | |
| neon | ● | | | ● | ● overlay | ● | | | |
| blueprint | ● | ● | | | ● overlay | | | | |
| silhouette | | ● | | ● | ● overlay | ● | | | |
| biolume | ● | | | ● | ● overlay | ● | | | |
| pixel | ● | ● | | | ● overlay | ● | | ● | |
| comic | ● | ● | ● | | ● overlay + dots | ● | | | |
| chalk | ● | ● | | | ● overlay + chalk tile | ● | ● | | |
| sketch | ● | ● | | | ● overlay + hatching | ● | ● | | |
| wash | ● | ● | | | ● overlay + granulation | ● | ● | | ● |
| impasto | ● | ● | ● | | ● overlay + dab strength | ● | ● | | |

Saturation, brightness, contrast and the three overlay settings apply to every kind.

Values the 12 built-ins set (everything else is the default):

| Style | Settings |
|---|---|
| Japanese Woodblock | outline 1, texture 0.55, depthFade 0.45 |
| Paper Diorama | outline 0.6, shadow 0.8, texture 0.4, jitter 0.5 |
| Neon Vector | glow 0.8, texture 0.35 |
| Blueprint | outline 1, texture 0.6 |
| Backlit Silhouette | outline 0.3, glow 0.6, texture 0.5, depthFade 0.8 |
| Bioluminescent Deep-Sea | glow 0.9, texture 0.55, depthFade 0.7 |
| 8-bit Pixel | outline 1, texture 0.35, depthFade 0.35, pixelSize 4 |
| Comic Halftone | outline 1, shadow 0.7, texture 0.7, depthFade 0.3, saturation 0.15 |
| Chalkboard | outline 1, texture 0.6, jitter 0.4 |
| Graphite Sketch | outline 1, texture 0.8, depthFade 0.6, jitter 0.4 |
| Line & Wash | outline 1, texture 0.7, jitter 0.5, wetness 0.6, bloom 0.5, pooling 0.6, coverage 0.75, penVariation 0.65, washRecipe lineAndWash |
| Thick Impasto | outline 0.7, shadow 0.5, texture 0.8, jitter 0.5, saturation 0.1 |

## E. What the important parameters do

| Parameter | Up | Down | Notes |
|---|---|---|---|
| lineWeight | every stroke thicker, in proportion | thinner; 0 removes strokes | continuous. In sketch, wash, impasto it also scales the pass strokes (pencil, pen ribbon, dabs) |
| outline | the edge of closed shapes thicker | thinner; at 0 no edge (blueprint and chalk keep 30 %) | continuous. Multiplies a per-kind base width (0.8 silhouette … 2.6 woodblock). Does not touch open lines |
| shadow | **paper:** shadow darker and further offset. **comic:** hard black offset larger. **impasto:** thickness shadow darker | lighter; 0 = none | three different shadows behind one slider. Blur and colour are fixed |
| glow | wider halo (blur radius) in neon, silhouette, biolume; particle effects glow more | tighter; 0 = none | the halo uses the same slot as the shadow: a shape has one or the other, never both |
| texture | the scene overlay stronger. **comic:** dots denser in tone. **chalk:** dustier. **sketch:** more and denser hatching. **wash:** more granulation. **impasto:** stronger dabs | weaker; at 0 the overlay and vignette are not drawn at all | one slider drives two unrelated things in five kinds: paper surface and shading amount |
| depthFade | far layers pushed further toward the background (paper, woodblock, pixel, comic), the horizon colour (silhouette, impasto) or transparent (neon); sketch gets sparser | all layers read alike | needs layers with different depth; no effect in a one-layer scene |
| jitter | **paper, chalk:** corner points of polygons moved up to 2.6 px. **sketch:** strokes overshoot, wobble and drift more. **wash:** wash further from the line, rougher. **impasto:** rougher paint edge | cleaner edges | in paper and chalk it moves only the corner points of closed polygons: rectangles, ellipses and smooth paths do not change |
| saturation, brightness, contrast | palette colours more saturated / lighter / further from mid grey | the reverse | applied to each colour before the kind runs. Not a filter on the frame |
| pixelSize | larger screen pixels | finer | pixel kind only, **in Play only**; the editor shows full resolution |
| wetness | the wash creeps further outside its shape and mixes with neighbours; blooms and gaps increase | washes stay inside | wash only |
| bloom | more cauliflower backruns in large shapes | none | wash only, shapes over 2 500 px² |
| pooling | darker, wider broken line at the wash edge | none | wash only |
| coverage | fewer specks of bare paper | more bare paper | wash only |
| penVariation | pen line swells and thins, blots appear | even line | wash only |
| overlay, overlayBlend, overlayScale | which tile covers the scene, how it mixes, how large | | categorical, categorical, continuous |

## F. Existing rendering mechanisms

| Mechanism | Where | Reached by styles today |
|---|---|---|
| Solid and linear-gradient fill by material and palette role | `StyleService.resolve` | all kinds |
| Gradient banding (hard steps) | pixel and comic kinds | only those two, fixed step counts |
| Stroke with width, dash, round caps | renderer | all; dash only blueprint, chalk and passes |
| One blur-shadow slot per shape: offset shadow **or** glow | `renderer.js` (Konva shadow) | paper, comic, impasto (shadow); neon, silhouette, biolume (glow) |
| Shape opacity | renderer | neon, biolume |
| Polygon corner jitter | `jitterPoints` | paper, chalk |
| Tiled texture clipped to a shape, with blend mode | `_wrapTexture`, 17 tiles | per material, per object, comic, chalk, wash |
| Scene overlay tile + vignette, with blend mode | `drawOverlay` | all kinds |
| Geometry passes: extra fills and strokes built from modifier stacks, each with colour, width, alpha, offset, dash, blend | `renderer.js` contour pipeline | sketch, wash, impasto |
| Fill geometry modifiers (the fill differs from the outline) | `ap.fillMods` | wash, impasto |
| Data recipe binding sliders to modifier parameters | `watercolor.js` | wash |
| Low-resolution frame scaled with hard pixels | `App.applyRenderMode` | pixel, Play only |
| Tone steps for 2.5D faces per kind | `DIM_LOOK` | all, fixed |
| Effect language for particles (glow, paper, ink, line, pixel, biolume, silhouette, flat) | `fx.js` | all, fixed per kind |

Outside the style system, and not driven by it: dynamic fills (8 painters: scroll, sweep, pulse, shimmer, light,
accumulate, wet, edge), advanced effects (fractal terrain, wave bands, star trails, starfield, flow), the light and
shadow layer of the Dimension system, weather.

## G. JSON, schema, import

* **Stored state:** `doc.styles = { activeId, custom: {builtInId: delta}, variants: [{id, name, base, palette,
  settings, materials, catalogId?, description?, source?}] }`. In a project the customisations and variants come
  from the project, overridden by the scene.
* **Identity:** an id. Built-ins are fixed in code. A variant is a **delta on a base**, resolved up to 8 levels deep.
* **Renderer identification:** `kind`. A variant always has its base's kind. Changing kind means importing a new style.
* **File format:** `D5STY`, extension `.d5style.json`: `{id, name, kind, basedOn?, description, source?, palette,
  settings, materials}` in the common definition envelope, exported with the vocabulary.
* **Import:** Look ▸ Styles ▸ Import style…, or the Definition box ▸ Import as new style. Validated, then stored as
  a variant on the first built-in of that kind. Never overwrites a built-in.
* **Validation (`validateDefinition`):** name required; kind must be one of the 13; palette keys must be known roles
  and `#rrggbb`; settings must be known and in range; materials must be known.
* **Unknown data:**

  | Where | What happens |
  |---|---|
  | unknown setting | **rejected**: "settings.x: unknown setting" |
  | unknown palette role | rejected |
  | unknown material | rejected |
  | unknown kind | rejected |
  | unknown top-level key | accepted and **dropped** (not stored) |
  | unknown key inside a material entry | accepted and **kept**, never shown |
  | unknown setting already inside a scene file | kept, ignored by every kind, never shown |

* **Can an imported parameter appear in the UI?** No. The sliders are the fixed `SETTINGS` list filtered by the
  kind's `uses` list.
* **Anything like a registry?** `Styles.schema()` and `Styles.vocabulary()` describe kinds, settings, roles and
  materials as data, with `usedBy` per setting. They are hand-kept lists. The geometry modifier registry is a real
  one but styles cannot address it.
* **Defect found:** the built-in Line & Wash does not survive its own export. Its definition contains
  `settings.washRecipe`, and the validator rejects that key. Not fixed in this audit.

## H. Current UI and workflow

* **Where:** side panel ▸ Look ▸ Styles. Also a style switcher with favourites in the top bar, and style cards in
  Library ▸ Definitions.
* **The panel, top to bottom:** Import, search, the list of built-ins with their variants, the catalog (collapsed,
  632 rows, search and two filters, Use, Install whole category), Customise (Save as variant, Reset, Duplicate,
  Rename, Delete), Palette (25 colour wells, generate from a harmony, preset palettes), Settings, Materials in this
  scene, Definition (the JSON), Bake and Extract.
* **Why styles show different sliders:** the panel shows a setting only if the kind lists it in `uses`.
* **Preview:** the scene itself, live. A style card shows 8 colour swatches. There is no thumbnail drawn in the style
  and no fixed test picture.

Problems for the goals stated:

1. No global view of parameters. The 20 settings can only be seen one kind at a time.
2. No description of any slider. "Texture" does five different things and nothing says so.
3. The kind, which is what actually defines the look, is shown as a small grey label and cannot be changed or opened.
4. The numbers that define each kind are in code and not visible at all.
5. The catalog presents 632 names as if they were styles. Nothing shows that 420 of them are a palette on a
   built-in, or that a quarter of the settings written into them do nothing.
6. No comparison: two styles cannot be seen side by side, and a change cannot be compared with the original.
7. Preview depends on the open scene. A parameter that needs depth, a large shape or a gradient shows nothing in a
   scene without one.
8. `washRecipe` has no control. Overlay opacity and vignette have no control.
9. Edits to a built-in are a hidden delta marked only by an asterisk.
10. `pixelSize` appears to do nothing until Play is pressed.

## I. What the system cannot express

### Missing parameter (the mechanism exists, the control does not)

* Glow on paper, woodblock, comic, chalk, sketch, wash, impasto, pixel. The halo is one renderer call; only three
  kinds make it.
* Shadow colour, blur and direction in every kind that has a shadow.
* Glow colour separate from the stroke colour; glow per material (the unused `glow` flag).
* Edge colour and how much darker the edge is than the fill.
* Dashed outlines in any kind but blueprint and chalk.
* Gradient banding outside pixel and comic, and the number of bands.
* Shape opacity outside neon and biolume.
* Hatching, loose strokes, roughened fill edges, offset washes, brush dabs in any kind but the three that call them.
  All 13 geometry modifiers and their 106 parameters are unreachable from a style definition.
* Halftone or any tile as shading on chosen materials in any kind but comic.
* Overlay opacity and vignette colour.
* The watercolour recipe choice, and editing a recipe.
* Every fixed number in C5.

### Missing rendering mechanism (the engine cannot do it)

* **A pass over the finished frame.** There is none. So no palette quantisation, no ordered or error-diffusion
  dithering, no CRT curvature, no real scanline or phosphor mask, no bloom, no blur, no chromatic aberration, no
  whole-frame colour grading. The only frame-level thing is the resolution ratio and a tiled overlay.
* **Shadow and glow together, or more than one of either.** One slot per shape.
* **Inner shadow, inner glow, bevel, emboss.** Nothing lights the inside of a shape from its edge.
* **Marks placed along a contour as shapes.** A dash is a gap in a line. There is no stitch, bead, tooth or stamp
  that is its own small shape with its own shadow, repeated along an edge or across a fill in a set direction.
* **A fill built from oriented rows that follow the shape** (satin stitch, wood grain, fur direction). Hatching is
  straight lines at one angle.
* **Colour variation inside one fill** other than a linear gradient or a multiplied tile (mottling, pigment
  variation, dither between two palette colours).
* **Texture that follows the shape or its edge** (frayed border, deckle edge, weave that bends). Tiles are flat and
  screen-aligned to the shape's box.
* **Radial or shaped gradients from a style.** Styles produce top-to-bottom linear gradients only.
* **Combining kinds.** One kind per style, chosen at creation.
* **Animated style behaviour** (flicker, boil, scan roll). Kinds are static; motion is in other systems.
* **Pixel look in the editor.** Low resolution is applied in Play only.

## J. Future style candidates (proposal only)

| Family | Already reachable | Missing parameter | Missing mechanism |
|---|---|---|---|
| **Pixel / retro (8, 16, 32-bit)** | low-res frame, hard bands, ink edge, pixel-grid and scanline tiles | band count, pixel size in the editor, dark outline on or off | frame pass: palette restriction, dithering, CRT mask and curve, bloom |
| **Embroidery / thread** | canvas tile for cloth, dashed offset outline as a running stitch, hatch pass as flat stitch rows, pass offset as a thread shadow | all of those, since no kind calls them this way | stitches as shapes along a contour, rows that follow the shape, frayed edge |
| Watercolour, gouache | wash kind and its recipes | recipe choice and recipe values | colour variation inside a fill |
| Crayon, pencil, charcoal | sketchStroke, hatch, roughen, paper tiles | all as style parameters | grain that breaks the stroke by the paper (stroke × texture) |
| Ink wash | wash recipe `wetInWet`, banding | number of tones | soft-edged fill (blur) |
| Woodcut, linocut | woodblock, banding, roughen, shift | carve-line hatch on dark tones | none essential |
| Screenprint, risograph | `shift` misregister, halftone tile, multiply passes | per-colour misregister amount, dot size | none essential |
| Collage, paper craft | paper kind, shadows, textures per material | torn edge from `roughen`, shadow direction | shadow plus edge highlight together |
| Felt, wool | paper kind, noise tile | fuzzy edge from `roughen` or `protrude` | fibres along the edge as shapes |
| Clay, stop-motion | nothing close | | inner shadow and highlight (bevel), fingerprint mottle |
| Stained glass | thick dark contour, glow | lead width, glass glow | light variation inside a pane |
| Comic, halftone | comic kind | dot size and angle, which materials, shadow direction | dots sized by tone (true halftone) needs a frame or fill pass |
| Neon, synthwave | neon kind, scan tile | glow colour, tube width | bloom, double glow |

The two families named first both test the same two gaps: a **frame pass** (pixel) and **marks as shapes along and
across a shape** (embroidery). Both gaps also serve most of the other rows.

There is one partial 16-bit already: the ninja level's *Crimson Moon 16-bit*. It is the pixel kind with its own
palette and `pixelSize 3`. It gives low resolution and hard bands. It has no palette restriction, no dithering and
no display effect.

## K. Architecture recommendation (not implemented)

1. **Make the mechanism the unit, not the kind.** Register each rendering mechanism the way geometry modifiers are
   registered: an id, where it acts (colour, fill, edge, pass, shadow / glow, shape texture, scene overlay, frame),
   and its parameters with id, label, description, category, type, range, step, default. The registry is then the
   list of registered mechanisms. No second list to keep.
2. **Make a kind a recipe in data.** A kind becomes a named list of mechanisms with their parameter values or
   bindings, in the value form the watercolour recipes already use (`{base, setting: factor}`). The 13 kinds are
   rewritten as 13 recipes that draw exactly what they draw now.
3. **Derive usage.** "Which styles use this parameter" and "which parameters does this style use" are computed from
   the recipes and definitions, never written by hand.
4. **Declare compatibility by slot.** Two mechanisms conflict when they claim the same exclusive slot. Splitting the
   shadow / glow slot in two removes the one hard conflict that exists today.
5. **Add the two missing stages only when a chosen style needs them:** a frame stage, and a mark-placement modifier.
6. **Relax import.** An unknown parameter is kept and reported, not rejected, and shown as unbound until a mechanism
   claims it.
7. **Move in steps that can be checked.** First lift the fixed numbers of C5 into declared parameters with the same
   defaults, and prove every built-in draws the same picture. Only then open combinations.

Open decisions for the review: whether the 632 catalog entries stay as they are, are re-labelled as palettes, or are
re-derived once the vocabulary is larger; and whether `texture` and `jitter` are split into the separate things they
control.
