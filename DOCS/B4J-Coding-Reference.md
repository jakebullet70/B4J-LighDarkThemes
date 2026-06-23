# B4J Coding Reference

A working reference for writing **B4J** code. Hand this to Claude as context and B4J code should follow these conventions, idioms, and library patterns.

> B4J runs on the JVM. It produces JavaFX desktop/UI apps, non-UI console apps, and server apps (jServer). The language is the shared B4X BASIC dialect.

---

## 1. Project & Module Types

**Project types**
- **UI app** — JavaFX desktop app. `MainForm As Form`, layouts loaded with `LoadLayout`.
- **Non-UI app** — console/background app. Entry point is `AppStart`; keep alive with `StartMessageLoop` / end with `StopMessageLoop`.
- **Server app (jServer)** — HTTP server using the `jServer` library and handler classes.

**Module types**
- **Main module** — entry point. UI: `AppStart(Form1 As Form, Args() As String)`. Non-UI: `AppStart(Args() As String)`.
- **Code module** — static-style shared subs/globals via `Process_Globals`. No instances.
- **Class module** — instantiable type; `Class_Globals` + `Initialize` + members.
- **B4XPages** — cross-platform page framework; each page is a class with `B4XPage_Created(Root As B4XView)`.

```b4j
' Non-UI Main
Sub Process_Globals
    Private const VERSION As String = "1.0"
End Sub

Sub AppStart (Args() As String)
    Log("Starting " & VERSION)
    StartMessageLoop   ' keep app alive for async work; omit for quick scripts
End Sub
```

---

## 2. Syntax Essentials

```b4j
' Comment
Dim count As Int = 0
Private name As String = "B4J"

If count > 0 Then
    Log("positive")
Else If count = 0 Then
    Log("zero")
Else
    Log("negative")
End If

Select Case name
    Case "B4J", "B4A"
        Log("b4x")
    Case Else
        Log("other")
End Select

For i = 0 To 9
    Log(i)
Next

For Each item As String In items
    Log(item)
Next

Do While count < 10
    count = count + 1
Loop
```

Operators: `+ - * / Mod`, comparison `= <> < > <= >=`, logical `And Or Not`, string concat `&`.

---

## 3. Types

| Type | Notes |
|------|-------|
| `Int` `Long` | 32 / 64-bit integers |
| `Float` `Double` | floating point |
| `String` | text |
| `Boolean` | `True` / `False` |
| `Byte` `Short` `Char` | smaller integral / char |
| `Object` | any reference |
| `List` `Map` | dynamic collections |
| Arrays | `Dim a(10) As Int`, `Dim a() As String = Array As String("x","y")` |

Objects must be **initialized** before use: `obj.Initialize`. Most uncaught runtime errors come from skipping this.

---

## 4. SmartStrings (preferred for text with interpolation/multiline)

Use `$"..."$` SmartStrings instead of manual quote-escaping.

```b4j
Dim user As String = "Sam"
Dim msg As String = $"Hello ${user}, you have ${count} messages."$
```

For multi-line / HTML / SQL, use normal quotes inside the SmartString — never doubled quotes:

```b4j
Dim html As String = $"
<div class="card">
  <span>${user}</span>
</div>
"$
```

Correct: `$"<div class="card"></div>"$` — Wrong: `$"<div class=""card""></div>"$`.

---

## 5. Subs, Getters & Setters

```b4j
Sub Add(a As Int, b As Int) As Int
    Return a + b
End Sub
```

Expose properties with `getX` / `setX` naming (B4X convention — the IDE surfaces these as properties):

```b4j
Private mText As String = ""   ' safe default

Public Sub setText(Value As String)
    mText = Value
End Sub

Public Sub getText As String
    Return mText
End Sub
```

Always provide safe default values for fields.

---

## 6. Classes & Class_Globals

```b4j
Sub Class_Globals
    Private mValue As Int
    Private mEnabled As Boolean = True
End Sub

Public Sub Initialize(start As Int)
    mValue = start
End Sub

Public Sub Increment
    mValue = mValue + 1
End Sub

Public Sub getValue As Int
    Return mValue
End Sub
```

Instantiate:

```b4j
Dim c As Counter
c.Initialize(5)
c.Increment
Log(c.Value)   ' 6
```

---

## 7. Events

Two patterns:

**Callback style** (passing a target + event name, common in components):

```b4j
' store mCallBack As Object, mEventName As String in Class_Globals/Initialize
If SubExists(mCallBack, mEventName & "_Click") Then
    CallSub(mCallBack, mEventName & "_Click")
End If
```

Guard with `SubExists` before `CallSub` to avoid runtime errors. Pass args with `CallSub2` / `CallSub3`.

**RaiseEvent style** (declared events on a class):

```b4j
' In Class_Globals
Private mEventName As String
' In Initialize(CallBack As Object, EventName As String)
mEventName = EventName

' Raise
RaiseEvent(...)   ' or via CallSub for cross-module
```

Event handler naming: `<eventname>_<Event>`, e.g. `button_Click`, `timer_Tick`.

---

## 8. Async: Wait For, Sleep, Resumable Subs

B4J async uses **resumable subs**. Code reads top-to-bottom; `Wait For` and `Sleep` yield without freezing.

```b4j
Sub DownloadJson
    Dim job As HttpJob
    job.Initialize("", Me)
    job.Download("https://example.com/data.json")
    Wait For (job) JobDone(job As HttpJob)
    If job.Success Then
        Log(job.GetString)
    Else
        Log("Error: " & job.ErrorMessage)
    End If
    job.Release
End Sub
```

```b4j
Log("before")
Sleep(1000)          ' yields ~1s, app stays responsive
Log("after")
```

`Wait For` can target a specific object instance, anonymous completions (`Wait For (sender) Complete(result As ...)`), or `Wait For Msgbox_Result`. Subs containing `Wait For`/`Sleep` are resumable and may need `StartMessageLoop` in non-UI apps to stay alive.

---

## 9. Common B4J Libraries & Patterns

**HttpJob (jOkHttpUtils2)** — HTTP requests, shown above. `job.PostString`, `job.PostBytes`, `job.GetString`, `job.GetBytes`, `job.Success`.

**jSQL** — SQLite/MySQL etc.

```b4j
Dim sql As SQL
sql.InitializeSQLite(File.DirApp, "data.db", True)
sql.ExecNonQuery("CREATE TABLE IF NOT EXISTS users (id INT, name TEXT)")
sql.ExecNonQuery2("INSERT INTO users VALUES (?, ?)", Array As Object(1, "Sam"))

Dim rs As ResultSet = sql.ExecQuery("SELECT name FROM users")
Do While rs.NextRow
    Log(rs.GetString("name"))
Loop
rs.Close
```

**jServer** — web server.

```b4j
Dim srvr As Server
srvr.Initialize("srvr")
srvr.Port = 51042
srvr.AddHandler("/hello", "HelloHandler", False)
srvr.Start
```

A handler is a class implementing `Handle(req As ServletRequest, resp As ServletResponse)`.

**jXUI / B4XPages** — cross-platform UI via `B4XView`, `XUI` colors/fonts, layouts via `Root.LoadLayout("name")`.

**JSON**

```b4j
Dim parser As JSONParser
parser.Initialize(jsonString)
Dim root As Map = parser.NextObject
```

---

## 10. JVM Interop (JavaObject & Inline Java)

For functionality not wrapped by a library, use `JavaObject` and inline Java.

```b4j
Dim jo As JavaObject
jo.InitializeStatic("java.lang.System")
Dim t As Long = jo.RunMethod("currentTimeMillis", Null)
```

Inline Java block (placed at end of a module):

```b4j
#If JAVA
public static String greet(String name) {
    return "Hello " + name;
}
#End If
```

Call it via `JavaObject` `RunMethod`. Prefer existing B4J libraries first; reach for JavaObject only when needed.

---

## 11. Conditional Compilation

```b4j
#If DEBUG
    Log("debug build")
#End If

#If RELEASE
    ' production-only code
#End If
```

CSS/JS blocks (BANano and similar) use `#If CSS ... #End If`. Build configurations toggle these symbols.

---

## 12. Files, Logging, Errors

```b4j
Log("message")                       ' to IDE logs

File.WriteString(File.DirApp, "a.txt", "hello")
Dim s As String = File.ReadString(File.DirApp, "a.txt")
```

Common dirs: `File.DirApp` (app folder, B4J), `File.DirAssets` (read-only project files), `File.DirTemp`, `File.DirData(...)`.

Error handling with `Try`/`Catch`:

```b4j
Try
    Dim n As Int = "not a number"   ' will throw
Catch
    Log("Failed: " & LastException)
End Try
```

---

## 13. Conventions to Follow When Generating B4J Code

- Use **SmartStrings** (`$"..."$`) for interpolation and multi-line text; never use double-double quotes.
- Initialize every object before use (`obj.Initialize`).
- Use `getX` / `setX` for public properties; give fields safe defaults.
- Put instance fields in `Class_Globals` (classes) or `Process_Globals` (code modules / Main).
- Guard cross-module events with `SubExists` before `CallSub`.
- Use `Wait For` / `Sleep` for async — never block the main thread; add `StartMessageLoop` in non-UI apps that rely on async.
- Prefer existing B4J libraries; use `JavaObject` / inline Java only when no library covers it.
- Release resources: `job.Release`, `rs.Close`, `sql.Close`.
- Keep classes small and APIs predictable; document non-trivial classes with a short header comment listing events and properties.

---

*Scope: general B4J. For BANano (B4J→HTML/CSS/JS) web components there are additional, stricter conventions (sk- CSS prefixes, lowercase HTML IDs, `#If CSS` styling, BANanoElement null checks, SKInnerAdd registration). Ask if you want those folded in or kept as a separate companion file.*
