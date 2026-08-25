# GFX.Terminal

`GFX.Terminal` provides a customizable terminal emulator for GFX windows. It
uses the portable PTY/ConPTY child transport owned by `STD` and renders its
screen through `GFX.Canvas`; it never embeds Terminal.app, Windows Terminal or
another operating-system terminal window.

The package has two explicit modes: `Terminal.Console` for an
application-owned command/log console, and `Terminal.Session` for a real
system shell or TUI connected through PTY/ConPTY. Both share the same screen,
theme and Canvas renderer. The renderer uses the bundled
`Canvas.Font.monospace()` by default and accepts any custom `Canvas.Font`.

```sx
use GFX.Terminal

let command = Terminal.Command(
    executable:"/bin/zsh",
    arguments:["-l"],
    current_directory:null,
    inherit_environment:true,
    environment:[],
    size:Terminal.Size(columns:100, rows:30)
)
var session = try Terminal.spawn(command)
var drawing = Terminal.drawing(session.screen())
```

Runnable examples:

- [ApplicationConsole.sx](Examples/ApplicationConsole.sx) opens an
  application-owned command console.
- [SystemTerminal.sx](Examples/SystemTerminal.sx) opens the platform shell in
  a live GFX window and routes text, control keys, arrows and terminal output.

See [Docs/README.md](Docs/README.md) for the supported VT subset and the
integration boundary.
