| F-40 | Every demo world's tours per station (the Dojo: nine) | the Demo run select (47 worlds × tours in one select) | C |

### D.7 Functional bugs met on the way (recorded, not fixed here)

| # | Bug | Source |
|---|---|---|
| F-41 | A bound TEXT object does not redraw in Play | WICK finding 8; roadmap gap |
| F-42 | The Inspector builds every definition card's form on each redraw (~1 s after an edit in Sounds) | roadmap gap P14.2 |
| F-43 | A compact rule card closes after every field edit | WICK §6 |
| F-44 | A generated thing rebuilt from a parameter drops a hand-set collider silently | WICK §6 |
| F-45 | "Based on" a library definition keeps the library's ids inside inherited rules and a changed inherited rule stores without its event and fails validation | WICK §6, §9 |
| F-46 | The equip action lists only items already saved with an item block | WICK §6 |
| F-47 | A click on a menu also reaches the scene under it | roadmap gap P10 |
| F-48 | `when: {expr}` in an entity's interact rule did not fire; `feedback on: interact` on the player makes it interactable; `controller: none` does not stop an inherited brain | benchmark 1a defects |

### D.8 Missing capabilities (genuine)

| # | Missing | Note |
|---|---|---|
| F-49 | Reverse references / "used by" | computable from data; no storage change |
| F-50 | Rename an id with references following | references are ids in known fields **and** inside expressions and action strings: a rewrite needs a walker per kind |
| F-51 | A weapon from scratch; a demo editor / recorder; a transformation editor; a stance editor; a timeline for sequences (P17 per the roadmap) | |
| F-52 | A new body plan (look) as data (recorded gap, benchmark 1a) | out of scope here |
| F-53 | A reach overlay (jump height / length) in the editor; drop to ground; one origin convention | WICK §11 |
| F-54 | A resizable side panel; a second column | |

---

## E. Proposed target workflow

The principle: **one question, one place; every place links down to what it is made of and up to what uses it;
every reusable thing is listed once, in the Library, with its origin.** The data model does not change for any of
this: every proposal below is a new reading of `Definitions.list`, `Entities.resolve`, `Movesets.resolved`,
`Actor.inputFor`, `ObjectCaps.describe` and the existing transient stages.

```
Project
 ├── World (a scene): Layers · the canvas · Inspector of the selected thing · scene settings
 ├── Library (the project): EVERY reusable kind, faceted by origin (Built-in · Project · This scene) and by what it is
 │      a row: name · id · origin · used by N · [Open in its editor] · [▶ Preview] · [Copy] · [Based on]
 ├── Character Studio (a hub): a character → its look · movement · equipment · moveset → a MOVE card → its weapon /
 │      ability / clip / reaction, each a link to the specialised editor, opened BESIDE the Studio, with a way back
 ├── Motion Lab (what the Combo preview already is, named and reachable): clips · stances · profiles · reactions ·
 │      strings · a weapon's or an ability's hit — played by the game
 ├── Look (Styles · Engines · FX Lab · geometry presets): one tab with sub-views, one gallery of generated things
 └── Game (project settings): flow · state · screens · input · audio · runtime systems — and the scene's own:
        sequences · shots · demos · turns / grid · dimension
```

Workflows after the change:

* **Character**: Studio ▸ a character ▸ Moveset ▸ a move ▸ *Weapon: Katana → Edit* opens the weapon editor in a
  side dock, the Studio stays; *Clip: kesa → Edit* opens the Motion Lab on that clip with the character; Esc
  returns to the move. The stage column's *Hit reactions*, *Statuses*, *Brain* name what the character uses with
  the same Edit-beside links.
* **Animation**: Library ▸ Motion ▸ Clips (every clip of every set and profile, with its owner) ▸ ▶ plays it on a
  default figure in the Motion Lab ▸ *Copy into a profile of mine* ▸ edit keys with the clip editor ▸ *Used by*
  shows which characters perform it.
* **Moveset**: Library ▸ Movesets ▸ Samurai ▸ ▶ (the Motion Lab plays each move on a default fighter) ▸ *Make a
  copy* (asks: same id = replaces it for everyone / new id = a second set) ▸ edit ▸ *Assign to…* (a character
  picker: the Studio's Add Moveset).
* **Asset**: Library ▸ Assets ▸ a tile shows origin, size, collider, role; *Used in this scene: 4*; a placed asset
  can keep a link (an instance of an asset) in a later pass — not now.
* **Style**: unchanged; the object's Style section gains *Open the style ▸*; screens get the style later.
* **Level**: unchanged in its parts; the Inspector with nothing selected keeps only the scene's own settings
  (world, planes, camera, environment, dimension, turns / grid, sequences, shots, demos, runtime systems for this
  scene); project definitions move to the Library and project settings to Game.

---

## F. Proposed hub structure

| Hub | Owns (edits directly) | Lists and links to | Previews | Assigns | Contextual actions | Does NOT hold |
|---|---|---|---|---|---|---|
| **World** (the canvas + Layers + Inspector of the selection + scene settings) | objects, layers, planes, placed instances' overrides, rules on objects, the scene's settings and scene-scoped definitions (sequences, shots, demos, transformations) | the definition an instance follows (Studio), the engine that built an object (Look), the style (Look), an object's capabilities (all) | the canvas, ✦ Effects, Play, Play from here | — | capability strip; + Add capability; Inspect capabilities; Objects here | project definitions (moved to the Library); the two designers (AI, weapon: become Edit-beside links to their editors) |
| **Library** (one tab) | nothing but copies / Based-on / import / export / delete; the asset gallery as today | **every kind** of A.2 plus assets, pieces, engine presets, geometry presets, samples; facets: origin (Built-in / Project / This scene / Code), kind, *used by*; search over all | a ▶ per kind (its stage: entity → stage; move / clip / profile / reaction → Motion Lab; effect → burst; sound → play; style → apply live; screen → canvas; sequence → run) | Insert (assets), Apply (styles), Assign to… (abilities, movesets, sets, profiles, reactions, statuses, brains → a picker of entities, opens the Studio there) | Open in its editor; Copy (same id / new id); New based on; Export; Delete with *used by* | editing forms (a row opens the owner's editor) |
| **Character Studio** (the hub for entity definitions) | entity definitions: Character · Movement · Equipment · Moveset (+ the resolved button table) · Combat and AI · Rules | per move: weapon, ability, clip, profile, reaction; per status: its definition; per item / projectile: its definition; its brain; its hit reactions | the stage, Test, Preview motion, Combo preview (Motion Lab) | Place; Assign a set / ability / status / brain | Edit beside (a dock): weapon, ability, brain, status, reaction, profile, clip; Used by; Copy / Based on | editing of weapons, abilities, brains, reactions, profiles (they are opened beside, not inlined) |
| **Motion Lab** (the Combo preview renamed and hoisted) | strings; clip keys and events (the clip editor); stances (a new editor over `aim` layers — later); profiles; reactions; a weapon's or ability's hit values — as drafts | the character (Studio), the weapon / ability definition, the set | the game, seek, slow motion, compare | — | Make a copy for this project; Save drafts | entity fields other than performance / tuning |
| **Ability & AI editors** (the two designers, as they are) | ability definitions; brains | what uses them (Library ▸ used by); the Studio | the Motion Lab (an ability played on a default fighter) | Assign to… | Duplicate / New variant / Make a copy for this project | — |
| **Look** (Styles + Engines + FX Lab + geometry presets, one tab with sub-views) | styles, variants, palettes, materials; engine presets; advanced-effect presets; geometry presets | the objects that use a material / a preset (used by) | the canvas; the audition grid; the FX canvas | apply / insert | Bake / Extract; Open in Engines from an object | — |
| **Game** (project settings) | flow, state, screens, input, audio (buses, ambience, music selection), combat defaults, runtime systems | the screens, sounds, music (Library) | screens on the canvas | — | — | scene settings (World) |

