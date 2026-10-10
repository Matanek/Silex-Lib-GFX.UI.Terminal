# Compose terminals with GFX.UI.Terminal

`GFX.UI.Terminal` extends `GFX.UI` with two intentions: `Terminal.Shell()`
communicates with the operating system, while `Terminal.Console()` runs
commands owned by the application.

Both are `UI.CustomView` values. They participate in composition, constraints,
focus, keyboard/text routing, and invalidation without placing their state in
an ECS world. The package depends on neither `GFX.ECS`, `GFX.Scene2D`, nor
`GFX.Rendering`.

[Lire cette documentation en français.](../FR/README.md)

## Install the package

```text
silex install GFX.UI.Terminal
```

This development revision requires the toolchain with `try` capture for
`Result` operations and extraction on ordinary calls.

## Open a system terminal

```sx
use GFX.UI
use GFX.UI.Terminal
use GFX.Viewer

var terminal = Terminal.Shell()
Viewer.show(UI.Session(terminal, autofocus:true))
```

`Terminal.Shell` owns the process and lifecycle by default. Its PTY or ConPTY
transport, writes, nonblocking polling, and cleanup remain internal.
`UI.Runtime.update(delta)` drains output when it visits the content; no process
loop or ECS system is required.

`autofocus:true` immediately focuses the first text-input control. `Ctrl+C` is
sent to the shell by default. A single-session window can choose
`close_on_interrupt:true`, which closes only the `ShellSession`.

## Declare an application console

`Terminal.Console` owns no process and grants no shell access. The application
declares its commands:

```sx
use GFX.UI
use GFX.UI.Terminal

func status(arguments:str) str { return "application is running" }
func echo(arguments:str) str { return arguments }

var console = Terminal.Console(
    commands:[
        Terminal.Command("status", status, description:"Show application status"),
        Terminal.Command("echo", echo, description:"Print the provided text"),
    ],
    prompt:"silex> ",
    welcome:["type 'help' to list commands"],
)
var session = UI.Session(console)
```

`Terminal.Command` accepts a one-word name, a `func(arguments:str) str` action,
and an optional description. `help` and `clear` are built in. Duplicates are
rejected, an unknown name produces a local diagnostic, and an empty response
simply restores the prompt.

## Own several sessions

A tabbed application can make the lifecycle explicit:

```sx
var process = Terminal.ShellSession()
var terminal = Terminal.Shell(
    session:process,
    on_exit:func() {
        // Remove the corresponding tab here.
    },
)

match (try process.close()) {
    failure(error) => { panic(error.operation + ": " + error.detail) }
    success => {}
}
```

`ShellSession.close()` terminates only that shell. `on_exit` runs once after
exit is observed, whether requested or initiated by the shell. The view ignores
late events instead of turning them into closed-pipe errors. The application
separately decides whether to remove the tab or close its window.

## Understand rendering and emulation

Both controls share `Screen`, `Theme`, `ViewSettings`, and the Canvas renderer.
They use `Canvas.Font.monospace()` by default. Parent constraints determine the
allocated frame and painting is limited to visible cells.

Adjacent cells with the same color and weight are coalesced into fixed-advance
Canvas runs. Column positions remain exact, and text textures and Scene2D draws
scale with style changes rather than character count.

The emulator recognizes printable UTF-8, carriage return, line feed,
backspace, tab, CSI movement and positioning, display or line erase, and the
standard 16 ANSI colors. Unknown sequences are ignored. Alternate screen,
scroll regions, OSC links, selection, clipboard, mouse protocols, and grapheme
width remain future extensions.

The [DedicatedSystemTerminal](https://github.com/Matanek/Silex-Examples/blob/main/Sources/DedicatedSystemTerminal.sx),
[ApplicationCommandConsole](https://github.com/Matanek/Silex-Examples/blob/main/Sources/ApplicationCommandConsole.sx),
and [ComposedSystemTerminal](https://github.com/Matanek/Silex-Examples/blob/main/Sources/ComposedSystemTerminal.sx)
applications belong to Silex-Examples. The
[TerminalScreenRendering](https://github.com/Matanek/Silex-Benchmarks/blob/main/Sources/TerminalScreenRendering.sx)
benchmark belongs to Silex-Benchmarks.
