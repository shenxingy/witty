# Changelog — witty fork

All notable changes **made by the witty fork on top of upstream Ghostty** are
documented here. Upstream Ghostty changes are not duplicated; see
[ghostty-org/ghostty](https://github.com/ghostty-org/ghostty) for those.

The format is based on [Keep a Changelog](https://keepachangelog.com/).

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

[witty-v0.1.0]: https://github.com/shenxingy/witty/releases/tag/witty-v0.1.0
