# Progress ledger

A short ledger of what each recent pass left **implemented**, **verified**, **partly verified** and **deferred**.
It does not repeat the records: the plan is `GRAND_ROADMAP.md`, the full status is
`DRAW5_IMPLEMENTATION_STATUS.md`, each pass has its checkpoint. Created on 2026-10-13 (the brief of that day names it;
no such file existed).

How the words are used:

| Word | Means |
|---|---|
| **implemented** | the code and its editor path exist |
| **verified** | driven end to end by a probe through the game's inputs or the editor's controls, or by unit tests, in the pass named |
| **partly verified** | verified by a stand-in for what was asked (a gamepad object for a controller; a bot for a player) - the stand-in is named |
| **deferred** | not built; it has a home in `GRAND_ROADMAP.md` ▸ *Recorded gaps* |

## 2026-10-16 — AN-5 effects and the reaction in the workspace (`ANIMATION_WORKSPACE.md` §12)

The user asked to edit what goes with a move in the workspace (the blade's swipe, the burst on contact) with a start and
an end; to add and remove effects; and to control what the enemy does on contact, including its path. The plan was
approved as recommended: the effects are stored in the move (1a); the swipe stays the crescent, with start, end and
tips (2a); the target's path has handles solved by running again (3a); engine additions A, B and C.

| | State | By |
|---|---|---|
| A: a cue's place (`at` middle / feet, `ahead`, `dy`, `scale`); B: an `at` entry repeated (`every`, `until`); C: a weapon attack's own cues (`fx`), compiled into its states; without them, as before | implemented · **verified** | `an_5.test.js` (9 of 20) |
| the run records the effects the game spawned | implemented · **verified** | `an5_effects.js` lane |
| the effects lane: swipes, cues and bursts from the definitions, laid on the run, seen or not; a swipe's end dragged (the hitbox out longer) | implemented · **verified** | `an5_effects.js` lane, swipe |
| the swipe's tips dragged on the stage (sweep / tilt) | implemented · **verified** | `an5_effects.js` tips |
| the burst on contact set to sparkle and to none: what is spawned at the hit | implemented · **verified** | `an5_effects.js` contact |
| + effect here, *Ahead* typed: spawned 20 px ahead at that moment; Delete removes it | implemented · **verified** | `an5_effects.js` add |
| the target's peak dragged: thrown as high (207.7 for 207.5 px), the attack's launch solved; ↶ | implemented · **verified** | `an5_effects.js` reaction |
| Save; Play in the level: the first cut's burst is the sparkle | **verified** | `an5_effects.js` save |
| the Reaction section shared with the Combo preview (`reactionEditing.js`) | implemented · **verified** shown; its form **not driven** here | `an5_effects.js`, `p7_5` - `p7_16` units |
| a moment dragged into another state, a stream's ends, the distance handle | implemented · **unit-tested only** | `an_5.test.js` |
| AN-1 - AN-4 probes again; the Combo preview probe (its Reaction section moved into the shared code) | **verified**: 61 / 61, 16 / 16, 20 / 20, 14 / 14; 45 / 45 | `an1` - `an4` probes, `p7_5_examples.js` |

### Not done or not verified

The swipe is still a crescent shown while the hitbox is out; a blade trail drawn by the held item's tip is the next step
and was not built. Changed from the plan: there is no `at: 'hit'` place; the target's own answer to a reaction is still
changed only in the Combo preview. A particle burst is not recorded by the run. Not run: the full battery,
`passRegression`. Nobody has used it by hand.

## 2026-10-14 — AN-4 the path of a move drawn by hand (`ANIMATION_WORKSPACE.md` §11)

The ask deferred in AN-3: a jump's one arc made to zigzag. Plan approved by the user as recommended: the op `travel`
takes points (not root motion), the moves `travel` already drives (the built-in jump stays physics), points in px from
the start.

| | State | By |
|---|---|---|
| `travel` passes through `through` points, `smooth` 0 (corners) - 1 (curve), an even speed; without points the old path step for step; facing mirrors; a wall stops it | implemented · **verified** | `an_4.test.js` (11) |
| handles over the trail where a `travel` drove the run; a point put by double-click, dragged, removed by double-click; each a draft and one undo step; Save stores `through` in the ability | implemented · **verified** | `an4_path.js` 14 / 14 |
| the run made again passes the dragged point (2 px); in Play the fighter's launcher passes it (2 px) | **verified** | `an4_path.js` drag, save |
| the *Path* section (fields, + point, Smooth) | implemented · **verified** shown; typing a value and Smooth **not driven** | `an4_path.js` section |
| AN-1 / AN-2 / AN-3 probes again | **verified**: 61 / 61, 16 / 16, 20 / 20 | `an1_editor.js`, `an2_context.js`, `an3_live.js` |

### Not done or not verified

The built-in jump has no path (physics); *Give it a path* not built; a weapon attack has no path; a point that is an
expression, and the launcher's end (`-p.height`), are fixed. Not run: the full battery, `passRegression`. Nobody has
dragged a path by hand.

## 2026-10-14 — AN-3 the Animation Workspace live (`ANIMATION_WORKSPACE.md` §10)

The user's second hands-on look: the Combo view did not play below 100 %; no slashes, sparks or target being hit; a move
of a string could not be auditioned alone; the timing could only be typed (in *Hits and timing…*), not dragged; the
target could not be changed nor its hitbox seen. The workspace is to be the one place, with the same controls for a
clip and a combo.

| | State | By |
|---|---|---|
| in context (In motion, Combos) the stage is the game's own, run to each moment and held there: the actor where the run had it (to the pixel, forward and after a moment back), the target reacting, the game's effects out at a hit | implemented · **verified** | `an3_live.js` live |
| Play at 25 % / 50 % / 100 %: world time runs at 0.25 / 0.5 / ~1 of real time (hit-stops lower the 100 % median) | implemented · **verified** | `an3_live.js` speed |
| hitboxes over the stage: both bodies, the hitbox out in an attack's active phase | implemented · **verified** | `an3_live.js` boxes |
| a move of the string played alone (its step's span) and looped; *Its moves* list with ▶ | implemented · **verified** | `an3_live.js` audition |
| drags on the Combo timeline, each a draft and one undo step: a step's wait (undone), a weapon attack's active phase (yardBlade cut1 0.08 → 0.2 s), a one-shot's length (risingSlash retimed), a key along the string | implemented · **verified** | `an3_live.js` timing; `an_3.test.js` |
| *Against*: another target from the list (the Arena skeleton), the string run against it | implemented · **verified** | `an3_live.js` target |
| *The move*: the hit of the action under the playhead (damage, reach, knockback, launch, hit-stop, reaction) beside the clip editor | implemented · **verified** (shown; an edit through it not driven) | `an3_live.js` move |
| one clip: an attack In motion is live (its effect and hitbox); the clip's end dragged = its length; a phase on the clip's action lane dragged | implemented · **verified** for the length; the phase drag on a clip **not driven** | `an3_live.js` clip |
| AN-2 / AN-1 / Combo preview probes again | **verified**: 16 / 16, 61 / 61 (one check rewritten: the figure is now an actor of the game's own stage - keys still never move it, the world is held), 5 / 5, 45 / 45 | `an2_context.js`, `an1_editor.js`, `cp_clip_from_timeline.js`, `p7_5_examples.js --editor` |

### Not done or not verified

**Deferred with the user's leave:** the path of a move drawn by hand (a zigzag in the air) - an engine capability (a
path the actor follows), recorded in `GRAND_ROADMAP.md` ▸ Recorded gaps ▸ *Animation authoring*. A ray's hitbox is not
drawn. Scrubbing backward rebuilds the stage each display frame (coarser than forward). Nobody has used it by hand.
Historical battery not run.

## 2026-10-14 — AN-2 the Animation Workspace in context (`ANIMATION_WORKSPACE.md` §9)

The user's hands-on look at AN-1: a leg clicked on the stage was not listed; dash, the launcher and a combo could not be
edited the way idle is (the Combo preview's clip editor was below five sections); a jump showed no rise.

| | State | By |
|---|---|---|
| every joint of the figure is a row of the parts table (the ones the clip does not move yet below a line); a joint clicked on the stage selects and shows its row | implemented · **verified** | `an2_context.js` parts |
| the one-shots open in the clip list, each clip with the actions that play it | implemented · **verified** | `an2_context.js` clips |
| *In motion*: the clip held where the game puts it (the game run for a moment through an action string, frame by frame), the view fitted to the path, the trail drawn; off: on the spot | implemented · **verified** for jump and fall | `an2_context.js` motion; `an_2.test.js` |
| *Combos*: a string of the character played by the game, every clip it shows end to end on one timeline (actions, clips, keys, in the air, the target's clips), the figure and the target where the run had them; scrub, frames, keys across clips; the editor edits the clip under the playhead; the string is run again after an edit | implemented · **verified** on the Combat Yard's launcher string | `an2_context.js` combo |
| ways in: the Combo preview's *🎞 Edit its animation* and *The whole string there*; a clip on its timeline opens the clip editor (94fe507) | implemented · **verified** | `an2_context.js`, `cp_clip_from_timeline.js` |
| AN-1's workspace probe, the Combo preview editor probe | **verified** again: 61 / 61, 45 / 45 | `an1_editor.js`, `p7_5_examples.js --editor` |

### Not done or not verified

Nobody has used it by hand. The figure is not turned when the run turns it (a string that faces back). A reaction clip
(hurt, launched …) is shown in motion only in the Combo view, on the target. In *Combos* a key cannot be dragged on the
string's timeline (the editor's own row and the ruler move keys). Historical battery not run.

## 2026-10-14 — AN-1 the Animation Workspace (`CHECKPOINT_AN_1.md`, `ANIMATION_WORKSPACE.md`)

A floating editor pass on the user's brief *Unified Animation Workspace*; **P18 and P19 are deferred by the user until
Draw5 is closer to alpha** (`GRAND_ROADMAP.md`).

| | State | By |
|---|---|---|
| one place for any animated thing (a character, a creature, a prop of the scene, a library asset, a set alone) and its clips, grouped by what chooses them | implemented · **verified** | `an1_editor.js` parts 1 – 3 |
| the clip held on its own stage: the world paused, no controller - keys pressed there do not reach the game | implemented · **verified** | `an1_editor.js` part 1 (real keys; the figure, its picture and the world clock checked) |
| play / pause, ± frame, key before / after, start, speed, loop, time / length / frame; a ruler (click, drag) | implemented · **verified** (drag of the ruler: by reading - the probe clicks it) | `an1_editor.js` |
| keys: a part's value typed, + part, + / − key, copy / paste pose, a key dragged to another moment, the length, the strike mark | implemented · **verified** (typed, + part, a key dragged with the real mouse, the length; + / − key and paste through the Combo preview's probe of the same editor) | `an1_editor.js`; `p7_5_examples.js --editor` 45 / 45 |
| events: moved, added, removed (row and lane); what each does and what answers it | implemented · **verified** (moved, added, removed by the row; the lane's drag by reading) | `an1_editor.js` part 2 |
| the action that plays a clip, laid on its time (states, hit, the performance) | implemented · **verified** for a weapon attack with a profile that aligns its mark | `an1_editor.js` part 2; `an_1.test.js` |
| how a clip is used (states, triggers, profiles, own entries, hit reactions, sequences, rules) | implemented · **verified** | `an_1.test.js`; `an1_editor.js` |
| where an edit goes (this character, this object, the set or its project copy, a profile); drafts, ↶ / ↷, Save as one undo step | implemented · **verified** (character, object, set copy; a profile: offered, not saved in a run) | `an1_editor.js` parts 1 – 3 |
| the edit where the clip is used: the fighter's cut and the sign's sway in Play | implemented · **verified** | `an1_editor.js` parts 2, 3 |
| written to a file, read back, reopened | **verified** | `an1_editor.js` parts 1, 3 |
| the ways in: toolbar, object menu, Studio, a set's card, the Motion Preview, the Combo preview | implemented · **verified** (the Combo preview's link by reading) | `an1_editor.js` |
| a non-figure: the library's Hanging sign and its set | implemented · **verified**; placed by a real click on its Library tile in the browser | `an_1.test.js`; `an1_editor.js` part 3 |
| layers, IK, states, triggers on the timeline; multi-key, curves, onion skin; the Inspector's *why* line; natives as data | **deferred** | Recorded gaps ▸ *Animation authoring* |
| whether it is good to use | **not known** | nobody has used it by hand |

## 2026-10-14 — P17a environmental effects seen from above (`CHECKPOINT_P17A.md`, commit `ce5cfdf`)

**Root cause** (reproduced by `test/probes/envfx_repro.js` before any change): the particle engine and its deposits
took "down" to be +y of the screen. In a scene seen from above that is depth on the ground, so snow, rain and leaves
slid across the map and "landed" on the north edge of each collider (white caps half way up a crate's front, a stripe
across a roof), never on the ground; a new Top-down project did not even say it was seen from above. One cause for
every falling effect; fixed once, in the shared engine and host.

| | State | By |
|---|---|---|
| a height axis for falling / rising particles seen from above (snow, rain, splashes, blown and falling leaves, crickets, beetles); the side view's path untouched | implemented · **verified** | `envfx.test.js` (11); `envfx_run.js` S6, R1, L1, L2 in Kiln Town, a new Top-down project, Larkmere |
| snow as an area deposit on the ground (under the hero's feet) and on tops with a height, at their lifted height, clipped to their outline | implemented · **verified** | `envfx_run.js` S1 – S3; pictures `dist/envfx/run_*_snow.png` |
| what lies on a surface drawn at that surface's depth (under the hero and the trees; a roof's snow under a tree in front of the house) | implemented · **verified** | `envfx_run.js` S4, S5; the pictures |
| open water takes snow | implemented · **unit-tested only** (no level of the four has open water seen from above in view) | `envfx.test.js` |
| a new Top-down project says `dimension.view: topdown` | implemented · **verified** | `envfx_run.js` (newtop) |
| the Almanac Green's seasons, rain rule, lamps, pond (P17) | **verified unchanged** | `p17_run.js` 17 / 17; `envfx_run.js` side-view checks |
| the spawn tier (frog rain, falling apples) seen from above; old files without `environment.effects`; solids without a height as roofs | **deferred** | `GRAND_ROADMAP.md` ▸ Recorded gaps ▸ *P17a left open* |
| how it looks to a person in the game | **not known** | nobody has looked since the fix |

Regression: almanacGreen, driftmark, kilntown (random earlier) identical to base `ba9f31d`. Unit suite 1862 / 0.

## 2026-10-14 — P17 world rules and story authoring, with the animation follow-up (`CHECKPOINT_P17.md`)

### P17 — world rules, conversations, a timeline: **done**

| | State | By |
|---|---|---|
| world rules: a condition of the world, targets by tag / name / id / role, a state and actions when it begins and ends; the last one decides; keep, once | implemented · **verified** | `p17.test.js` (the planner); `p17_run.js` *play* (seasons, rain, dusk, the pond, back in spring) |
| a generated object's state by its generator's parameters (a tree in its season) | implemented · **verified** | `p17.test.js`; *play*; pictures `dist/p17/run_*.png` |
| the event `season`, the action `season`, `env` / `rules` in conditions | implemented · **verified** | the same |
| conversations: lines, answers with conditions, once, actions, entries by the situation, on a screen that holds the game | implemented · **verified** | `p17.test.js`; *play* (the interact button, the menu keys) |
| conversations remembered per scene (kernel remember / recall) | implemented · **not verified**: no probe leaves a scene and comes back | - |
| the sequence step `talk` | implemented · **validated and unit-tested, not run** (the demo begins its conversation by a rule) | `p17.test.js` |
| the timeline: lanes, estimated and measured times, edits (form, edge drag, order, + per lane), run, hold, +1 frame, run to a moment | implemented · **verified** | `p17_editor.js` part 4 (a real drag of an edge) |
| the editors: World rules, Conversations (Read it through, Begun by), States (+ a state per season, a state's parameters) | implemented · **verified** | `p17_editor.js` parts 1 – 3, 6 |
| the demo level, the Almanac Green | implemented · **partly verified**: played by a probe through the game's inputs; its tour 51 / 51 | `p17_run.js`; `passRegression` |
| whether it is good to use and to play | **not known** | the audition of `ALMANAC_GREEN.md` |

### The animation follow-up: **done as scoped** (`ANIMATION_AUTHORING_AUDIT.md` §10)

| | State | By |
|---|---|---|
| a clip shown alone in the Motion Preview (no action), paused, stepped, scrubbed; a set's Library card opens on its clips | implemented · **verified** | `p17_editor.js` part 5 |
| what the figure shows and why (a state and its condition, a one-shot and what decided it, a held moment, no set) | implemented · **verified** for a state, a one-shot and a held moment; *no set* by reading only | the same |
| pause and +1 frame in the Motion Preview | implemented · **verified** | the same |
| keys edited on the clip stage; the why line in the Inspector | **deferred** | Recorded gaps ▸ *Animation authoring* |

### Not run in this pass

The historical battery (`--regress`, `kernel_systems`, the pace sweep) and every earlier probe. Run: the whole Node unit
suite, the two P17 probes, and `passRegression` (driftmark, glimmerreef, kilntown - its random pick - and the new level).

## 2026-10-13 — P16 inventory and items · the animation authoring audit · benchmark 4 (`CHECKPOINT_P16.md`)

### P16 — inventory and items: **done**

| | State | By |
|---|---|---|
| items as entity definitions with stack, category, use, icon, keep | implemented · **verified** | `p16.test.js`; `bench4_run.js` *play* |
| a bag per entity (opt-in), places and stacks, capacity | implemented · **verified** | the same: a full stack, a full bag, a recipe refused for room |
| collect, use, equip from the bag, drop and take again, transfer, stash | implemented · **verified** | *play* |
| recipes as data: needs, tools kept, a station, a condition; five refusals | implemented · **verified** (`when`: unit tests only - no recipe of the game uses it) | `p16.test.js`; *play* |
| the bag travels (a door, a saved game); a chest and what was dropped are remembered | implemented · **verified**: through doors and a revisit (*play*); a saved game written (1.2 KB, the bag's stacks in it), what was carried then thrown away, the game loaded - the bag, the food, the emptied chest, the driftwood taken and the hide dropped on the shore are all as saved (*save*) | *play*, *save* |
| feedback lines, the event `item` | implemented · **verified** | *play* reads the lines |
| the three built-in screens, row kinds, a screen's subject | implemented · **verified** | `p16.test.js`; *play*, *keyboard*, *pad* |
| the editor: Game ▸ Items and recipes; the Studio's Item and Bag; the recipe form; the action Item | implemented · **verified** | `p16_editor.js` (30 of 30) |
| a game without an inventory is unaffected | **verified** on three levels | `passRegression`: frame-identical |
| shops, weight, durability, a slot grid, a two-sided container screen, timed crafting, pictures in menus | **deferred** | Recorded gaps ▸ *P16 left open* |

### The hex map (made for benchmark 4): **done as far as the benchmark needs**

| | State | By |
|---|---|---|
| cells as objects, a map as a scene, the way, the neighbour a direction means, what a cell needs | implemented · **verified** | `p16.test.js`; *play* |
| choose by direction and by pointer, travel, enter, discovery, what the scene remembers | implemented · **verified** | *play*, *keyboard* (real clicks), *pad* |
| the editor: Hex map, Hex cell, a cell beside it | implemented · **verified** | `p16_editor.js` |
| flat-topped cells | implemented · **verified by unit tests only** (the game's map is pointy) | `p16.test.js` |
| the rule action `hex` (reveal, goto, block, open) | implemented · **not verified** (no rule of the game uses it) | - |
| anything else on the map (rivals, time, generated regions, a brush) | **deferred** | Recorded gaps ▸ *Hex map left open* |

### Controller and keyboard / mouse parity

| | State | By |
|---|---|---|
| keyboard through the whole loop | **verified** with key events of the page (default codes) | *keyboard* |
| mouse: a cell chosen and travelled to, a menu line chosen | **verified** with real clicks of the window | *keyboard* |
| a pad through the whole loop: stick, d-pad, A / B / X / Y; every menu | **partly verified**: a standard-mapping gamepad object handed to the game by the Gamepad API. **No physical controller was used.** | *pad* |
| pause on a pad; a pad in the rebinding screen; default pad buttons for *interact* and the bag | **deferred** | Recorded gaps |

### Benchmark 4 — Driftmark: **built; played by a bot, not by a person**

| | State | By |
|---|---|---|
| the whole game, shore to beacon | **partly verified**: an input bot | *play*, 47 checks |
| seven of eight places | **verified** as used by the bot | the Kelp Shallows were **not entered** |
| the loss | **verified** (hunger) | *idle* |
| hostile wildlife and the sling | **measured once**, not a check of the probe | `CHECKPOINT_P16.md` §6 note in the report |
| a watchable demonstration | implemented · **verified** | the demo `tour`, 47 of 47 |
| whether it is good to play | **not known** | the audition of `BENCHMARK_4_DRIFTMARK.md` §9 |

### The animation authoring audit: **written; nothing redesigned**

| | State |
|---|---|
| the audit, its matrix, its nine recommendations | `ANIMATION_AUTHORING_AUDIT.md` |
| claims marked **[ran]** | exercised in this pass |
| the representative workflow (the clip editor through its controls) | **verified** on this working tree: `p7_5_examples.js --editor` 45 of 45, `bench2_editor.js --editor` 39 of 39 |
| every other workflow of the audit | **read in the source**, not run (the testing limit) - each says so |
| its recommendations | **deferred**: Recorded gaps ▸ *Animation authoring* |

### Not run in this pass

The historical battery (`--regress`, `kernel_systems`, the pace sweep, earlier probes other than the two above).
