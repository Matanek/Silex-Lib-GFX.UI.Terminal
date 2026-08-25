# Terminal sessions and views

`Terminal.spawn` starts a real terminal child through
`STD.Subprocess.spawn_terminal`. `Session.update` drains terminal output into
an inspectable `Screen`; `write` sends UTF-8 input and `resize` updates both the
emulator grid and PTY/ConPTY dimensions.

`Terminal.Console` is the complementary application-console mode. It owns no
process and grants no shell access: the host writes messages, accepts edited
input with `type`/`backspace`, and interprets the string returned by `submit`.
Both modes expose the same `Screen`, so themes and view code stay shared.

The first release recognizes printable UTF-8, carriage return, line feed,
backspace, tab, CSI cursor movement/positioning, erase-in-display,
erase-in-line and the standard 16 ANSI foreground/background colors. Unknown
control sequences are ignored without becoming visible text. Alternate screen,
scroll regions, OSC links, selection, clipboard, mouse protocols and grapheme
width are explicit future extensions.

`drawing(screen, settings)` returns ordinary `GFX.Canvas.Canvas` content. An
IDE may place it in a `GFX.Scene2D.Canvas`, refresh it after `Session.update`,
and route `GFX.Input` text/key events to `Session.write`. `Theme`, font, cell
dimensions, padding and cursor visibility remain application-controlled.
