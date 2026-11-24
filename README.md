# Saule Maker

A game maker in the style of GameMaker 8.0, written in Saule on top of
**uikit** (views) and **Shine2D** (window, drawing, input).

uikit is used **from the local checkout at `../saule-uikit`** (a path
dependency), so fixes made there while building this take effect right
away. Switch back with `saule remove ../saule-uikit` followed by
`saule install lauriszz123@saule-uikit:main` once they are pushed.

```sh
saule run
```

## What's there

The main window has the classic layout:

```
┌──────────────────────────────────────────────────────────────┐
│ File  Edit  Resources  Scripts  Run  Window  Help            │  menu bar
├──────────────────────────────────────────────────────────────┤
│ new open save │ exe publish │ ▶ 🐞 │ 9 × create │ info … │ ? │  tool strip
├──────────────┬───────────────────────────────────────────────┤
│ ▸ Sprites    │                                               │
│ ▸ Sounds     │   editor windows float here: drag, resize,    │
│ ▸ …          │   minimise, maximise, close, bring to front   │
│ Game Info    │                                               │
├──────────────┘                                               │
│  resource tree  ┃  draggable splitter  ┃  workspace          │
└──────────────────────────────────────────────────────────────┘
```

* **Menu bar**: click a title, then slide along the bar and each menu opens
  in turn. Click away or press Escape to close it.
* **Tool strip**: one button per command, with a tooltip showing the shortcut.
* **Resource tree**: a folder per kind. Double-click to open an item, and
  right-click for its menu.
* **Workspace**: each resource opens in its own child window. Only the
  focused window is drawn at full strength; the others are washed out but
  stay live where they are visible, so hovering over them still highlights
  things. Whatever a window covers ignores the pointer.

### The sprite editor

**Sprite Properties** (double-click a sprite, or Resources → Create Sprite):

* **Load Sprite** opens a file picker (there is no native one in Shine, so
  it's built in `widgets/dialogs`) with a preview of the selected PNG. A
  file named `something_stripN.png` is cut into N subimages side by side.
  `assets/samples/` has three to try: `coin_strip8.png`,
  `slime_strip4.png` and `ball.png`.
* The preview magnifies pixel art in whole steps with crisp sampling, and
  marks the origin with a red cross. Click or drag on the image to move the
  origin, type it into X and Y, or press **Center**.
* **Show** steps through the subimages and ▶ plays the animation.
* The two collision checkboxes are stored on the sprite.

**Edit Sprite** opens the Sprite Editor: every subimage as a thumbnail, with
buttons to add more from a file, duplicate, delete, and move one left or
right.

**The image editor** paints one subimage. Open it by double-clicking a
thumbnail in the Sprite Editor, or select one and press the pencil. Add an
empty subimage there to draw from scratch.

* **Tools:** pencil, eraser, line, rectangle, filled rectangle, flood fill
  and colour picker.
* **Two colours**, as in GameMaker: the left button paints with the left
  colour and the right button with the right one.
  * Click the front (left) or back (right) colour square to open the colour
    picker for that slot. **Choose colour...** opens it for the left one.
  * The picker has a saturation/brightness square, a hue strip, an opacity
    bar, hex and R/G/B/A fields, and old/new swatches (click "old" to go
    back).
  * Colours you pick appear under **Recent**, shared by every image editor.
  * You can also take colours from the palette (left or right click), type
    hex (`#RRGGBB`, or `#AARRGGBB` for transparency), or sample them with the
    picker tool. The ⇄ button swaps the two.
* **Undo and redo** with the buttons or Ctrl+Z / Ctrl+Y (Ctrl+Shift+Z also
  redoes), up to 64 steps.
* **Zoom** with the mouse wheel or the buttons. "Fit" picks the biggest
  whole magnification that fits the window. The pixel grid shows at 6× and
  up, and can be turned off.
* The status line shows the pixel under the pointer, the image size and the
  zoom.

Changes show in the thumbnails and the Sprite Properties preview as soon as a
stroke ends.

Each subimage owns its pixels (loading a strip cuts it into separate
images), so any one can be painted on without touching the others. Nothing is
saved to disk yet; that comes with saving projects.

### The object editor

**Object Properties** has three columns: the object's settings, its events,
and the selected event.

* **Settings:** the sprite (picked from a list, with a thumbnail; **New**
  makes a sprite for it, **Edit** opens it), Visible, Solid, Persistent,
  Depth, the **Parent** to inherit events from (objects that would make a
  loop are greyed out), and a collision **Mask** sprite.
  **Show Information** writes the whole object out as text.
* **Events:** **Add Event** offers Create, Destroy, Alarm, Step (begin,
  normal, end), Collision, Keyboard, Key Press, Key Release, Mouse, Other,
  User and Draw. **Change** turns an event into a different one, keeping
  its code. Right-click an event for Edit Script, Change and Delete. An
  object can't have the same event twice.
* **Alarm and User events are named**, for example Alarm (ShootRate) or
  User (OnHit). Scripts start alarms and fire user events by those names.
  An alarm also has a default time in steps or seconds, **Repeat**, and
  **Start on create**. These are the defaults the game's scripts start from;
  the timing itself is up to the scripts.
* **Inherited events** from the parent (and its parents) show greyed out,
  with the parent's name. Selecting one offers **Override** (copy it into
  this object to change) or opening the parent.
* **Scripts only, no drag and drop:** each event holds Saule code. The right
  column previews it, and **Edit Script** (or double-clicking the event)
  opens it in its own window. It's saved as you type.

Objects refer to sprites and other objects by id, so renaming never breaks
anything. Deleting a sprite clears it from objects that used it. Deleting an
object clears it as a parent, and removes collision events with it.

Commands that aren't built yet (Open, Save, Run, …) are greyed out
everywhere. Each one lights up once its action exists in
`editor/Commands.sau`.

## Layout of the source

Each folder is one layer. A layer only imports the layers listed above it,
never the ones below:

```
src/
├── main.sau              entry point: EditorApp().run()
│
├── util/                 HELPERS THE STDLIB LACKS
│   └── Parse             text → integer, decimal, hex
│
├── model/                WHAT A GAME IS (no UI)
│   ├── Signal            tiny observer: connect / emit / disconnect
│   ├── ResourceKind      Sprite, Sound, Background, Path, Script, Font, TimeLine, Object, Room
│   ├── Resource          one resource: id, kind, name, plus `sprite` / `object`
│   ├── SpriteData        frames, origin, collision settings
│   ├── SpriteFrame       one subimage: its own image (pixels), size, id
│   ├── EventKind         Create, Destroy, Alarm, Step, Collision, Keyboard, …
│   ├── EventCatalog      step phases, keys, mouse and other events
│   ├── ObjectEvent       one event: kind + detail/target, code, AlarmSettings
│   ├── ObjectData        sprite, mask, parent, flags, depth, sorted events
│   ├── ResourceGroup     a folder of resources (nests)
│   └── GameProject       the whole game, inheritance, event titles; emits `changed`
│
├── editor/               THE EDITING SESSION (no UI)
│   ├── Document          one open editor window: what it edits, where it is
│   ├── Workspace         open documents in z-order: open, focus, move, resize, cascade…
│   ├── Editor            project + workspace + tree selection; the views call this
│   ├── SpriteImporter    PNG (or _stripN.png) → sprite frames, blank frames
│   ├── ObjectReport      an object as plain text ("Show Information")
│   ├── ImageSession      painting one image: tools, colours, undo/redo
│   ├── Shortcut          Ctrl+S and friends
│   ├── Command           one user action, shared by menu, tool strip and keyboard
│   └── Commands          every command, built once
│
├── theme/                HOW IT LOOKS
│   ├── Palette           every colour, named by use, plus a uikit ThemeData
│   ├── Metrics           shared sizes
│   ├── Glyphs            Material Icons codepoints
│   ├── ResourceStyle     glyph + accent per resource kind
│   └── EventStyle        glyph + accent per event kind
│
├── widgets/              REUSABLE UI COMPONENTS (know nothing about games)
│   ├── common/           Label, Rule, Edges, DropShadow, PressWatcher
│   ├── menu/             MenuBar, Menu, MenuEntry, MenuPanel, openMenuPopup
│   ├── toolstrip/        ToolStrip, ToolButton, ToolSeparator
│   ├── tree/             TreeView, TreeRow, TreeNode
│   ├── split/            SplitPane, Splitter
│   ├── mdi/              MdiArea, MdiWindow, MdiTitleBar, MdiFrame, ResizeGrip
│   ├── form/             Field, NumberField, TextArea, PushButton, ListBox, FormRow,
│                         Dropdown, ChoiceList
│   ├── preview/          SpritePreview, ImageRegion, Checkerboard, GridCanvas
│   ├── paint/            PixelCanvas, Swatch, ColorPair, PixelReadout, Argb, Hsv,
│                         ColorFields (square / hue / alpha), openColorPicker
│   └── dialogs/          openFileDialog, openTextDialog, DialogFrame, Paths
│
├── views/                THE EDITOR'S SCREENS (widgets + editor)
│   ├── MainWindow        the whole screen; the one view that listens for changes
│   ├── shell/            MainMenu, MainToolStrip, WorkspaceView, commandEntry
│   ├── resources/        ResourceTreePanel
│   └── editors/          ResourceEditor (base) → Sprite/Object/Room/Script/Pending editors,
│                         SpriteFramesEditor ("Edit Sprite"), ImageEditor
│                         (painting a subimage), EventPanel, AddEventDialog,
│                         EventScriptEditor (one event's code), GameInfoEditor,
│                         GameSettingsEditor, DocumentContent (picks one)
│
└── app/                  THE APPLICATION
    ├── AppInfo           name, version, asset paths
    └── EditorApp         the window, the theme, the Editor instance
```

### How data flows

```
 user clicks ──► view calls Editor / Workspace / Command
                        │
                        ▼
                 model or workspace changes ──► `changed` signal
                                                      │
 MainWindow rebuilds ◄────────────────────────────────┘
        │
        └─► every panel redraws from the editor's current state
```

No panel keeps its own copy of anything, so no panel can go stale. Widgets
are *controlled*: they take data and callbacks and report what the user did.
The owner decides what happens next.

## Conventions

* One class per file, and the file is named after the class.
* Every folder has an `init.sau` that re-exports its files, so code imports
  a folder (`from widgets import *`) instead of reaching into it.
* Widgets read colours and sizes from `theme`, never as literals.
* 4-space indentation (`saule fmt -w <files>` follows `saule.config`).

### Saule gotchas found while setting this up

* Don't name a folder `core`, `io`, `math`, `os`, `project`, `string`,
  `table`, `iter`, `ops`, `bytes`, `coroutine` or `saule`. Those are stdlib
  packages, and an import of that name resolves to the stdlib before it
  looks at your folder. That's why the data layer is `model/`, not `core/`.
* Don't start a statement with `(`. As in Lua, it continues the line above,
  so `(a ?? b).apply()` turns into a call on the previous expression. Bind it
  to a `local` first.
* A file that calls an inherited method has to import the type that declares
  it. For example, `main.sau` imports `App` to call `EditorApp().run()`.
* uikit exports a *view* called `Image`, and shine has a native `Image`. A
  file that imports both should get sizes through `Images.size(path)`.
  Saule 26.30 compiled `image.getDimensions()` against the wrong `Image`;
  that's fixed after 26.30.
* In 26.30, `info.kind == FsKind.Dir` is always false. Use
  `match info.kind case FsKind.Dir …` instead (also fixed after 26.30).
