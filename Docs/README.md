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
Viewer.show(UI.Session(terminal, autofocus:true))
```

The PTY/ConPTY transport, writes, resize synchronization and polling are
implementation details of `Shell`. Automatic polling is part of the terminal
content update, not an Application system, so neither GFX.UI nor
GFX.UI.Terminal depends on ECS or a game world.

For a one-terminal window, `Terminal.Shell()` owns that lifecycle implicitly.
Tabbed applications can instead retain a `Terminal.ShellSession` and pass it
to the view with `Terminal.Shell(session:process, on_exit:...)`.
`process.close()` terminates that shell only. The `on_exit` callback lets the
application remove the corresponding tab and decide whether an empty tab set
should also close its window. Terminal never makes that window-level decision.
After an exit is observed, the view no longer accepts input and discards late
key events without treating the expected closed input stream as a runtime
error.

Initial keyboard focus is opt-in through
`UI.Session(terminal, autofocus:true)`. `Terminal.Shell` forwards `Ctrl+C` to
the child shell by default, which is the expected behavior for full terminal
and multi-tab applications. A single-session window may instead use
`close_on_interrupt:true`; that terminates only its `ShellSession`. The
application remains responsible for removing the tab or closing the window,
typically from `on_exit` and an application/viewer close predicate.

## Rendering and emulation

Both controls share `Screen`, `Theme`, `ViewSettings` and the Canvas renderer.
They use `Canvas.Font.monospace()` by default and accept a custom font. The
control measures itself from terminal rows, columns, cell dimensions and
padding; parent UI constraints determine the allocated frame. Painting is
limited to the rows and columns visible in that frame.

Adjacent cells with the same foreground and weight are coalesced into
fixed-advance Canvas runs. This preserves custom cell dimensions and exact
column positions while bounding retained text textures and Scene2D draws by
visible style runs rather than visible characters.

`drawing(screen, settings)` remains available for static snapshots outside a
UI tree. Interactive applications should use `Terminal.Shell` or
`Terminal.Console`, whose content revision invalidates `UI.Runtime`.

The current emulator recognizes printable UTF-8, carriage return, line feed,
backspace, tab, CSI cursor movement/positioning, erase-in-display,
erase-in-line and the standard 16 ANSI foreground/background colors. Unknown
control sequences are ignored without becoming visible text. Alternate
screen, scroll regions, OSC links, selection, clipboard, mouse protocols and
grapheme width remain future extensions.

The centralized [dedicated system terminal](https://github.com/Matanek/Silex-Examples/blob/main/Sources/DedicatedSystemTerminal.sx)
opens the minimal interactive terminal window. The centralized
[application command console](https://github.com/Matanek/Silex-Examples/blob/main/Sources/ApplicationCommandConsole.sx)
and [composed system terminal](https://github.com/Matanek/Silex-Examples/blob/main/Sources/ComposedSystemTerminal.sx)
provide the complete `UI.Session` applications. Package checks live under
`Tests/`; none of these paths creates, receives or stores terminal state in an
ECS world.
