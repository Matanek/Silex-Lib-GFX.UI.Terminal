# Terminal controls

`GFX.UI.Terminal` is an extension of `GFX.UI`, not a rendering-world plugin.
Its `Terminal.View` and `Terminal.ConsoleView` subclasses of `UI.CustomView`
compose anywhere a `UI.View` is accepted. `UI.Runtime` owns their retained
identity, layout, focus, text/key routing, paint dispatch and invalidation.
No terminal state is inserted into an ECS `World`.

## Application console

`Terminal.Console` owns no process and grants no shell access. The host writes
messages, receives edited input through `ConsoleView`, handles the submitted
command and publishes a response:

```sx
use GFX.UI
use GFX.UI.Terminal

func execute(command:str, console:&Terminal.Console) {
    if command == "clear" { console.clear() }
    else { console.respond("unknown command: " + command) }
}

var console = Terminal.Console(prompt:"silex> ")
var view = Terminal.ConsoleView(console, on_submit:execute)
var runtime = UI.Runtime(view)
```

## System terminal

`Terminal.spawn` starts a terminal child through
`STD.Subprocess.spawn_terminal`. `Session.update` drains output into an
inspectable `Screen`; `write` sends UTF-8 input and `resize` updates both the
emulator grid and PTY/ConPTY dimensions.

Process polling stays explicit. A host normally performs these two operations
in order:

```sx
match session.update() {
    failure(error) => { panic(error.detail) }
    success(status) => {}
}
let redraw = runtime.update(delta)
```

`Terminal.View` never calls `Session.update()` itself. This separation lets a
game, editor or event-driven tool choose its own process scheduling without
coupling `GFX.UI` to `Application`, ECS or a game world.

## Rendering and emulation

Both controls share `Screen`, `Theme`, `ViewSettings` and the Canvas renderer.
They use `Canvas.Font.monospace()` by default and accept a custom font. The
control measures itself from terminal rows, columns, cell dimensions and
padding; parent UI constraints determine the allocated frame. Painting is
limited to the rows and columns visible in that frame.

`drawing(screen, settings)` remains available for static snapshots outside a
UI tree. Interactive applications should use `Terminal.View` or
`Terminal.ConsoleView`, whose content revision invalidates `UI.Runtime`.

The current emulator recognizes printable UTF-8, carriage return, line feed,
backspace, tab, CSI cursor movement/positioning, erase-in-display,
erase-in-line and the standard 16 ANSI foreground/background colors. Unknown
control sequences are ignored without becoming visible text. Alternate
screen, scroll regions, OSC links, selection, clipboard, mouse protocols and
grapheme width remain future extensions.

The [application console](../Examples/ApplicationConsole.sx) and
[system terminal](../Examples/SystemTerminal.sx) examples contain complete
`UI.Runtime` hosts built on `GFX.Viewer`; neither creates, receives or stores
terminal state in an ECS world.
