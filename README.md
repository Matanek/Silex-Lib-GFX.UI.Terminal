# GFX.UI.Terminal

`GFX.UI.Terminal` adds terminal controls to `GFX.UI`. It provides two modes:
`Terminal.ConsoleView` for an application-owned command/log console and
`Terminal.View` for a real shell or TUI connected through PTY/ConPTY.

Both controls are ordinary `UI.CustomView` values. They participate in UI
composition, constraints, focus, keyboard/text routing and runtime
invalidation without placing terminal or UI state in an ECS `World`. The
package has no dependency on `GFX.ECS`, `GFX.Scene2D` or `GFX.Rendering`.

```sx
use GFX.UI
use GFX.UI.Terminal

func execute(command:str, console:&Terminal.Console) {
    console.respond("received: " + command)
}

func interface(console:Terminal.Console) UI.Column {
    var terminal = Terminal.ConsoleView(console, on_submit:execute)
    terminal.style.grow = 1.0
    let children:UI.View[] = [UI.Text("Console"), terminal]
    return UI.Column(children)
}
```

`Terminal.Session.update()` deliberately remains host-controlled: an
application polls its process, then calls `UI.Runtime.update(delta)` to detect
the changed screen revision. This keeps process scheduling outside the UI
tree and avoids introducing an application-world dependency.

Runnable examples:

- [ApplicationConsole.sx](Examples/ApplicationConsole.sx) hosts an
  application-owned command console.
- [SystemTerminal.sx](Examples/SystemTerminal.sx) hosts the platform shell and
  shows the explicit session/UI update boundary.

See [Docs/README.md](Docs/README.md) for the supported VT subset and detailed
integration contract.
