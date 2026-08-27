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
Viewer.show(UI.Session(terminal))
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

`Terminal.Shell` owns the platform process, its PTY/ConPTY session, nonblocking
polling and cleanup. Both modes invalidate through their surrounding
`UI.Runtime`; `UI.Session` supplies that behavior to any `Canvas.Session`
presenter. None of these layers introduces an application-world dependency.

Runnable examples:

- [QuickStart.sx](Examples/QuickStart.sx) opens the minimal interactive
  terminal window.
- [ApplicationConsole.sx](Examples/ApplicationConsole.sx) hosts an
  application-owned command console.
- [SystemTerminal.sx](Examples/SystemTerminal.sx) hosts the platform shell and
  composes it with additional UI.

See [Docs/README.md](Docs/README.md) for the supported VT subset and detailed
integration contract.
