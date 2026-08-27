# Terminal controls

`GFX.UI.Terminal` is an extension of `GFX.UI`, not a rendering-world plugin.
Its `Terminal.Shell` and `Terminal.Console` subclasses of `UI.CustomView`
compose anywhere a `UI.View` is accepted. `UI.Runtime` owns their retained
identity, layout, focus, text/key routing, paint dispatch and invalidation.
No terminal state is inserted into an ECS `World`.

## Application console

`Terminal.Console` owns no process and grants no shell access. The application
declares named commands; the console edits and parses the input, dispatches the
matching action and prints its returned response:

```sx
use GFX.UI
use GFX.UI.Terminal

func status(arguments:str) str { return "application is running" }
func echo(arguments:str) str { return arguments }

var console = Terminal.Console(
    commands:[
        Terminal.Command("status", status, description:"Show application status"),
        Terminal.Command("echo", echo, description:"Print the provided text")
    ],
    prompt:"silex> ",
    welcome:["type 'help' to list commands"]
)
var session = UI.Session(console)
```

`Terminal.Command` accepts one word as its name, an action
`func(arguments:str) str`, and an optional description. `help` lists the
descriptions, `clear` resets the display, duplicate command names are rejected
and an unknown name produces a local diagnostic. Returning an empty string
simply restores the prompt after an action with no textual result.

## System terminal

`Terminal.Shell()` starts the conventional interactive shell for the active
platform and is itself a UI view. Its options control terminal dimensions,
appearance, runtime error handling and exit notification without exposing the
child process or its transport.

The shell drains pending process output non-blockingly when
`UI.Runtime.update(delta)` visits its custom content. The usual interactive
path therefore contains no process loop of its own:

```sx
var terminal = Terminal.Shell()
Viewer.show(UI.Session(terminal))
```

The PTY/ConPTY session, process lifecycle, writes, resize synchronization and
polling are implementation details of `Shell`. Automatic polling is part of
the terminal content update, not an Application system, so neither GFX.UI nor
GFX.UI.Terminal depends on ECS or a game world.

## Rendering and emulation

Both controls share `Screen`, `Theme`, `ViewSettings` and the Canvas renderer.
They use `Canvas.Font.monospace()` by default and accept a custom font. The
control measures itself from terminal rows, columns, cell dimensions and
padding; parent UI constraints determine the allocated frame. Painting is
limited to the rows and columns visible in that frame.

`drawing(screen, settings)` remains available for static snapshots outside a
UI tree. Interactive applications should use `Terminal.Shell` or
`Terminal.Console`, whose content revision invalidates `UI.Runtime`.

The current emulator recognizes printable UTF-8, carriage return, line feed,
backspace, tab, CSI cursor movement/positioning, erase-in-display,
erase-in-line and the standard 16 ANSI foreground/background colors. Unknown
control sequences are ignored without becoming visible text. Alternate
screen, scroll regions, OSC links, selection, clipboard, mouse protocols and
grapheme width remain future extensions.

The [quick start](../Examples/QuickStart.sx) opens a minimal interactive
terminal window. The [application console](../Examples/ApplicationConsole.sx) and
[system terminal](../Examples/SystemTerminal.sx) examples contain complete
`UI.Session` values presented by `GFX.Viewer`; neither creates, receives or stores
terminal state in an ECS world.
