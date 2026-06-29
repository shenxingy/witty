# Changelog — witty fork

All notable changes **made by the witty fork on top of upstream Ghostty** are
documented here. Upstream Ghostty changes are not duplicated; see
[ghostty-org/ghostty](https://github.com/ghostty-org/ghostty) for those.

The format is based on [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

### Added

- **`altscreen:` keybinds now require *real* tmux, not just the alternate
  screen.** The alternate screen is shared by tmux, Neovim, `less`, `htop`,
  Claude Code, and every other fullscreen TUI, so firing tmux control bytes at
  all of them was wrong (e.g. `cmd+ctrl+=` leaking `C-b E` into Neovim). An
  `altscreen:` binding now fires only when the terminal is on the alternate
  screen **AND** a private DEC mode, **`?8771`** (`tmux_active`), is set. tmux
  announces itself by setting `?8771` from the shell prompt (through its
  passthrough); witty clears it on every alternate-screen transition so a stale
  value can't leak into the next TUI. If you don't set up the signal, the
  `altscreen:` keys simply never fire and every key keeps its normal binding —
  a safe, opt-in default. See
  [`docs/witty-altscreen.md`](docs/witty-altscreen.md#gating-on-real-tmux-not-just-the-alternate-screen)
  for the one-time `~/.tmux.conf` + shell setup.

### Fixed

- **`?8771` is re-armed on tmux *re-attach*.** witty clears the signal when the
  outer terminal enters the alternate screen, which a tmux client does on every
  attach. With `tmux new -As <name>` re-attaching to an existing session, the
  inner shell stays at its already-drawn prompt, so the per-prompt shell hook
  never re-fires and the `altscreen:` keybinds went dead until the next Enter.
  The documented setup now adds a tmux `client-attached` hook that re-emits
  `?8771h` straight to the attaching client's tty on every attach (raw, not
  passthrough-wrapped, since it bypasses tmux; it fires after tmux's smcup so it
  survives the enter-screen clear).
- **`?8771` is cleared on alternate-screen *entry* too, not only exit.** A
  stale or spoofed `?8771h` set while on the primary screen can no longer leak
  into a non-tmux fullscreen app launched directly (vim/less/Claude Code over
  plain SSH). Real tmux re-asserts the signal right after entry, so the gate
  re-arms on its own.
- **App-scope keybinds are looked up with `altscreen=false` so the tmux gate
  isn't bypassed.** Otherwise an app-scope lookup could match an `altscreen:`
  binding without the gate, firing tmux bytes outside tmux.
- **Stuck mouse tracking is cleared at each new shell prompt.** A fullscreen
  program that exited without disabling mouse reporting no longer leaves the
  shell in a state where clicks emit escape sequences.

### Docs

- Documented the `?8771` (`tmux_active`) gating, the `~/.tmux.conf` +
  shell-prompt setup, and the `client-attached` re-attach hook in
  [`docs/witty-altscreen.md`](docs/witty-altscreen.md), and folded the signal
  setup into the README's recommended remote-tmux workflow.

## [witty-v0.1.1] — 2026-06-19

### Fixed

- **`cmd+left` now jumps to the start of the line inside tmux.** It used to send
  C-a (`0x01`), which collides with a `C-a` tmux prefix and gets swallowed
  before reaching the shell. On the alternate screen it now sends Home
  (`CSI H`), which is independent of whatever tmux prefix is configured and is
  still "beginning of line" in readline/zle. On the primary screen it still
  falls back to C-a. (`cmd+right`/`cmd+backspace`/`alt+left`/`alt+right` were
  unaffected — they don't use the prefix byte.)

### Docs

- Added a "Recommended workflow" (persistent remote dev over ssh + tmux) and a
  "Platform support" section (including Windows) to the README.
- Clarified that the `cmd+ctrl+=` default is prefix-*dependent* (targets the
  C-b default prefix) and documented the one-line override for other prefixes
  such as `C-a`.

### Internal

- `cmd+ctrl+=` is now encoded as the escaped `\x02E` form to match the
  surrounding natural-text-editing bindings (identical bytes on the wire).

## [witty-v0.1.0] — 2026-06-19

Forked from upstream Ghostty at `49a91815`.

### Added

- **`altscreen:` keybind prefix.** Gate any keybinding to the terminal's
  alternate screen. On the primary screen the lookup falls back to whatever the
  key was already bound to, so the prefix overlays behavior rather than
  replacing it. See [`docs/witty-altscreen.md`](docs/witty-altscreen.md).
- **Default macOS `cmd+ctrl+=`.** `equalize_splits` on the primary screen;
  sends tmux `C-b E` (`select-layout -E`) to even out tmux panes on the
  alternate screen, where Ghostty can't resize tmux-owned panes.

### Fixed

- **Atomic-safe tmux sequence.** `cmd+ctrl+=` now sends `\x02E` (prefix + key)
  instead of `\x02:select-layout -E\r` (command prompt). Ghostty delivers a
  `text:` action as a single atomic pty write, which tmux's command prompt
  drops; the key-binding path handles it reliably. Verified end-to-end against
  real tmux (2-, 3-, and 4-pane layouts even out).
- **macOS menu key equivalents.** Altscreen-gated bindings are no longer
  dispatched to macOS menu key equivalents.
- **Trigger-chain continuation.** A non-matching altscreen gate continues the
  trigger lookup chain instead of aborting it, so the primary-screen fallback
  resolves correctly.
- **Key-table stack.** The key-table stack stores `Set` pointers directly
  rather than taking the address of a temporary.

### Commits

```
26a47fe fix(input): altscreen cmd+ctrl+= sends tmux prefix-E (atomic-safe), not command-prompt
9271e18 feat(input): altscreen cmd+ctrl+= sends tmux even-out, falls back to equalize_splits
9c3c459 fix(macos): don't dispatch altscreen-gated bindings to menu key equivalents
d972d63 fix(input): altscreen gate continues the trigger chain instead of aborting lookup
1805adb fix(surface): key table stack stores Set pointers — don't take their address
15e4d78 feat(input): altscreen bindings fall back to the replaced binding on primary screen
8103116 feat(input): add altscreen: keybind prefix
```

[witty-v0.1.1]: https://github.com/shenxingy/witty/releases/tag/witty-v0.1.1
[witty-v0.1.0]: https://github.com/shenxingy/witty/releases/tag/witty-v0.1.0
