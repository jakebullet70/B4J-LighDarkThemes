# Handoff: .NET-style Form helper for B4J

## Goal
Build a B4J helper class you initialize with a `Form` that exposes WinForms-style
form members (WindowState, FormBorderStyle, Opacity, size/pos, Icon, ShowDialog, etc.).
Project: c:\dev\b4x\src\GUIHelpers (B4J + B4XPages, JavaFX under the hood).

## Status
Phases 1–4 implemented & being tested via a button harness in B4XMainPage.
- P1 window state/style, P2 size/pos, P3 appearance, P4 behavior/dialog — done.
- P5 (events) was built then REMOVED on purpose — B4J's native Form events already cover it.

## Key files
- `formHelpers.bas` — the helper class (NOTE: renamed from frmHelper → **formHelpers**).
- `frmDialog.bas` — secondary modal form to test ShowDialog.
- `B4XMainPage.bas` — test buttons + handlers.
- `FormHelper-ToDo.md` — full comparison table + phase log.
- `GUIHelpers.b4j` — main module; `MainForm` is Public. `icon.png` registered in Files.

## Things you'd get wrong by guessing
- A B4J `Form` does NOT unwrap to the JavaFX Stage. Resolve it lazily via
  RootPane → getScene → getWindow (see `GetStage`). All stage calls go through it.
- Don't initialize a global from another global in Class_Globals — B4X throws at runtime.
- `ShowDialog` is a **ResumableSub** using modal Show + Sleep-poll on isShowing.
  Do NOT use `showAndWait` or JavaObject `CreateEvent`/`setOnHidden` proxies —
  they break B4XPages event delivery / "missing RaiseSynchronousEvents".
- B4X has no `Throw`; we throw via an inline `#If JAVA` helper (`throwError`).
- `FormBorderStyle` stage style only applies BEFORE Show; only `Resizable` is live.
- `Hide` disables JavaFX implicit-exit (else hiding last window quits app); `Show` restores it.
- Can't compile here (no B4J toolchain) — user builds/runs in the B4J IDE.
  The B4J IDE may overwrite externally-edited files that are open; confirm edits stick.

## Very next step
Then it's a good stopping point (optional: key/mouse forwarding).
