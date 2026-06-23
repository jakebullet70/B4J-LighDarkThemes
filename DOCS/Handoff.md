# Handoff: theming + custom title bar for B4J

## Goal
A reusable B4J GUI toolkit: app-wide light/dark theming (`ColorThemes`, engine: AtlantaFX),
a cross-platform custom title-bar window (`frmTitleBar` + `TitleBarHelper`), and the earlier
.NET-style Form API (`formHelpers`). All three are functional and exercised by a tabbed test
harness in `B4XMainPage`.

## Status

- **formHelpers** (done) — WindowState, FormBorderStyle, size/pos, Icon, BackColor, Cursor,
  Show/Hide/Close/Activate, modal `ShowDialog` (ResumableSub). See HandOff-FormHelpers.md.
- **ColorThemes** (done) — families Primer/Nord/Cupertino/Dracula + Modena; Light/Dark/Toggle;
  **Auto/follow-OS** (Win + Linux GNOME/KDE + mac); **persistence**; **theme-changed callback**;
  **per-form accent**; **density** (Compact/Normal/Comfortable via root font scale).
- **Custom title bar** (done) — undecorated window with themed header (title + min/max/close),
  drag-to-move, double-click maximize, **edge/corner resize**, **Aero-snap** (top=max, sides=half,
  corners=quarter, restore-on-drag), **rounded corners + drop shadow** (`RoundedShadow`),
  **accent / focus-aware borders**, configurable min/max buttons, maximize/restore glyph.
- Remaining ToDo (DOCS/ColorTheming-ToDo.md): `AvailableThemes()` list; apply the custom bar to
  the B4XPages **main** window; optional snap-preview overlay.

## Key files
- `ColorThemes.bas` — theming. `Apply`/`ApplyAuto`, `SetLight/SetDark/Toggle`, `UseModena`,
  `LoadOrDefault`/`Save`, `AddThemeListener`, `SetAccent`/`ClearAccent`, `SetDensity`, `IsOsDark`.
- `TitleBarHelper.bas` — the title-bar engine: builds card/header/content, owns the view events
  (move/resize/snap/buttons), theming. Attach to any Form via `Initialize(form, title)`.
- `frmTitleBar.bas` — thin form shell: creates the Form, forwards `Resize`/`FocusChanged` to the
  helper, exposes pass-through API. (NOTE: module is `frmTitleBar`; `TitleBar.bas` is an orphan.)
- `formHelpers.bas` — .NET-style Form API. `frmDialog.bas` — modal test form.
- `B4XMainPage.bas` — tabbed harness (Form Testing + Theme Testing tabs).
- `GUIHelpers.b4j` — `MainForm` Public; `Files/*.css` = 7 AtlantaFX themes; modules:
  B4XMainPage, ColorThemes, formHelpers, frmDialog, frmTitleBar, TitleBarHelper.
- `DOCS/`: ColorTheming-ToDo.md, FormHelper-ToDo.md, B4J-Coding-Reference.md, HandOff-FormHelpers.md.

## Things you'd get wrong by guessing

- **JavaFX is 17.0.6** (JDK 19 at `C:\dev\b4x\java19`) — NO Platform Preferences API. OS dark
  detection uses one-time process calls (`reg query` / `gsettings` / `kreadconfig` / `defaults`),
  not the JavaFX API. No live OS-change listener.
- A B4J `Form` does NOT unwrap to the JavaFX Stage — `formHelpers` resolves it via
  RootPane → Scene → Window.
- **Event routing:** a view's events go to the module that *created* it. So `TitleBarHelper`
  (which builds the header/buttons) gets their events, but the **Form's** Resize/FocusChanged go
  to `frmTitleBar` (which created the form) and are forwarded to the helper.
- AtlantaFX is app-global via `Application.setUserAgentStylesheet` (null = Modena). Per-form accent
  and the title bar colors use AtlantaFX looked-up vars (`-color-bg-default`, `-color-accent-emphasis`).
- `RoundedShadow` needs a TRANSPARENT stage (set before Show) + a "card" inset by shadow padding;
  rounding/shadow are suppressed while docked (maximized/snapped) so it sits flush.
- CSS files / new modules must be **registered in the project AND the IDE reopened** (the IDE caches
  the in-memory file/module list — files or modules added on disk while it's open won't load).
- B4X gotchas: don't init a global from another global in Class_Globals; a param-taking sub can't
  start with `get`/`set`; avoid reserved type names as identifiers (e.g. `tab` vs `Tab`);
  `File.Exists` on `DirAssets` is unreliable.
- Follow `DOCS/B4J-Coding-Reference.md`; cite GitHub source links in file headers.
- Can't compile here (no B4J toolchain) — user builds/runs in the B4J IDE.

## Housekeeping

- `TitleBar.bas` is an orphan (superseded by `frmTitleBar.bas`, which is the registered module).
  Safe to delete.

## Very next step
Pick from the remaining ToDo: `AvailableThemes()` (drives a real settings UI), apply the custom
title bar to the B4XPages main window, or the snap-preview overlay.
