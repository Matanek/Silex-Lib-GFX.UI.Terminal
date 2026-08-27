# GFX.UI.Terminal

`GFX.UI.Terminal` adds terminal controls to `GFX.UI`. It provides two modes:
`Terminal.Shell()` communicates with the operating system, while
`Terminal.Console()` exposes commands owned by the application.

Both controls are ordinary `UI.CustomView` values. They participate in UI
composition, constraints, focus, keyboard/text routing and runtime
invalidation without placing terminal or UI state in an ECS `World`. The
package has no dependency on `GFX.ECS`, `GFX.Scene2D` or `GFX.Rendering`.

The shortest path to a system terminal is deliberately small:

```sx
use GFX.UI
use GFX.UI.Terminal
use GFX.Viewer

var terminal = Terminal.Shell()
Viewer.show(UI.Session(terminal, autofocus:true))
```

The application console declares its commands directly. `help` and `clear` are
built in; the remaining text is passed to the selected action:

```sx
use GFX.UI
use GFX.UI.Terminal

func status(arguments:str) str { return "application is running" }
func echo(arguments:str) str { return arguments }

var terminal = Terminal.Console(
    commands:[
        Terminal.Command("status", status, description:"Show application status"),
        Terminal.Command("echo", echo, description:"Print the provided text")
    ],
    welcome:["type 'help' to list commands"]
)
```

By default, `Terminal.Shell` owns the platform process and its lifecycle.
Its PTY/ConPTY transport, nonblocking polling and cleanup remain internal.
Both modes invalidate through their surrounding
`UI.Runtime`; `UI.Session` supplies that behavior to any `Canvas.Session`
presenter. None of these layers introduces an application-world dependency.

Applications that own several terminal tabs can make the lifecycle explicit:

```sx
var process = Terminal.ShellSession()
var terminal = Terminal.Shell(
    session:process,
    on_exit:func() {
        // Remove this terminal's tab. The application may close its window
        // separately when no tabs remain.
    }
)

match process.close() {
    failure(error) => { panic(error.operation + ": " + error.detail) }
    success => {}
}
```

`ShellSession.close()` terminates only that shell. It never closes an
application window. `on_exit` runs once after the terminal observes the child
exit, whether it was requested through `close()` or initiated by the shell.
Once exited, the view stops accepting input; late key or text events are
ignored instead of being reported as broken-pipe runtime errors.

`UI.Session(terminal, autofocus:true)` optionally focuses the first text-input
control immediately, so its caret is visible without a click. By default,
`Ctrl+C` is forwarded to the shell. A single-terminal application can set
`Terminal.Shell(close_on_interrupt:true)` to make it close that shell session
instead; `on_exit` or a `Viewer.show_until` predicate can then decide whether
the surrounding tab or window should close.

Visible cells with the same text style are retained as fixed-advance Canvas
runs. A typical row therefore uses one text texture and one Scene2D draw while
preserving exact terminal cell positions; ANSI style changes split only the
affected row into additional runs.

Runnable applications live in `Silex-Examples`: the
[dedicated system terminal](https://github.com/Matanek/Silex-Examples/blob/main/Sources/DedicatedSystemTerminal.sx),
[application command console](https://github.com/Matanek/Silex-Examples/blob/main/Sources/ApplicationCommandConsole.sx),
and [composed system terminal](https://github.com/Matanek/Silex-Examples/blob/main/Sources/ComposedSystemTerminal.sx).
Their package-level rendering and public-boundary checks live under `Tests/`.

See [Docs/README.md](Docs/README.md) for the supported VT subset and detailed
integration contract.
