# Light / Dark theming — ColorThemes (AtlantaFX)

Goal: app-wide light/dark theming for B4J forms via a `ColorThemes` class.

## Decisions

- **Engine:** [AtlantaFX](https://github.com/mkpaz/atlantafx) v2.1.0 (MIT). Requires JavaFX 17+ —
  our build is **JavaFX 17.0.6** (JDK 19 at `C:\dev\b4x\java19`), so it's compatible.
- **Mechanism:** AtlantaFX is applied as the JavaFX **user-agent stylesheet**
  (`Application.setUserAgentStylesheet`). This is **app-global** (themes every form/control
  with one call), NOT per-form. `null` restores the built-in Modena theme.
- **No Java dependency:** we use the prebuilt CSS files only (self-contained via data-url
  fonts, hence the 17+ requirement). Files live in `Files/` and are registered in the project.

## Bundled theme files (in Files/)

`primer-light` / `primer-dark` · `nord-light` / `nord-dark` ·
`cupertino-light` / `cupertino-dark` · `dracula` (dark only)

## Done

- [x] Download AtlantaFX 2.1.0 CSS into `Files/`, register all 7 in `GUIHelpers.b4j`.
- [x] `ColorThemes` class: `Apply(family, dark)`, `ApplyCssFile(name)`, `SetLight`/`SetDark`/
      `Toggle`, `UseModena`, `IsDark`/`Family` getters. Registered as `Module4=ColorThemes`.
- [x] Test harness on a "Theme Testing" tab (TabPane) in B4XMainPage.
- [x] Apply theme at startup (`B4XPage_Created`).
- [x] **Auto / follow-OS mode** — `IsOsDark` + `ApplyAuto(family)`. Detects via process calls
      (no extra library): Windows `reg query AppsUseLightTheme` (0x0=dark); Linux GNOME
      `gsettings ... color-scheme` (prefer-dark); KDE Plasma `kreadconfig6/5 ... ColorScheme`
      (BreezeDark); `gtk-theme` name fallback; macOS `defaults read -g AppleInterfaceStyle`. One-time read —
      JavaFX 17 has no live OS-change listener (22+ only).
- [x] **Persistence** — `Save` (auto-called by `Apply`) + `LoadOrDefault(family, dark)` store the
      family + mode under `File.DirData("GUIHelpers")\theme.txt`. Startup calls `LoadOrDefault`,
      so the app reopens with the last-used theme (first run falls back to the OS preference).
- [x] **Validate CSS exists** — `ApplyCssFile` checks `File.Exists` and logs instead of silently
      failing on a missing/mis-typed css.
- [x] **Theme-changed callback** — `AddThemeListener(Target, EventName)` /
      `RemoveThemeListener(Target)`. Fires `<EventName>_ThemeChanged (Family As String,
      Dark As Boolean)` via CallSubDelayed on every Apply/UseModena.

## ToDo

- [ ] **Live follow-OS polling** — optional `Timer` that re-checks `IsOsDark` every few seconds
      and re-applies, since JavaFX 17 has no OS-change listener (only resolves once today).
- [x] **Per-form accent / custom colors** — `SetAccent(Form, Clr)` / `ClearAccent(Form)`.
      Sets AtlantaFX accent CSS vars (`-color-accent-fg/-emphasis/-muted/-subtle`) on the form's
      root node, cascading to that window only. Includes a color-int → `#RRGGBB` helper.
- [ ] **`AvailableThemes()` list** — return families/files to populate a real settings dropdown.
- [x] **Density / font scale** — `SetDensity(Target, Level)` scales the container's root
      `-fx-font-size` (Compact 11 / Normal 14 / Comfortable 17 px); em-based paddings follow, so
      the whole UI tightens/expands. Demo: Density buttons on the Theme tab.

## Related: custom title bar (TitleBar.bas)

- [x] Cross-platform custom (undecorated) title bar — split into `frmTitleBar` (thin form shell)
      + `TitleBarHelper` (engine: builds header/content, owns view events). Themed header
      (title + min/max/close) + content area; drag to move, double-click to maximize. Colors use
      AtlantaFX looked-up vars (`-color-bg-default`, etc.) so it follows light/dark automatically.
      Pure JavaFX (no native calls). Demo: "Open Custom Title Bar Window" on the Theme tab.
- [x] Edge/corner resize (8 zones with resize cursors + min-size clamp).
- [x] Maximize/restore glyph toggle; `ShowMinimize`/`ShowMaximize` to hide those buttons.
- [x] Optional borders: `AccentBorder` (always accent) and `FocusBorder` (accent when focused,
      neutral when not).
- [x] `RoundedShadow` — rounded corners + drop shadow via a transparent-stage "card" inset by
      shadow padding. Must be set before Show.
- [x] Aero-snap drag gestures — release near a screen edge snaps: top = maximize, left/right =
      half, corners = quarter (uses `Screen.getVisualBounds`, so it respects the taskbar).
      Dragging a snapped/maximized window restores it under the cursor. While docked the
      rounding/shadow is suppressed so it sits flush.
- [ ] Apply the custom bar to the B4XPages **main** window (needs header-above-page layout work).

## Known gotchas

- **Designer/explicitly-set colors win.** Controls colored in the layout or via code
  (e.g. formHelpers `BackColor`) won't follow the theme. Leave controls uncolored to inherit.
- **B4XPages panel background.** The page panel may sit over the form root; same issue seen
  with `BackColor`. The user-agent stylesheet covers controls, but a custom root fill won't.
- **Switching is global & instant** — calling `Apply` re-themes all open windows at once.
