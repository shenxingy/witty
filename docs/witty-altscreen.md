# The `altscreen:` keybind prefix

[← Back to README](../README.md)

witty adds a single keybind prefix, `altscreen:`, that gates a binding to the
terminal's **alternate screen**. This document covers the syntax, the fallback
semantics, the macOS `cmd+ctrl+=` default that ships with witty, and the
caveats you should know.

## Contents

- [Background: the two screens](#background-the-two-screens)
- [Syntax](#syntax)
- [Fallback semantics](#fallback-semantics)
- [The `cmd+ctrl+=` default](#the-cmdctrl-default)
- [Why `C-b E` and not `:select-layout`](#why-c-b-e-and-not-select-layout)
- [The `cmd+left` default](#the-cmdleft-default)
- [Caveats](#caveats)
- [Examples](#examples)

## Background: the two screens

Terminals expose two screen buffers:

- The **primary** screen — your shell prompt and scrollback.
- The **alternate** screen — a separate buffer that fullscreen programs switch
  to while they run: `tmux`, `vim`/`nvim`, `less`, `htop`, `fzf`, and so on.
  When the program exits, the terminal switches back and your shell is exactly
  as you left it.

A keybinding that's useful at the shell is often useless (or wrong) inside a
fullscreen TUI, and vice-versa. The `altscreen:` prefix lets one physical key
serve both contexts.

## Syntax

Prefix any keybind trigger with `altscreen:`:

```ini
keybind = altscreen:<trigger>=<action>
```

The binding is only consulted while the terminal is on the alternate screen.
Like `performable:`, an `altscreen:` keybind is **not** registered as a global
shortcut and is matched only against live key events.

```ini
# Example from Ghostty's own docs for the prefix:
keybind = altscreen:cmd+d=text:\x01|
```

## Fallback semantics

The defining behavior of `altscreen:` is the **fallback**:

1. On a key event, witty first checks the alternate-screen table **if** the
   terminal is currently on the alternate screen.
2. If there's no alternate-screen match — or you're on the primary screen — the
   lookup **continues down the normal chain** and resolves to whatever that key
   was already bound to.

So `altscreen:` never *adds* a new key; it **overlays** an alternate-screen
behavior on top of an existing binding. This is what makes the pattern clean:

```ini
keybind = ctrl+r=reload_config           # primary screen: reload config
keybind = altscreen:ctrl+r=text:\x02[    # alt screen: send tmux C-b [
```

Press `ctrl+r` at the shell → config reloads. Press it inside tmux → tmux enters
copy-mode. One key, context-aware, with no loss of the original binding.

> Note on `physical:` — the macOS default below binds the **physical** `=` key
> (`physical:equal`) so the alternate-screen override and the primary-screen
> `equalize_splits` can coexist on the same physical key regardless of layout.

## The `cmd+ctrl+=` default

witty ships one default that uses the prefix (macOS):

```ini
keybind = super+ctrl+equal=equalize_splits          # primary screen
keybind = altscreen:super+ctrl+equal=text:\x02E     # alternate screen
```

- **Primary screen** → `equalize_splits` balances Ghostty's own split panes.
- **Alternate screen** → witty sends `\x02E` to the pty. That's tmux's default
  prefix `C-b` (`0x02`) followed by `E`, which is tmux's built-in
  `prefix E` binding for `select-layout -E` ("spread the current layout's panes
  out evenly"). The tmux prefix is intercepted by tmux regardless of what's
  running in the focused pane.

The reason for the split: when a single Ghostty surface is running tmux, the
panes you see belong to **tmux**, not Ghostty. `equalize_splits` would be a
no-op because Ghostty has exactly one surface. So on the alternate screen witty
asks tmux to even out *its* panes instead.

## Why `C-b E` and not `:select-layout`

An earlier version sent `\x02:select-layout -E\r` — the tmux **command prompt**
(`C-b :`) followed by the command and Enter. That turns out to be unreliable:
Ghostty delivers a `text:` action as a **single atomic write** to the pty, and
tmux's command prompt isn't ready for the bytes that follow `:` in that same
read, so the typed command gets dropped.

`C-b E` goes through tmux's normal **key-binding** path instead of the command
prompt, and acts on the whole atomic write reliably. It's also shorter (2 bytes)
and `select-layout -E` preserves your split structure (rather than forcing a
`tiled` grid), matching the spirit of Ghostty's own `equalize_splits`.

This was verified end-to-end against real tmux: feeding `\x02E` as one write to
a real tmux client evens out 2-, 3-, and 4-pane layouts; the atomic
command-prompt form does not.

## The `cmd+left` default

witty ships a second altscreen default (macOS) for "jump to the start of the
line":

```ini
keybind = super+arrow_left=text:\x01          # primary screen: C-a
keybind = altscreen:super+arrow_left=csi:H    # alternate screen: Home (CSI H)
```

The problem it solves: `cmd+left` normally sends **C-a** (`0x01`), which readline
treats as beginning-of-line. But `C-a` is also a very common **tmux prefix**
(`set -g prefix C-a`, screen-style). When tmux is running, your `cmd+left` C-a is
intercepted by tmux as its prefix and never reaches the shell — so the cursor
doesn't move (and the next key gets eaten as a tmux command).

On the alternate screen witty instead sends **Home** (`CSI H` = `ESC [ H`).
Unlike a control byte, Home isn't anybody's tmux prefix, so it passes straight
through to the shell, where readline/zle still map it to beginning-of-line. On
the primary screen (a bare shell) `cmd+left` keeps sending C-a, which is the most
universally understood "start of line".

This fix is **prefix-independent**: it works whether your tmux prefix is `C-a`,
`C-b`, or anything else. (`cmd+right` = C-e, `cmd+backspace` = C-u, and the
`alt+←/→` word motions don't use the prefix byte, so they already work in tmux
and are left unchanged.)

## Caveats

- **Assumes tmux's default prefix `C-b`.** If you remapped your prefix to
  `C-a`, change `\x02` to `\x01` in your binding. Other prefixes need the
  corresponding control byte.
- **Assumes tmux's default `E` binding** (`select-layout -E`). If you unbound or
  rebound `E` in tmux, adjust accordingly.
- **Fires on *any* alternate-screen app, not just tmux.** The prefix only knows
  "alternate screen," not "tmux specifically." Pressing the default inside
  `vim`/`less` sends `C-b E` (a harmless page-up + word-motion there), not a
  tmux command. Bind alternate-screen overrides to keys you won't fat-finger in
  other TUIs.
- **Linux/GTK:** the prefix works the same, but witty ships no default binding
  that uses it on Linux — add your own.

## Examples

```ini
# tmux copy-mode from a shell-friendly key
keybind = ctrl+r=reload_config
keybind = altscreen:ctrl+r=text:\x02[

# even out tmux panes (witty's macOS default)
keybind = super+ctrl+equal=equalize_splits
keybind = altscreen:super+ctrl+equal=text:\x02E

# C-a prefix users: send C-a E instead of C-b E
keybind = altscreen:super+ctrl+equal=text:\x01E

# new tmux window on a key that opens a Ghostty tab on the primary screen
keybind = cmd+t=new_tab
keybind = altscreen:cmd+t=text:\x02c
```

---

[← Back to README](../README.md)
