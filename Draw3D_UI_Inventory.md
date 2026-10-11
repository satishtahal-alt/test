# Draw3D — inventory of the editor's interface

Written 2026-10-11, from the code as it stands on branch `m2-stable-terrain` after the
consolidated pass of that day. It is a reference to review, rename and regroup from.
**Nothing was renamed to make it**: every name below is the name on the screen now.

**How to read it.** Each entry gives the exact name, where it is, what it really does
(read from the script that does it, named in brackets), what it changes and how far,
what it needs first, and what it overlaps or hides. *Scope* is the important column:
**view** (changes nothing in the world), **one thing**, **one maker and everything it
made**, **a design and every thing of it**, **the world**, **this machine** (a
preference, never saved with a world), or **the game being played**.

**How sure it is.** Everything here was read from the scripts in `editor/godot`
(`desk.gd`, `main.gd`, `makers.gd`, `designs.gd`, `marks.gd`, `handles.gd`,
`terrain.gd`, `palette.gd`, `inspect.gd`, `kindview.gd`, `textview.gd`, `hubview.gd`,
`feel.gd`, `newworld.gd`, `checklist.gd`, `tint.gd`, `controls.gd`). Where a control's
behaviour was taken from its label, its tooltip or its script's own description
rather than traced through the code that runs, the entry says **(not traced)**. What
scripted checks exercise is noted in `PROGRESS.md`, not here. Nothing here was tried
by hand.

**The words the editor uses** (as the user fixed them, 2026-10-11): a **thing** is one
object; its **sort** is what kind it is (a rock); a **design** is what a thing is,
apart from where it stands (rock no. 2, or a kind of your own); a **maker** is what
creates things and controls their making (a wood); its **arrangement** is the things
it made. The code and some labels also say **kind** (a design written as text in this
world), **mesh** (a shape of corners and faces a kind is made of), **node** and
**generator** (a maker, in the Inspector), **design/kind/sort** somewhat
interchangeably: see section 12.

---

## 1. The layout

- **The bar** across the top (`desk.gd`). Two rows while editing: the menus and
  buttons; under them a line that says what is picked or what tool is in use, with
  buttons for it (the *context row*). While playing, a different first row.
- **Windows**: every panel is a floating window. Drag its title bar to move, its
  corner to resize, **-** folds it, **x** puts it away. The **Window** menu lists
  them all. Where each is, its size and whether it is folded are kept on this
  machine (`user://layout.cfg`). They do not dock.
- **The view**: the world, behind everything. The left button picks, or does what
  the tool in use does.
- **Notices**: a box at the bottom middle says what just happened; it fades.
- **Going to Play** puts the editing windows away and brings them back afterwards.

---

## 2. The bar while editing

| Name | What it does | Scope | Notes |
|---|---|---|---|
| **World v** | menu, section 3 | | |
| **Environment v** | menu, section 4 | | |
| **Things v** | menu, section 5 | | |
| **Characters v** | menu, section 6 | | |
| **Window v** | menu, section 7 | | |
| **Undo** | Back one step (Ctrl+Z). Greyed when there is none | the world | Also in World menu and in the Place window: three places |
| **Redo** | Forward one step (Ctrl+Y) | the world | Same three places |
| **Play (Tab)** / **Play as NAME (Tab)** | Starts walking in the world from the green flag (where a game starts), or with no start put down, from where the view looks. Nothing edits the world until you come back | the game being played | The label changes when the walker is a kind of yours |
| **Checklist** | Opens the list of what to try by hand | this machine | |
| **Keys** | Opens the window of every key and mouse action | view | |

### The context row (second row)

It says one of: what the engine is working on; what is being shaped; the maker tool
in use; the land tool in use; what is being placed; what is picked; the maker
selected; or "Nothing picked…". Its buttons appear only when they apply.

| Button | Appears when | What it does | Scope |
|---|---|---|---|
| **Stop it (Esc)** | the engine is working | Stops the work; nothing of it is kept | the world |
| **Its text (F6)** | a mesh is being shaped | Opens Kinds and meshes on that mesh's text | view |
| **Done (Esc)** | a mesh is being shaped | Puts the corners away; what was dragged stays | view |
| **No tool (Esc)** | a land tool is chosen | The left button picks again | view |
| **End the line (Enter)** | a line is being drawn | The places clicked become a line (a road) | the world |
| **Cancel (Esc)** | any maker tool is in use (placing, painting, adding, duplicating, moving) | Ends the tool; nothing more is done | view |
| **Just pick (Esc)** | a thing is armed to be placed | The left button picks again | view |
| **Inspect (F7)** | something picked or a maker selected, and the Inspector is shut | Opens the Inspector | view |
| **Shape by hand** | one thing is picked | Opens the Inspector and takes hold of the thing's corners. Something generated (a roof) is first written out as a mesh | **a design and every thing of it**, or one thing: the strip says which | Overlaps Makers > "Shape this one only…" and "Shape its design…", which say the scope before you press. This button does not |
| **Play as this** | the thing picked is of a kind with a person's bones | The walker becomes that kind, and play starts beside it | the world (which kind the walker is) |
| **Its kind (F6)** | the thing picked is of a kind of yours | Opens Kinds and meshes on that kind's text | view |
| **Duplicate this one… (Ctrl+D)** | one thing is picked | A frame follows the pointer; a click makes another, **standing by itself**; Esc gives up | adds one thing, of no arrangement |
| **Move this one…** | one thing is picked | A click says where it is to stand | one thing |
| **View its maker in Makers** | one thing is picked, Makers is shut | Opens Makers on its maker; the thing stays picked | view |
| **Select its maker** | one thing of several is picked | The maker becomes what is selected instead of the thing | view (selection) |
| **Makers (Ctrl+M)** | a maker is selected, Makers is shut | Opens Makers on it | view |
| **Look at it** | a maker is selected | The view goes to the maker | view |
| **Take away this one (Delete)** | one thing is picked | Takes away that one thing; its maker keeps it away and can put it back. A thing added by hand is simply no longer added | one thing |
| **Let go** | anything is picked or selected | Nothing is chosen | view |

**Usability notes.** The row can wrap to three lines when one thing is picked (eight
buttons). "Shape by hand", "Its kind", "Inspect" and Makers' shaping buttons are four
ways into nearly the same place. "Select its maker" and "View its maker in Makers"
differ only in whether the thing stays picked.

---

## 3. World menu

| Entry | Key | What it does | Scope | Notes |
|---|---|---|---|---|
| *The world open: FILE* | | A heading: which file, or "not written to a file yet" | | |
| **New world…** | Ctrl+N | Opens the New world window (section 8.11) | | |
| **Makers: what this world is built of** | Ctrl+M | Opens Makers (8.1) | | Makers is under **World**, though most of what it does is about things and growth. Hard to guess |
| **Open a world…** | Ctrl+O | A file dialog; replaces the world in the editor. Asks first if there are unsaved changes | the world | |
| **Save the world** | Ctrl+S | Writes the world to its file; with no file yet, asks where | the world's file | **Writes over the file without asking.** The user's world of 2026-10-09 was lost this way once |
| **Save the world as…** | Ctrl+Shift+S | Asks for a file and writes there; that becomes the world's file | the world's file | |
| *N changes not in any file* | | A heading | | |
| **Undo** / **Redo** | Ctrl+Z / Ctrl+Y | As on the bar | the world | Duplicate of the bar |
| **What needs a look** | I | Lists changes by hand that no longer fit what a maker makes, and what the last build had to say | view | The name does not say it is about hand edits that lost their place. Rarely needed, easily missed |
| **Engine status** | | A window of numbers: things, cells, timings | view | Developer information |

Not in the menu: **F5 / F9** save and load a *game* (what has happened while
playing), which is not the world. The two kinds of saving are easy to confuse; F5 and
F9 are only said in Keys and in the play row's Game menu.

---

## 4. Environment menu

| Entry | Key | What it does | Scope | Notes |
|---|---|---|---|---|
| **Shape and paint the land, and where things may grow** | F8 | Opens "The land, by hand" (8.4) | | This is where terrain sculpting lives. Its window title is "The land, by hand", its key F8, its menu entry this sentence: three names for one tool |
| **Add land…** | | Adds an island. Greyed if the world has one | the world | |
| **Draw a line: a road…** | | Click places on the land; Enter ends it. The land levels the ground along it | the world | A "line" is a road only because the land is given it; the same line is what a street follows |
| **Add woods, about one place…** | | A click on the land adds a wood reaching 40 m; press and drag sets the reach | the world | 40 m is wide: on a small plateau most trees stand below it. Said in the notice after adding |
| **Add a meadow, about one place…** | | The same, reaching 30 m | the world | |
| **What grows: trees (how many, which, how near the road)…** | | Opens the **Inspector** on the maker named "woods" | view | Only finds a maker called exactly "woods". Since 2026-10-11 the count is also in Makers > "Where it makes its things", so this entry now duplicates that |
| **What grows: grass, flowers, bushes, rocks (how many)…** | | The same for the maker named "meadow" | view | Same limits |
| *Style: NAME shown at HH:00 weather: NAME* | | A heading | | |
| **The next style** | F2 | Gives the world the next style in `styles.txt`; after the last, none | the world | |
| **Style: summer / autumn / highland / evergreen / none** | | Gives the world that style by name (new 2026-10-11) | the world: colours, light, and for *evergreen* what woods and meadows are made of | A style is an undoable edit of the world. *evergreen* changes shapes, the other three only colours and numbers: nothing in the menu says which |
| **An hour on (a look: time runs while playing)** | T | Shows the world an hour later | view only, while editing | |
| **The next weather** | N | The next weather written in `weather.txt` | the world's weather fact **(not traced)** | |

**Usability notes.** "Environment" holds three unlike things: shaping the ground,
adding makers of four kinds, and the look of the world. Adding woods is here, but
changing a wood afterwards is in Makers (under World) or the Inspector (under
Things). The land's *paint* (rock, sand, grass) and a maker's *painted area* (where a
wood grows, new) are both called painting and are in different windows.

---

## 5. Things menu

| Entry | Key | What it does | Scope | Notes |
|---|---|---|---|---|
| *Picked: …* | | A heading: what is picked, in the user's words | | |
| **Place things, and move what is picked** | | Opens "Place and move" (8.3) | | |
| **Designs: everything there is to place, or to give a maker** | | Opens Designs (8.2) | | |
| **Inspector: what is picked, and what made it** | F7 | Opens the Inspector (8.5) | | |
| **Kinds and meshes: what things are made of** | F6 | Opens Kinds and meshes (8.6) | | A kind **is** a design of your own; the two windows show the same things two ways |
| **Put down a house…** | | A click puts one house down by itself | the world | |
| **Add a street of houses, along a line…** | | Draws a line if none is selected in Makers, then adds houses along it | the world | |
| **Duplicate the one picked…** | Ctrl+D | As the context row's | adds one thing | |
| **Move the one picked…** | | As the context row's | one thing | |
| **Shape what is picked by hand** | | As the context row's "Shape by hand" | a design, or one thing | Does not say the scope before it acts |
| **Take away the one thing picked (its maker stays)** | Delete | As the context row's | one thing | |
| **Detach what is picked from what made it** | X | Makes the thing an authored thing no maker makes any more | one thing | **Unclear and hard to reverse in practice.** Nothing else in the interface explains detaching, and it overlaps Duplicate + Take away. **(not traced beyond the command it sends)** |

---

## 6. Characters menu

| Entry | Key | What it does | Scope |
|---|---|---|---|
| *The walker: …* | | A heading: which kind the walker is | |
| **Kinds: someone to walk as, creatures to put down** | F6 | The same Kinds and meshes window | |
| **Who can do what, with what** | F4 | Opens the hub (8.8) | view |
| **Moves, poses and acts: what is written** | F10 | Opens "What is written" (8.7) | |
| **Movement numbers: speeds, the jump, what swings** | F1 | Opens Movement numbers (8.9) | this machine's `feel.cfg` |
| *A game starts: …* | | A heading | |
| **Put down where a game starts…** / **Move where a game starts…** | | A click puts the green flag | the world |
| **Play from where the view looks** | | Plays without using the flag | the game being played |

---

## 7. Window menu

| Entry | Key | What it does | Scope |
|---|---|---|---|
| one entry for each window (section 8), "(open)" before those shown | their keys | Opens it, or puts it away | this machine |
| **Middle drag turns the view about the place under the pointer** | P | On: the view orbits the ground under the pointer. Off: the middle of the screen | this machine, until the editor closes |
| *Selection colours: a maker X, one thing Y, what is new Z* | | A heading (new 2026-10-11) | |
| **Colours: Sky blue and gold / Magenta and yellow / Red and white / Purple and orange** | | What everything selected is marked in: a maker, one thing, the rest of its arrangement, what is about to be put down, handles. Kept for the next start | this machine (`user://editor.cfg`) |
| **Close all panels** | | Puts every window away | this machine |
| **Put every window back where it starts** | | Forgets where windows were put | this machine |

**Usability note.** A preference (colours, orbit) sits in the list of windows because
there is no Preferences place. The orbit choice is not kept between starts; the
colours are.

---

## 8. The windows

### 8.1 Makers: what this world is built of (Ctrl+M) — `makers.gd`

The most loaded window. Top to bottom:

| Part | Control | What it does | Scope | Notes |
|---|---|---|---|---|
| top | a list of kinds of maker, under headings (Land and lines; Buildings; Growing things; Animals and boats; A game's start and its puzzles) | Chooses what **Add…** adds | view | |
| top | **Add…** | Adds a maker of that kind: at once, or by a click on the land, or after a line is drawn | the world | |
| top | the description under it | What the maker is, what it reads, how it is added | | |
| list | every maker in the world, by name, with what it made and what was changed by hand | Click selects it | view (selection) | Things put down one at a time are makers too ("One thing put down"), so a world with many boats has many rows |

With a maker selected (or one thing picked), under the list:

**The sentence in colour.** "CHOSEN: THE MAKER 'woods' (Woods)…" or "PICKED IN THE
VIEW: ONE ROCK (yellow frame)… It belongs to the arrangement of N things made by the
maker…". Its colour is the colour that thing is marked in.

**"This one SORT"** — only with one thing picked; everything here is of that one thing.

| Control | What it does | Scope | Notes |
|---|---|---|---|
| **Take away this one (Delete)** | Removes that thing; its maker keeps it away | one thing | |
| **Duplicate this one… (Ctrl+D)** | Another, by a click, standing by itself | adds one thing, of no arrangement | |
| **Move this one…** | A click says where | one thing | |
| **Put this one back as its maker makes it** | Takes back everything done to it by hand | one thing | Only shown if something was |
| **its design** (list) | Makes **this one thing** another design, where it stands (new) | one thing | "as its maker makes it" appears once it was changed |
| the **no. N** list beside it | Which of that design's sorts | one thing | |
| **its size (×)** | How big it is beside its design: 1 as designed (new) | one thing | Not for a house, nor for what has bones |
| *Its design: X — …* | Says whose the design is and how far shaping it would go (new) | | |
| **Shape this one only…** | Gives this thing a design of its own (a copy) and takes hold of its corners (new) | one thing | Makes a new design named `X-own` each time it has to copy: these collect under "Yours" in Designs |
| **Shape its design 'X' (all N of it)…** | Takes hold of the design's corners on this thing; every thing of the design follows (new) | **a design and every thing of it** | Only for a design of yours that more than one thing is of |
| **Add another SORT to this arrangement…** | Each click puts one more down as one of the maker's own things; Esc ends (new) | adds things **to the arrangement** | Deliberately not Duplicate |
| *This SORT belongs to the … arrangement made by the maker …* | Says the arrangement, the maker, the design | | |
| **Select its maker ('NAME')** | Selects the maker in place of the thing | view | |

Then a line: "ITS MAKER, 'NAME', AND ITS ARRANGEMENT. Everything below is of the
entire maker…", and the maker's parts, each of which folds:

**"Settings and placement"** (for one thing put down: "Where it stands, and on what")

| Control | What it does | Scope |
|---|---|---|
| **name** | Renames the maker (Enter sets it) | one maker |
| **stands at** east / south | Moves the whole maker, in metres | one maker and everything it made |
| **turned (°)** | Turns the whole maker | one maker and everything it made |
| **reach (m)** | How far it reaches; 0 is the whole land. Since 2026-10-11 it keeps its thickness: the count changes with it, and says so | one maker and everything it made |
| **rests on** (one thing put down) | As its sort does / the ground / the water | one thing |
| **Its numbers (Inspector)** | Opens the Inspector on the maker's parameters | view |
| **Look at it** | The view goes to it | view |
| **Move the whole maker…** / **Move it…** | A click says where its middle is; press and drag also sets the reach | one maker and everything it made |

**"Where it makes its things: …"** — new 2026-10-11; a wood or a meadow only.

| Control | What it does | Scope | Notes |
|---|---|---|---|
| the sentence | Says which of three ways holds: everywhere in its reach; on its own level; only where painted | | |
| **how many** | How many it tries to make (some find no ground and are left out) | one maker | The same number as "count" in the Inspector: two places |
| **keep to the level of its middle, within (m)** (box and number) | Makes nothing on ground higher or lower than that from where its middle stands | one maker | The answer to "trees off my plateau" that needs no painting |
| **Paint where…** | A brush on the land: it makes its things where painted and nowhere else | one maker | Once painted, the reach is ignored. Said, but surprising |
| **Rub out…** | The same brush, removing | one maker | Only when something is painted |
| **Nothing painted: fill its reach again** | Clears the paint | one maker | |
| **brush (m wide)** | The brush's width; **[** and **]** while painting | this machine, until the editor closes | |

**"What it is made of: …"** — a wood, a meadow, a street.

| Control | What it does | Scope | Notes |
|---|---|---|---|
| the rows | Each design given: which sort, how many **parts**, **take out** | one maker and everything it made | "Parts" are shares, not percentages |
| **Add a design…** (list) | Adds a design to what it is made of | one maker | |
| **All designs…** | Opens Designs | view | |
| **Back to the usual** | Gives it no designs: it makes what it always did (or what the style says) | one maker | In the *evergreen* style "the usual" is conifers: the label does not say so |

A **street given a design has no houses** (it is one or the other): a known gap, and a
surprise.

**"Its arrangement: the N things it made"**

| Control | What it does | Scope |
|---|---|---|
| the sentence | How many of each sort | |
| **Put back the N taken away** / moved / turned / otherwise changed | Takes those changes by hand back, all of that kind | one maker and the things concerned |
| **Give the N their maker's design back** / **Make the N resized their own size again** / **Take out the N added by hand** (new) | The same for what the maker keeps of single things | one maker and the things concerned |
| **Add one thing by hand…** (list of designs, new) | The next clicks each put one down as the maker's own | adds things to the arrangement |

**"What it reads"** — which land, which line. Lists to connect another. **Read by:** who
reads this one. *Scope: one maker.* The words "reads" and "read by" are the engine's;
a user would say "follows this road", "stands on this land".

**"Duplicate or remove the entire maker"**

| Control | What it does | Scope |
|---|---|---|
| **Duplicate the entire maker…** | A ring follows the pointer; a click puts another maker down with all it makes (a line is copied 8 m off at once) | adds a maker |
| **Remove the entire maker and its generated contents…** | Asks "ARE YOU SURE?", then removes the maker and everything it made | one maker and everything it made |

**In the view, with a maker selected:** its ring (or the outline of what was painted, or
with no reach a square about all it made), a stroke over each thing it made, a **box**
in its middle (drag: moves the whole maker) and a **ball** on its ring (drag: sets the
reach, measured on the maker's own level).

**Usability notes.** One window does five jobs: adding makers, listing them, editing
one thing, editing a maker, removing a maker. Its name is the engine's word. A user
looking for "change this tree" or "make the forest smaller" has to know that both are
under World > Makers.

### 8.2 Designs: what things are — `designs.gd`

| Control | What it does | Scope | Notes |
|---|---|---|---|
| the list, under **Yours** and **Built in** | Every design: how many sorts, how many stand, which makers were given it | view | No pictures: names only |
| **which of its sort** | any, or no. N | view | |
| **Place one…** | The next click on the land puts one down by itself | adds one thing | Same as the Place window |
| **Give it to 'MAKER'** | The maker selected in Makers makes its things of this too | one maker and everything it made | Greyed unless a wood, meadow or street is selected in *another window* |
| **Make it mine** (with a name) | Writes a **copy** of a built-in design as a design of this world | adds a design; **changes nothing that stands** | The name suggested ownership of something in the world. It now says it made a copy nothing is of |
| **Shape it in the view…** (yours; new) | Takes hold of its corners on a thing of it; with none standing, the next click puts one down first | **a design and every thing of it** | |
| **Change its text (Kinds and meshes)** (yours) | Opens its text | a design and every thing of it | Was "Change it" |

### 8.3 Place and move — `palette.gd`

| Control | What it does | Scope | Notes |
|---|---|---|---|
| the top button ("put down: rock (click to choose)") | Opens a grid of everything placeable | view | Its own label is its state: easy to miss that it is a button |
| a thing in the grid | Arms it: each left click on the land puts one down | adds things | Keys 1–9 put the first nine down under the pointer without arming |
| **just pick** | Disarms | view | |
| **new kind… F6** | Opens Kinds and meshes | | |
| **all designs…** | Opens Designs | | |
| *picked: …* | What is picked | | |
| **turn left / turn right** | 15° ( [ and ] ) | one thing | |
| **away / nearer / left / right** | Moves it half a metre, as the view shows | one thing | |
| **take away**, **undo**, **redo** | As elsewhere | | Third copy of undo and redo |

Overlaps Designs almost wholly (Designs also chooses which sort). Two windows to put
a thing down.

### 8.4 The land, by hand (F8) — `terrain.gd`

Reached by **Environment > Shape and paint the land, and where things may grow**.

| Group | Tool (button) | What it does | Scope |
|---|---|---|---|
| shape | **raise**, **lower**, **smooth**, **flatten** | Brushes dragged on the land. Flatten levels to the height where the drag began | the land's own layer, where dragged |
| surface | **paint** | Changes the surface (rock, grass, sand, dirt, or "as generated"), not the shape. Not under water | the land's layer |
| what grows | **clear**, **let grow** | Says nothing grows under the brush, or that it does again. Plants nothing | the land's layer; every wood and meadow obeys it |
| stamp | **cliff**, **ledge** | A click raises a sheer block, or a level shelf | the land's layer |
| by hand | **restore** | Takes away what was done by hand (shape, paint, or both) | the land's layer |

| Control | What it does |
|---|---|
| the numbers, each a slider, a step either side and a field: **radius, m**; **m each dab**; **how much**; **high, m**; **back, m**; **above, m**; **turned, °**; **soft rim, m** | Only those the tool uses are shown. Keys: **[ ]** radius, **- =** strength or softness, **, .** turn a stamp |
| **made** (list) | What paint makes the surface |
| **what grows goes by the surface painted** (box) | On: rock, sand and dirt stop growth. Off: painted for looks alone |
| **takes away** (list) | What restore removes |
| **restore all of the land** | Removes everything done to the land by hand, everywhere (undoable) |
| **middle drag turns the view about the pointer (P)** | The orbit choice, again |
| the lines under the tools | What the tool is for; what a stroke did to what grows; what is kept under the pointer |

**Usability notes.** "clear" and "let grow" control where *every* wood and meadow may
grow; Makers' new "Paint where…" controls where *one* maker grows. Both are brushes,
in different windows, and nothing links them. "restore all of the land" is a large
action in a small button. The window's name gives no hint of "terrain" or "sculpt".

### 8.5 Inspector (F7) — `inspect.gd`

| Control | What it does | Scope | Notes |
|---|---|---|---|
| the list at the top | Every maker ("node") in the world; choosing one shows its numbers | view | The same list as Makers, by another name |
| what is picked, in words | Its sort, its maker, its path under the maker, what was changed by hand | | Engine words: "generator", "path", "live" |
| **east (m)**, **south (m)**, **turned (°)** | Where the picked thing stands | one thing | |
| the maker's **parameters**, each a number or a text | Changes it and makes the maker's things again | one maker and everything it made | Includes raw text parameters: `of`, and since 2026-10-11 `each`, `added`, `level`. Editing `each` or `added` by hand here is possible and easy to get wrong |
| **?** beside a parameter | Says what the change would do before it is done | view | Valuable and hard to notice |
| **freeze** / **thaw** | Keeps what the maker made exactly as it is | one maker | |
| **another seed** | Lays the maker's things out anew | one maker and everything it made | **(not traced)** |
| a line's places (for a road) | Drag one on the world; add or remove a place | one line | |
| **edit this kind (F6)** | Opens the thing's kind | | |
| **edit its shape: write it out as a mesh** / **open its mesh 'X' (F6)** | For a roof, wall, window, door: makes its shape a mesh of yours | one thing | |
| **shape it by hand: take hold of its corners** | The shaping strip | a design and every thing of it, or one thing | |
| **the generated shape back** | Undoes that | one thing | |

Overlaps Makers for selecting a maker, moving a thing and changing a count.

### 8.6 Kinds and meshes (F6) — `kindview.gd`

A list on the left (kinds, then meshes in blue), the text of the one chosen on the
right, what it could not understand under it.

| Control | What it does | Scope |
|---|---|---|
| **New** | A kind to start from | adds a design |
| **Duplicate** | A kind "like" the chosen one (it stays linked to it) | adds a design |
| **Delete** | Removes the kind, unless something is of it | a design |
| **Put one down** | The next click puts one down | adds one thing |
| **Walk as this** | The walker becomes this kind | the world |
| **Add the examples** | Writes Wren and two cats into this world | adds designs |
| **New mesh** | A block to start from | adds a mesh |
| **A kind of this mesh** | A kind whose one part is the mesh | adds a design |
| **Shape it in the view** | Takes hold of the mesh on a thing made of it | a design and every thing of it |
| **Apply (Ctrl+Enter)** | Writes the text into the world | a design and every thing of it |

**Usability notes.** This is a text editor for designs. "Kind" and "design" are the
same thing under two names in two windows. "Duplicate" here links the copy to the
original ("like"), unlike every other duplicate in the editor. Which things a mesh is
the shape of is said only in a status line.

### 8.7 What is written: moves, poses, acts (F10) — `textview.gd`

A list of texts (moves, poses, acts, eyes, weather), the text, **Apply (Ctrl+Enter)**,
**As the world has it** (drops what was typed). *Scope: the world; applying also
writes the file of that name beside the scripts.* **(not traced)**

### 8.8 Who can do what, with what (F4) — `hubview.gd`

Read-only: everyone with a body, every kind of thing, every act; what is said of the
one chosen; underlined words lead to where it is written. **open FILE** opens the text
file in the machine's own editor; **write it here (F10)** opens 8.7 at that entry.
*Scope: view.* **(not traced)**

### 8.9 Movement numbers (F1) — `feel.gd`

A readout of what the body is doing, then a slider, steps and a field for each number
the walker moves by. **all back to how they began**. *Scope: this machine's
`feel.cfg`, read by every world.* Available editing and playing.

### 8.10 Shape by hand — `handles.gd`

Opens itself when a shape is taken hold of (from Makers, Designs, the Inspector, Kinds
and meshes, or the context row).

| Control | What it does | Notes |
|---|---|---|
| first line (new) | **SHAPING ONE THING ONLY…**, or **SHAPING THE DESIGN 'X': EVERY thing of it changes, all N…**, or **SHAPING THIS ONE THING** (a generated roof) | The scope of every drag |
| second line | The mesh, its corners and faces, its size | |
| in the view | The shape's edges in the colour of what is picked; every corner a dot (up to 1,500); the one under the pointer marked | |
| left drag on a dot | Moves that corner (Shift: up and down only) | |
| left drag on a face | Moves it the way it looks (Ctrl: pushed out on new corners) | |
| **reach** (slider) | Corners within that distance go with the one dragged (**[ ]**) | |
| **a drag goes** free / X / Y / Z / XZ / XY / YZ | Keeps a drag to a line or a plane (keys X, Y, Z) | Y is up |
| **its text (F6)** | The mesh's lines | |
| **done (Esc)** | Puts the corners away | |

Each drag is one line of text in the mesh and one step of undo.

### 8.11 New world (Ctrl+N) — `newworld.gd`

**What it begins as** (The playground island / An empty island / Nothing at all),
**seed**, **hills (m)**, **size (patches)**, **Make the new world…** which asks once
more ("Yes: replace the world in the editor" / "No, keep this world"). *Scope: the
world in the editor; no file is touched.*

### 8.12 Checklist, Engine status, Keys and mouse

- **Checklist**: boxes to tick, kept on this machine; **untick all**.
- **Engine status**: numbers. Developer information.
- **Keys and mouse**: the whole list of keys.

---

## 9. The bar while playing

| Name | Key | What it does |
|---|---|---|
| **Stop: back to editing** | Tab or Esc | The world stops; what happened in it is undone |
| **Movement** | F1 | Movement numbers |
| **Controls** | H | The card of controls |
| **Game v** | | Save the game (F5), Load the saved game (F9), An hour on (T), The next weather (N), The next eye (V), Keyboard or controller shown (F3) |
| **Who can do what** | F4 | The hub |
| **Window v** | | As section 7 |
| **Checklist** | | |

---

## 10. Mouse and keys in the view, while editing

| Input | What it does |
|---|---|
| left click | Picks a thing; with a thing armed, puts one down; with a tool, the tool's |
| left drag on a maker's box or ball | Moves the maker / sets its reach |
| right drag on what is picked | Moves it (Esc drops it) |
| right drag elsewhere | Moves the view |
| Shift + right click | Puts what is picked there |
| middle drag | Turns the view |
| wheel | Nearer, farther |
| W A S D, Q E, R F | Move, turn and tip the view |
| [ ] | Turn what is picked; or the brush / reach of the tool in use |
| Delete | Take away the one thing picked |
| X | Detach what is picked (or, shaping, keep a drag to X) |
| 1 … 9 | Put down a rock, tree, bush, barrel, crate, sacks, fence, cart, lamp |
| O | Put down a rock (an old key) |
| Ctrl+D | Duplicate what is picked |
| Ctrl+Z / Ctrl+Y | Undo, redo |
| Ctrl+N, Ctrl+O, Ctrl+S, Ctrl+Shift+S | New, open, save, save as |
| Ctrl+M | Makers |
| F1, F4, F6, F7, F8, F10 | Movement numbers, hub, kinds, inspector, land, texts |
| F2, T, N | Next style, an hour on, next weather |
| I | What needs a look |
| P | Orbit about the pointer or the middle |
| Tab | Play |
| Esc | Ends a tool, a drag, or the engine's work |

**Keys that mean two things:** **[ ]** (turn a thing / brush radius / reach), **X**
(detach / lock a drag to X), **N** (next weather; Ctrl+N new world), **- =**, **, .**
only inside the land tool.

---

## 11. Where one task is spread over several places

| Task | Where it can be done |
|---|---|
| Put one thing down | Place and move; Designs > Place one…; keys 1–9; Kinds and meshes > Put one down; Things > Put down a house |
| Select a maker | Makers list; Inspector list; context row "Select its maker"; Makers "Select its maker" |
| Move one thing | right drag; Shift + right click; "Move this one…" (bar, menu, Makers); Place and move arrows; Inspector east/south |
| Change how many trees | Makers > Where it makes its things > how many; Inspector > count; Environment > What grows: trees |
| Say where things grow | The land > clear / let grow (all makers); Makers > keep to the level; Makers > Paint where… (one maker); a maker's reach (ring, number) |
| Shape something | context row "Shape by hand"; Things menu; Makers "Shape this one only…" and "Shape its design…"; Designs "Shape it in the view…"; Kinds and meshes "Shape it in the view"; Inspector "shape it by hand" |
| Undo | bar; World menu; Place and move; Ctrl+Z |
| Edit a design | Designs; Kinds and meshes; Inspector "edit this kind"; context row "Its kind" |

---

## 12. Names that are suspect, for review

| Name now | Why it is suspect |
|---|---|
| **Makers** | The engine's word. Covers adding, listing, editing a maker and editing one thing. A user thinks "forest", "street", "this tree" |
| **Kinds** vs **Designs** vs **sort** | A kind is a design of your own; "sort" is both what a thing is and which variant of a design ("which of its sort") |
| **Inspector** / **node** / **generator** / **parameter** | Programming words for "this maker's settings" |
| **What it reads** / **Read by** | The engine's connections; a user would say what it follows or stands on |
| **The land, by hand** | Terrain sculpting and painting; nothing says "shape" or "terrain" in the title |
| **clear** / **let grow** | Sound like editing commands; they set where growth is allowed |
| **Paint** (land surface) vs **Paint where…** (a maker's area) | Two different paints |
| **Make it mine** | Makes a copy; changes nothing that stands |
| **Detach** | Unexplained anywhere |
| **What needs a look** | About hand edits that lost their place |
| **the usual** | What a maker makes with no designs given: depends on the style |
| **parts** | Shares of a mix, not pieces of a thing |
| **reach** | A radius; "size" or "radius" would be found sooner. Also the shaping strip's "reach" (how many corners move together): the same word for two things |
| **arrangement** | Chosen by the user for "what a maker made"; appears beside "maker" in most sentences, which makes them long |
| **Save the world** vs **Save the game** | Two different things saved, F5/F9 against Ctrl+S |
| **Environment** | Holds terrain tools, adding makers, and style/time/weather |

---

## 13. What changed in the interface on 2026-10-11

So that this inventory can be compared with what was there the day before:

- **New:** Window > Colours (four); Environment > Style: NAME entries; Makers >
  "Where it makes its things" (how many, keep to the level, Paint where…, Rub out…,
  Nothing painted, brush); Makers > "This one …": its design, its size, Shape this
  one only…, Shape its design…, Add another … to this arrangement…; Makers > "Its
  arrangement": three more put-back buttons and "Add one thing by hand…"; Designs >
  "Shape it in the view…"; the shaping strip's first line, edges in the selection
  colour, and corner dots.
- **Changed behaviour:** a maker's reach, dragged or typed in Makers, keeps the
  maker's thickness (its count changes); dragging a reach measures on the maker's own
  level; a maker's ring is drawn on the land while dragged.
- **Changed words:** Designs "Change it (Kinds and meshes)" is "Change its text
  (Kinds and meshes)"; the notes under "Make it mine"; the notice after adding a wood;
  colour names in sentences follow the colours chosen.
- **Not changed:** every other name, menu and place.
