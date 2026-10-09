# Draw5 — Design Tools & Workflows: audit, organisation, sequence

Written 2026-10-09 on the user's brief, after the Style Manager (SM-1 to SM-5) was reviewed by hand. This is an
**audit and a plan**. Nothing of the editor's layout was changed by it. One runtime defect found on the way was fixed
(§6); everything else is recorded for a coordinated implementation at the point §8 names.

It complements `UI_WORKFLOW_AUDIT.md` (2026-10-08), which inventoried the **definition kinds** (entity, ability,
weapon, moveset, AI, sound …) and led to workflow pass 1 (the hubs Library · Look · Game, *who uses it*, return paths).
That audit is not repeated here. This one covers what it left out: the tools with which a thing is **drawn, built and
given its surface**, and how those connect to the systems that make the thing behave.

## 1. Assessment in one table

| | Item | Why |
|---|---|---|
| **Fixed now** | A dynamic fill did not show in Play in any style that gives shapes a shadow (Paper Diorama, the default) | a confirmed functional bug, four files, twelve lines, proved by a probe that fails without it (§6) |
| **Proceed** | P14.3, the completion of audio | still the next milestone of the main roadmap; nothing found here blocks it (§9) |
| **Defer, in order** | DT-1 (what this is and what can be done with it), DT-2 (the Object Designer), DT-3 (handoff both ways, paths), DT-4 (foundations that block) | none is a prerequisite of P14.3; DT-1 is worth placing before P15 (§8) |
| **Defer, no date** | advanced style mechanisms and cross-system effects (§10); layout polish (§8.5) | not needed for a coherent workflow |

## 2. What exists: the design and construction capabilities

One row per family. *Access* is where a person reaches it today. *Scope* is what it acts on.

| Family | What exists | Access today | Scope | Depends on | Limits | Opportunity |
|---|---|---|---|---|---|---|
| **Drawing** | 13 tools: select, rectangle, ellipse, line, polygon, freehand, pen (bezier), spline, text, gradient, erase, warp, motion path. Vertex and path-point editing, pivot, transform handles (`tools.js`, `handles.js`) | the toolbox (left) | a shape | — | the toolbox names tools, not purposes: *Motion Path* sits among drawing tools | group the toolbox by purpose; say what a tool applies to |
| **Generators ("Engines")** | about 70 parametric, seeded, regenerable prefabs in 15 files (`js/gen/`): **building** (27 parameters: stories, width, story height, bays, roof kind, pitch, overhang, door and door style, windows, shutters, lit windows, chimney, smoke, balcony, awning, sign, colour scheme, wall and roof material, timber frame, fence, flower boxes, bushes, lamp, path, rooftop), tree, flower, terrain, mountains, ground detail, water, fire, torch, critter, figure, aura, obstacles, volumes (block house, lintel, stairs), HUD meter. A placed instance is an ordinary group that remembers `obj.gen {type, preset, params, seed}` | Look ▸ Engines (browse, configure, audition variations by seed, *Vary*, *Refine from this*, save a configuration D5CFG); Library tiles; Inspector ▸ **Generated** on an instance (parameters, seed, Detach, *Open in Engines*) | an object (a group) | styles (materials), the library | a generator's `build` is code: a new body plan cannot be made in the editor (recorded gap, benchmark 1a). To change one window by hand the instance is **detached** and loses its parameters. Engines lives under *Look* although it makes things | the strongest existing design tool and the hardest to find from a placed thing; the natural core of an Object Designer (§7.3) |
| **World construction** | `scatter` with distributions (`placement.js`), spatial context and construction rules (`construction.js`): placement along a path or inside a region, scale by depth, orientation, keep-clear | the same Engines panel (family Placement / construction); Inspector ▸ Generated; the Erase tool takes one instance out | a region or a path of a scene | generators, paths, regions | which paths and regions a construction reads is a parameter, not shown on the path | list on a path what uses it (§7.5) |
| **Gameplay pieces** | platform, bridge, gate, lever and others as generators of family *gameplay* with connectors, snap and extend (`pieces.js`) | Library tiles (Level geometry, Obstacles) | an object | generators, rules | — | belongs to the same designer as other generated things |
| **Geometry modifiers** | a non-destructive stack on any vector shape (`obj.geometry`): 14 algorithms — roughen, wave, **protrude**, taper, bend, round corners, grow / shrink, shift, **symmetry**, sketch strokes, hatching, marks along the outline, ink ribbon, blooms — and 29 presets (fur, grass, feathers, pine tips, clouds, claws …) applied as a live link with per-object overrides and seed | Inspector ▸ **Geometry (procedural)**, only once a shape is selected | a shape | — | nothing outside the Inspector says it exists; presets are per document | surface it as "Shape" tools in the context header (§7.2) |
| **Warp** | mesh or perspective warp of any object, text, group or effect region; Bake (`warp.js`) | the Warp tool (W); Inspector ▸ Transform (warp) | any object | — | — | same |
| **Appearance** | fill: solid, linear or radial gradient (and the Gradient tool); outline; a **texture** on any fillable shape (13 tiles, among them paper, canvas, hatching, halftone, stripes, scanlines, **drafting grid**, **pixel grid**); colour drag and drop onto the canvas (nearest colour slot; Ctrl one shape, Shift the scene); the **material** of a shape, through which a style decides its colours; a per-object, per-style override; bake and extract (`recipes.js`) | Inspector ▸ **Appearance**, ▸ **Style**; the Gradient tool; any colour chip | a shape; a group through its shapes | styles | the material is the link between a shape and the Style Manager, and nothing on the shape leads there. Pixel treatment exists twice: a texture (per shape) and a frame mechanism of a style (whole picture) | from a shape: *this is material Roof; the scene's style draws Roof like this* with a way into the Style Manager |
| **Dynamic fills** | 8 painters as effect types (scroll, sweep, pulse, shimmer, light, accumulate, wet, edge); accumulate and wet follow a number, time, the weather, a deposit or a game variable (`fillfx.js`, `DYNAMIC_FILLS.md`) | Inspector ▸ **Effects** ▸ + Add ▸ *Fill · …*; the toolbar's ✦ Effects preview | a shape; a group (its shapes, optionally one material) | the kernel system `fills`; the environment; game variables | listed among effects, away from Appearance where the fill is edited. A fill that follows a game variable shows empty in the editor preview (no game runs there). **Defect fixed, §6** | show them beside the fill; let the preview set a level |
| **Object effects and simple motion** | effects: glow, shadow, trail, emitter, particles, sound, flicker (and the fills). Behaviours: sway, float, bob, rock, rotate, oscillate, drift, flap, pulse, mover, **follow path**, player control, actor, brain | Inspector ▸ Effects, ▸ Behaviors; context menu ▸ Add effect / Add behaviour; *+ Add capability* | an object | the kernel; paths | — | — |
| **Advanced effects (FX Lab)** | five procedural techniques (fractal terrain, wave bands, star trails, starfield, flow) inserted as a region or attached as an aura | Look ▸ FX Lab; Inspector ▸ Engine (procedural effect) | a region; an entity (aura) | — | not in the library index | — |
| **Composition** | groups; **Components** (named slots inside a group: one of several parts shown, a library asset added as a component, the configuration saved as an asset variant); Rig; *Part of …* on a child; sockets and items drawn on them (animation sets); attached effects (a spawn that stays on an entity: body, feet, head, a socket; in front, behind, around) | Inspector ▸ Components, ▸ Rig, ▸ *Part of*; Studio ▸ Equipment; rules and abilities for attachments | a group and its children | the library; animation sets | three ways to say "this thing is made of parts" (a generator's parameters, component slots, an entity's parts and equipment), each with its own home | one parts view in the Object Designer that shows which of the three a part is |
| **Paths** | the Motion Path tool draws a route and attaches the selected object to it (behaviour *follow path*); a path is also read by world construction (roads, rivers, shorelines) and by AI patrols | the toolbox; Inspector ▸ Behaviors; a construction's parameters | an object; a scene | — | one tool, three consumers, and no place that lists who uses a path | *who uses it* for paths; "give this a path" from the thing (§7.5) |
| **Classification** | game role (12: player, platform, enemy, creature, npc, collectible, hazard, environment, decoration, goal, spawn, none), material, tags, collider; an **entity instance** (`obj.entity`), a **generated instance** (`obj.gen`), a procedural region, text. `ObjectCaps.describe` turns any object into groups of capability rows, each a link (`app.goTo`) | Inspector ▸ Object; the capability strip; right-click ▸ Inspect capabilities (Shift+I) | an object | — | the classification is read for the strip but does not shape the Inspector: every object gets the same long list of sections in the same order | the basis of design in context (§7.1) |
| **Layer, scene, environment** | layers with hide / isolate / lock and a depth plane (parallax, zoom, y-sort); gameplay planes; world size; play camera; environment (weather, hour, season, wind); the scene's Dimension (view, light, shadows); turns and grid; forces; the scene's style and its history | the Layers tab; the Inspector with nothing selected | a layer; a scene | — | scene-level design (light, weather, style) and scene-level game settings share one list | a scene context header as for an object |

## 3. What already exposes these (kept as owners)

| Surface | Owns | Pattern worth reusing |
|---|---|---|
| **Inspector** | every per-object field | sections; pop-out |
| **Character Studio** (`studio.js`) | entity definitions: identity, movement, equipment, moveset, AI | an isolated stage; tabs; *What it can do now*; drafts with Save; definitions opened **beside** it; ← back |
| **Style Manager** (`styleManager.js`) | looks, mechanisms, parameters, experiments, library, generator | browser · preview · recipe; every row says how it stands and why; a candidate is seen before it is applied; baseline; history |
| **Engines panel** | generator configurations | choose → configure → audition → refine → save |
| **Library** (Assets · Definitions · Abilities), **Look**, **Game** | assets, 23 definition kinds, project settings | one row vocabulary, *who uses it*, Copy ▾ |
| **Ability Designer, Combo Preview, Motion Preview, brain designer** | abilities, strings, clips, brains | the game itself as the preview |
| `nav.js`, `references.js`, `defKinds.js`, `ObjectCaps`, `app.goTo` | navigation, usage, classification | **the plumbing a design context needs already exists** |

## 4. Findings

1. **Design tools appear only after the right thing is selected, as sections of one long list.** A person who has
   placed a house does not learn from the screen that its roof pitch is a parameter, that its outline can take a
   modifier, or that its windows can carry a moving light.
2. **The classification is known and unused.** `ObjectCaps` can say "a generated building, environment, no collider";
   the Inspector shows the same sections in the same order for it and for the hero.
3. **The same idea has three homes.** "Made from parameters": a generator instance, a geometry preset, a component
   configuration. "Looks like": Appearance, Style override, material, texture, dynamic fill, glow.
4. **Makers are filed under Look.** Engines and the FX Lab build things; styles colour them.
5. **Links into the owners exist one way.** An instance leads to the Studio, a generated thing to Engines. A shape
   does not lead to the Style Manager, a path does not lead to what follows it, and the Studio does not lead to the
   tools that draw a character's look.
6. **A preview and the game can disagree** (§6 was such a case). The editor has one preview profile; what it cannot
   show (a fill driven by a game variable) is not said.
7. **Unavailable is silent.** A section is absent when it does not apply; nothing says why (a dynamic fill on a
   line, geometry on a group).

## 5. Principle

**Design in context; manage each capability in the system that owns it; move between connected systems without
losing the place or the selected thing.**

From it:

* No hub per feature. One context layer over what exists, and one workspace for the one case that needs a stage.
* The Design layer creates and configures things and **opens** the owners. It implements none of them a second time.
* A capability that does not apply says so, with the reason and the nearest thing that does.

## 6. The dynamic-fill finding (classified, fixed)

**Report:** in a demonstration level the preview of dynamic fills works; a trigger meant to switch one on in the
game may not.

**Traced** in the Fill Lab (`js/worlds/fillLab.js`), by `test/probes/fill_trigger.js`:

| Step | Result |
|---|---|
| Definition: the tank carries *Accumulate* with level `var:fill`; the lever's rule says `setVar fill +1` on interact | as written |
| Editor preview (✦ Effects): the tank's fill is painted, empty | correct: no game runs, the variable is 0 |
| Play: the lever is an interactable; beside it the hero's interact reaches it | ok |
| Play: the key E raises `fill` (1, 2, 3); the toast says so | ok |
| Play: the painter's canvas shows the water rising (19 %, 38 %, 58 %) | ok |
| Play: **the screen** shows the water | **no: 0 %** |

**Cause.** In Play the renderer keeps every shape that has a shadow as a bitmap (`renderer.js` `cacheStatic`), for
speed. The fills system declared its shapes as changing, which kept them out of *shared* bitmaps, but a single shape
with a shadow was still cached on its own, before its first fill was painted. Paper Diorama gives every shape a
shadow, so in the default style no dynamic fill ever showed in Play: not the tank, and not the sea, the lava, the
snow on the roofs or the lit window either. The editor preview makes no caches, which is why it worked there. The
rule has been so since the feature was made (M12); it is not a regression of the style passes.

**Class:** a confirmed functional bug of integration (renderer cache against the fills system). The trigger, the
variable and the painter were all correct. It was not a discoverability problem, and moving a control would not have
changed it.

**Fix (the smallest that is not a workaround).** The kernel hook `dynamic` gained the answer `'paint'`: the object's
shapes are drawn anew every frame and none of them is kept as a bitmap, alone or in a batch. The fills system
answers it; the renderer honours it. `js/kernel.js`, `js/world.js`, `js/renderer.js`, `js/fillfx.js`, twelve lines.
Gameplay is not touched (drawing only).

**Evidence.** `fill_trigger.js`: 11 of 11 with the fix; 9 of 11 without it (nine fill shapes inside a cache, water on
the screen 0 %). Picture: `dist/shots/fill_trigger_play.png`. Unit tests and the sequences probe: §11.

**Left as found:** the Fill Lab's conveyor appears to be hidden behind the floor that is drawn after it (a mistake of
the level, recorded). A fill that follows a game variable cannot be tried in the editor preview (→ DT-1).

## 7. Proposed organisation

### 7.1 One registry of design capabilities

`ObjectCaps` grows into the single description of what can be done with a thing. Each capability family registers:

```
{ id, label, group: shape | build | surface | motion | parts | connected,
  scopes: [shape, group, generated, entity, envObject, component, layer, scene],
  applies(obj, ctx) -> true | { no: 'the reason', instead: <capability id> },
  owner: { where: inspector section | workspace | manager, open(subject, returnTo) } }
```

The Inspector, the context menu, *Inspect capabilities*, the Object Designer and the LLM manifest read the same
registry. A new capability registered once appears in all of them.

### 7.2 The context header (the Inspector stays)

With a thing selected the Inspector opens with **what this is** (one line from the classification: *Generated
building · environment · no collision*) and its tools in five groups, each a row of links:

| Group | Holds | Owner |
|---|---|---|
| **Shape** | geometry modifiers, warp, path points, symmetry | Inspector sections |
| **Build** | generator parameters, variations, components, parts | the Object Designer (§7.3); Inspector ▸ Generated for a quick value |
| **Surface** | fill, gradient, texture, dynamic fills, glow and shadow, material → how the scene's style draws it | Inspector ▸ Appearance; the Style Manager for the material |
| **Motion** | simple behaviours, follow a path, states | Inspector; the path tool |
| **Connected** | movement, abilities, weapon, AI, combat, encounter, dimension, sounds | the Studio, the Ability Designer, the brain designer … opened **with this thing as the subject** |

What does not apply is shown greyed with its reason. The sections below keep their content; their order follows the
classification (an environment object leads with Build and Surface, an entity instance with Connected).

By scope:

| Selected | Leads with | Connected entries |
|---|---|---|
| a shape | Shape, Surface | its group; its material in the Style Manager |
| a group or assembled object | Build (parts, components), Surface | Save as asset; Use as … |
| a generated environment object or building | Build (parameters, variations), Surface | Engines configuration; construction that placed it |
| an entity instance | Connected (the definition in the Studio), Motion (its path) | Studio tabs; abilities; AI; combat; *its look* → the Object Designer |
| a component of a group | *Part of* (the slot, its alternatives) | the parent |
| a layer | plane, visibility, what it holds | scene |
| nothing (the scene) | style, light and dimension, weather, camera | Style Manager; Game |

### 7.3 One workspace: the Object Designer

For things made of parts a section list is not enough. One workspace in the manner of the Style Manager and the
Studio, for a **generated or assembled object**:

* left: the parts (generator parameters by group, component slots, children) — one tree that says which kind each is;
* middle: the object alone on a stage (the Studio's stage), in the scene's style, with variations by seed and
  *compare with what is placed* (the baseline idea of SM-5);
* right: the selected part's parameters, modifier stack and surface;
* actions: apply to the placed thing, save as a configuration or an asset, place another.

The Engines panel's browse and audition become its *New from an engine*. Nothing new is stored: it edits `obj.gen`,
`obj.geometry`, components and appearance through the existing commands. The Studio opens it for an entity's look and
gets the result back.

### 7.4 Handoff between managers

One call for every jump, built on `nav.js` (homes, the return stack, *open beside*):
`Nav.open(owner, { subject: { objectId, defId, part }, returnTo })`. Each manager accepts a subject and offers ← back
to where the person was, with the same thing selected. Exists today: instance → Studio, generated → Engines,
Studio → a definition beside it. Missing: shape → Style Manager (its material), Studio → the look's designer,
anything → a path, Style Manager → "shapes that use this role".

### 7.5 Paths

A path stays one object made with one tool. Added around it: *who uses it* (`references.js`: followers, constructions,
patrols) on the path; *Give it a path* in the Motion group of any thing, which starts the existing tool with the thing
attached. No second path editor.

## 8. Implementation sequence

Substantial passes, each testable through the screen by a probe and then by hand.

| Pass | Kind | Contents | Done when |
|---|---|---|---|
| **DT-0** ✅ | runtime defect | the dynamic-fill cache (§6) | done 2026-10-09 |
| **DT-1 What this is, and what can be done with it** | discoverability and navigation | the capability registry (§7.1); the context header and section order by classification (§7.2); reasons for what does not apply; the missing one-way links (shape → material in the Style Manager, a path's users, *Give it a path*); the effects preview can set a level for a fill that follows a variable; Engines and FX Lab reachable from *Build* (their place in the side panel is the user's re-bucketing) | for a shape, a group, a generated building, an entity instance, a layer and the scene: the header names it, every listed tool opens its owner with the thing as subject and comes back; nothing drawn or stored changes |
| **DT-2 The Object Designer** | design workspace | §7.3, proved on the building, a tree and one assembled asset; Engines' browse and audition inside it | a placed house is opened, its roof pitch and one window treatment changed, a modifier put on its outline, compared with the placed one, applied; saved as a configuration; another placed |
| **DT-3 Handoff both ways** | cross-manager context | §7.4 complete: the Studio ↔ the look's designer, the Style Manager ↔ shapes of a role, an ability or a brain opened from a thing and returned from; the scene context header | a character's look is changed from the Studio and the Studio shows it; from a shape to its role's colour and back with the shape still selected |
| **DT-4 Foundations that block** | missing capability | only what DT-2 and DT-3 show to be in the way. Known candidates: a look as data (parts, joints, bones: recorded gap, benchmark 1a); a part of a generated instance changed without *Detach*; `dim` and `persist` on an entity definition (recorded gaps) | per item, when taken |

**8.5 Deferred polish:** toolbox grouping by purpose; the two pixel treatments explained in one place; narrow-column
layout of the Style Manager's history rows; the generator panel covering the Recipe; the Fill Lab's conveyor.

**Where in the roadmap.** P14.3 comes first (§9). **DT-1 is recommended between P14.3 and P15**: P15 is the Vehicle
Studio, a further authoring surface; with the registry and the handoff in place it plugs into them instead of adding a
hub of its own. DT-1 adds no engine capability and no stored shape, so it runs as a floating editor pass (working
rule 8), as workflow pass 1 did. **DT-2 and DT-3 after P15 and before P16**, which is the slot the roadmap already
names "the audit before Phase 4". DT-4 items go with the pass that meets them. Each starts on the user's word.

## 9. The main roadmap: verified

* The Style Manager's five stages are done and committed (`8a2c9a2`), and reviewed by the user (2026-10-09).
* **Next milestone: P14.3, the completion of audio.** Checked against the roadmap row, `P14_PLAN.md` §3 and the
  repository: P14.0 to P14.2 are done; nothing was started for P14.3; no later pass was begun.
* To reconcile inside P14.3 (existing work, no new prerequisite): six buses since P14.2 (`speech` was added), so the
  options screen has six lines; *Sounds* moved from the Inspector to the Game view in workflow pass 1; four recorded
  gaps name P14.3 as their home (the per-bus volume lines, the cramped bus row, and the open lists of P14.1 and
  P14.2 to be carried into the report); the level it owes must be a new place under *Demo breadth*.
* The implementation prompt is in `P14_PLAN.md` §9.

## 10. Style capability, later

Kept on the roadmap; not started here.

1. **The remaining limit of the Style Manager:** of 24 mechanisms, 6 can be added to any look (Hatching, Brush dabs,
   Lit ridge, Hatch lines, Stitches, Low resolution); 16 are drawn by one base look's own code and come only with that
   base. Giving them a function of their own (`STYLE_MANAGER_DESIGN.md` 15.2) is the next style step and is what
   widens the generator.
2. **Effects that cross systems**, for example watercolour pigment that diffuses (style × dynamic fills × particles)
   or loose threads that answer wind and movement (geometry × attachment × animation × forces). They need a mechanism
   to be able to own state over time and to read the environment, which no style mechanism does today.
3. **Direct authoring stays the goal.** An LLM writing a new capability is one optional way to extend the engine when
   an effect needs new code across systems (the registries and `EXTENDING.md` are the contract it would write to). It
   is not a step of ordinary design work.

## 11. Evidence of this pass

| Check | Result |
|---|---|
| `test/probes/fill_trigger.js` (new) | 11 of 11; without the fix 9 of 11 |
| Unit tests (`node test/run_all.js`) | 1483 pass, 0 fail (one test of `fillfx.test.js` asserted the old answer of the hook and was updated; it now also checks that the renderer and the world honour `paint`) |
| `test/probes/sequence_examples.js` (Play caches, the effects preview on the Fill Lab) | 69 of 69 |
| Not run | the pass regression and the pixel probes: the change is in what Play draws for shapes with a fill effect only, which no trace reads |
