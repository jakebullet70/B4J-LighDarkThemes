# frmHelper — .NET-style Form API for B4J

Goal: a helper class you initialize with a B4J `Form` and that exposes WinForms-style
properties/methods (`WindowState`, `FormBorderStyle`, `StartPosition`, …).

- **Host language:** B4J (JavaFX). A B4J `Form` wraps a JavaFX `javafx.stage.Stage`.
- **Access strategy:** use native B4J `Form` members where they exist; otherwise reach
  the underlying `Stage` / `Scene` with `JavaObject` (`frm.As(JavaObject)` or a `Stage`
  field obtained via reflection).
- **Existing stub:** [frmHelper.bas](frmHelper.bas) already holds `frm As Form` and `fx As JFX`.

## How to reach the native objects

```vb
Private jo As JavaObject = frm                 ' the JavaFX Stage
Private scene As JavaObject = jo.RunMethod("getScene", Null)
```

Most properties below that say "JavaObject → Stage" map to a single
`stage.RunMethod("setX", Array(value))` / `getX` call.

---

## Comparison table: .NET WinForms Form → B4J

Legend for **Source**:
- ✅ **Native** — B4J `Form` already exposes it; helper just wraps/renames.
- 🟡 **RootPane/JFX** — achievable through `frm.RootPane` or the `JFX` object.
- 🔧 **JavaObject** — needs the underlying JavaFX `Stage`/`Scene`.
- ⛔ **No clean equivalent** — JavaFX limitation; document the gap.

### Window state & style

| .NET member | Type | Source | B4J mapping / notes |
|---|---|---|---|
| `WindowState` (Normal/Minimized/Maximized) | prop | 🔧 | `stage.setIconified(true)` = minimized; `stage.setMaximized(true)` = maximized; both false = normal. Expose as enum. |
| `FormBorderStyle` (None/Sizable/Fixed…) | prop | ✅/🔧 | `frm.SetFormStyle("DECORATED"/"UNDECORATED"/"TRANSPARENT"/"UTILITY")` — **must be set before Show**. Combine with `Resizable` to model Fixed vs Sizable. |
| `StartPosition` (CenterScreen/Manual/…) | prop | 🔧 | `stage.centerOnScreen()` for CenterScreen; Manual = set Left/Top before Show. |
| `TopMost` | prop | ✅ | `frm.AlwaysOnTop`. |
| `Opacity` (0–1) | prop | 🔧 | `stage.setOpacity(d)`. |
| `ShowInTaskbar` | prop | ⛔ | No reliable JavaFX API; document limitation (UTILITY style hides from taskbar on some OSes). |
| `MinimizeBox` / `MaximizeBox` / `ControlBox` | prop | ⛔/🔧 | JavaFX can't toggle individual title-bar buttons; only via `StageStyle` (UTILITY drops min/max). Document. |

### Size & position

| .NET member | Type | Source | B4J mapping / notes |
|---|---|---|---|
| `Text` (title) | prop | ✅ | `frm.Title`. |
| `Width` / `Height` | prop | ✅ | `frm.WindowWidth` / `frm.WindowHeight` (outer) vs `frm.Width`/`frm.Height` (content). Decide which `Size` maps to. |
| `ClientSize` | prop | ✅ | `frm.Width` / `frm.Height` (scene/content area). |
| `Size` | prop | ✅ | Wrap Width+Height. |
| `Location` / `Left` / `Top` | prop | 🔧 | `stage.getX/getY` + `setX/setY`. |
| `DesktopBounds` / `Bounds` | prop | 🔧 | Compose from Location + Size. |
| `MinimumSize` | prop | 🔧 | `stage.setMinWidth/setMinHeight`. |
| `MaximumSize` | prop | 🔧 | `stage.setMaxWidth/setMaxHeight`. |
| `Resizable` (`FormBorderStyle`) | prop | ✅ | `frm.Resizable`. |

### Appearance

| .NET member | Type | Source | B4J mapping / notes |
|---|---|---|---|
| `Icon` | prop | 🔧 | `stage.getIcons().add(image)` — load via `fx.LoadImage`/`Image`. |
| `BackColor` | prop | 🟡 | Set on `frm.RootPane` (CSS `-fx-background-color` or `RootPane.Color`). |
| `ForeColor` | prop | 🟡 | CSS on RootPane; per-control really. Document scope. |
| `Font` | prop | 🟡 | CSS on RootPane. |
| `Cursor` | prop | 🔧 | `scene.setCursor(Cursor.cursor("WAIT"…))`. |
| `Visible` | prop | ✅ | `frm.Show` / `stage.hide()`. |
| `Enabled` | prop | 🟡 | `frm.RootPane.Enabled`. |

### Behavior & dialog

| .NET member | Type | Source | B4J mapping / notes |
|---|---|---|---|
| `Show()` | method | ✅ | `frm.Show`. |
| `ShowDialog()` (modal) | method | 🔧 | `stage.initModality(APPLICATION_MODAL)` + `stage.showAndWait()`. Needs owner set before show. |
| `Hide()` | method | 🔧 | `stage.hide()`. |
| `Close()` | method | ✅ | `frm.Close`. |
| `Activate()` / `BringToFront()` | method | 🔧 | `stage.toFront()` / `requestFocus`. |
| `SendToBack()` | method | 🔧 | `stage.toBack()`. |
| `Focus()` | method | 🔧 | `stage.requestFocus()`. |
| `CenterToScreen()` | method | 🔧 | `stage.centerOnScreen()`. |
| `DialogResult` | prop | 🔧 | Track manually; set on close of a modal form. |
| `AcceptButton` / `CancelButton` | prop | 🟡 | Wire Enter/Esc key handlers on the scene to chosen buttons. |
| `KeyPreview` | prop | 🔧 | Add a scene-level key filter when true. |

### Events (.NET → B4J)

| .NET event | B4J equivalent | Source | Notes |
|---|---|---|---|
| `Load` | none direct | 🔧 | Approximate with `Show` + first layout pass, or `stage.setOnShown`. |
| `Shown` | `setOnShown` | 🔧 | Raise helper event. |
| `FormClosing` (cancelable) | `MainForm_CloseRequest(EventData)` | ✅ | Already in [GUIHelpers.b4j](GUIHelpers.b4j); EventData can `Consume` to cancel. |
| `FormClosed` | `MainForm_Closed` | ✅ | Already present. |
| `Activated` / `Deactivate` | `MainForm_FocusChanged(HasFocus)` | ✅ | Already present. |
| `Resize` | `MainForm_Resize(W,H)` | ✅ | Already present. |
| `Move` | `stage.x/yProperty` listener | 🔧 | Add listener via JavaObject + event. |
| `WindowState`/min change | `MainForm_IconifiedChanged` | ✅ | Already present (iconify only; add maximize listener for full state). |
| `KeyDown`/`KeyPress`/`KeyUp` | scene key handlers | 🔧 | `scene.setOnKeyPressed` etc. |
| `MouseClick` | RootPane mouse events | 🟡 | `frm.RootPane` touch/click events. |

---

## Implementation ToDo

### Phase 0 — scaffolding ✅

- [x] Add `stage As JavaObject` and `scene As JavaObject` fields to [frmHelper.bas](frmHelper.bas), populated in `Initialize`.
- [x] Define enums (B4J has no real enums — use `Public Const` int sets):
      `WindowState`, `FormBorderStyle`, `StartPosition`.

### Phase 1 — high-value props (the ones you named) ✅

- [x] `WindowState` get/set (iconified + maximized).
- [x] `FormBorderStyle` get/set (`SetFormStyle` + `Resizable`); "before Show" constraint documented in code.
- [x] `StartPosition` (CenterScreen / Manual).
- [x] `Text` (Title), `TopMost` (AlwaysOnTop), `Opacity`.

### Phase 2 — size & position ✅

- [x] `Left`/`Top`/`Location`, `Width`/`Height`/`Size`, `ClientWidth`/`ClientHeight`.
- [x] `MinimumSize`/`MaximumSize` (as `SetMinimumSize`/`SetMaximumSize`).
- [x] `CenterToScreen()`.

### Phase 3 — appearance ✅

- [x] `Icon` (`SetIcon` / `SetIconFromFile`).
- [x] `BackColor` via RootPane.
- [x] `Cursor` (CURSOR_* names).

### Phase 4 — behavior & dialog ✅

- [x] `Show`/`Hide`/`Close`/`Activate`/`BringToFront`/`SendToBack`/`Focus`.
- [x] `ShowDialog()` modal + `DialogResult` (RESULT_* values).
      Implemented as a `ResumableSub` (modal `Show` + `Wait For`), NOT `showAndWait` —
      `showAndWait`'s nested event loop breaks B4XPages event delivery. Call with
      `Wait For (helper.ShowDialog) Complete (Result As Int)`. Throws if already shown.

### Phase 5 — events (dropped)

Not implemented. B4J's native `Form` events (`_Resize`, `_FocusChanged`,
`_IconifiedChanged`, `_Closed`, `_CloseRequest`) already cover these, so wrapping them in
the helper only duplicated what B4J provides. Use the native form events directly.

### Documented gaps (no clean JavaFX equivalent)
- [ ] `ShowInTaskbar`, `MinimizeBox`/`MaximizeBox`/`ControlBox` — note limitations in code comments.

---

## Open decisions
1. **`Size` semantics** — should `Width`/`Height` map to the outer window
   (`WindowWidth`) or content area (`Width`)? .NET `Size` = outer, `ClientSize` = inner.
2. **Enum representation** — string constants (readable) vs integer constants (faster compare)?
3. **Pre-Show vs post-Show** — some props (`FormBorderStyle`) only apply before `Show`.
   Cache requested values and apply on `Show`, or just document the constraint?
4. **Event wiring** — forward through B4J's existing `MainForm_*` delegates, or attach
   fresh JavaFX listeners in the helper?
