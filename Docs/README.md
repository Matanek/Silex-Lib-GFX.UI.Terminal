# Terminal sessions and views

`Terminal.spawn` starts a real terminal child through
`STD.Subprocess.spawn_terminal`. `Session.update` drains terminal output into
an inspectable `Screen`; `write` sends UTF-8 input and `resize` updates both the
emulator grid and PTY/ConPTY dimensions.

`Terminal.Console` is the complementary application-console mode. It owns no
process and grants no shell access: the host writes messages, accepts edited
input with `type`/`backspace`, interprets the string returned by `submit`, then
publishes a response and the next prompt with `respond`. Both modes expose the
same `Screen`, so themes and view code stay shared.

The first release recognizes printable UTF-8, carriage return, line feed,
backspace, tab, CSI cursor movement/positioning, erase-in-display,
erase-in-line and the standard 16 ANSI foreground/background colors. Unknown
control sequences are ignored without becoming visible text. Alternate screen,
scroll regions, OSC links, selection, clipboard, mouse protocols and grapheme
width are explicit future extensions.

`drawing(screen, settings)` returns one ordinary `GFX.Canvas.Canvas` and stays
useful for snapshots or small, infrequently updated consoles. An interactive
terminal should instead retain a `Terminal.View`. Its first `changed_rows`
call returns every row; later calls return only rows whose cells or cursor
changed. The application keeps one `GFX.Scene2D.Canvas` per row and replaces
those returned by `changed_rows` with `line_drawing`. A separate
`background_drawing` supplies the terminal backdrop.

```sx
var view = Terminal.View(Terminal.ViewSettings())

// Create the background and one Scene2D Canvas per initial row.
var background = view.background_drawing(screen)
for row in view.changed_rows(screen) {
    var line = view.line_drawing(screen, row)
}

// After Session.update(), replace only these row canvases.
for row in view.changed_rows(screen) {
    var line = view.line_drawing(screen, row)
}
```

This keeps typing cost proportional to the changed line rather than to all
visible text. Scrolling still invalidates every row because the entire visible
grid genuinely changes. `width`, `height`, `row_height` and `row_y` provide the
layout values needed to place the retained Canvas entities. The view uses
`Canvas.Font.monospace()` by default; `Theme`, a custom font, cell dimensions,
padding and cursor visibility remain application-controlled.

`Screen.revision()` remains available for integrations that retain a single
drawing. `Terminal.View` consumes finer internal row revisions and is the
recommended path for latency-sensitive consoles and terminals.

The [application console](../Examples/ApplicationConsole.sx) and
[system terminal](../Examples/SystemTerminal.sx) examples show the complete
window, input, Scene2D Canvas and incremental redraw integration.
