<!-- LOGO -->
<h1 align="center">witty</h1>

<p align="center">
  A <a href="https://github.com/ghostty-org/ghostty">Ghostty</a> fork with
  <b>altscreen-aware keybindings</b>.
  <br />
  Bind one key to do one thing normally — and something else the moment a
  fullscreen TUI (tmux, Neovim, less) takes over the screen.
</p>

<p align="center">
  <a href="#what-witty-adds">What witty adds</a>
  ·
  <a href="#quick-example">Quick example</a>
  ·
  <a href="docs/witty-altscreen.md">Docs</a>
  ·
  <a href="#install">Install</a>
  ·
  <a href="#built-on-ghostty">Upstream</a>
</p>

---

witty is [Ghostty](https://github.com/ghostty-org/ghostty) — the fast, native,
feature-rich terminal emulator — plus **one** focused addition: the
`altscreen:` keybind prefix. Everything else is stock Ghostty, merged from
upstream regularly. If you don't use the new prefix, witty behaves exactly like
Ghostty.

## Contents

- [What witty adds](#what-witty-adds)
- [Quick example](#quick-example)
- [How it works](#how-it-works)
- [Install](#install)
- [Built on Ghostty](#built-on-ghostty)
- [License](#license)

## What witty adds

A terminal has two screens: the **primary** screen (your shell, scrollback) and
the **alternate** screen, which fullscreen programs like tmux, Neovim, `less`,
or `htop` switch to while running. witty lets a single keybinding behave
differently depending on which one is active.

| Addition | What it does |
| --- | --- |
| **`altscreen:` keybind prefix** | Gate any binding to the **alternate screen** only. On the primary screen the lookup falls back to whatever that key was bound to before, so you lose nothing. |
| **Default `cmd+ctrl+=` (macOS)** | On the primary screen: `equalize_splits` (balance Ghostty's own splits). On the alternate screen: send tmux `C-b E` (`select-layout -E`) so the **tmux panes** get evened out instead — because Ghostty can't resize panes that belong to tmux. |

## Quick example

```ini
# ~/.config/ghostty/config

# ctrl+r reloads the config on the normal screen,
# but enters tmux copy-mode (C-b [) when tmux is in the foreground:
keybind = ctrl+r=reload_config
keybind = altscreen:ctrl+r=text:\x02[

# The macOS default that ships with witty — one key, two behaviors:
keybind = super+ctrl+equal=equalize_splits
keybind = altscreen:super+ctrl+equal=text:\x02E   # C-b E → even out tmux panes
```

`\x02` is tmux's default prefix (`C-b`). See
**[docs/witty-altscreen.md](docs/witty-altscreen.md)** for the full syntax,
fallback rules, the why-not-`:select-layout` story, and caveats (custom tmux
prefixes, non-tmux fullscreen apps).

## How it works

The `altscreen:` prefix registers a binding on a separate, alternate-screen key
table. On a key event, witty first consults that table when the terminal is on
the alternate screen; if there's no match (or you're on the primary screen) it
continues the normal lookup chain and falls back to the binding the key already
had. The prefix never *adds* a key — it overlays behavior on a key you've
already bound, which is what makes the tmux/non-tmux split clean.

Full design notes, fallback semantics, and edge cases live in
**[docs/witty-altscreen.md](docs/witty-altscreen.md)**.

## Install

witty is currently macOS-focused (the shipped default binding is macOS). Linux
(GTK) builds the same way as upstream Ghostty.

**Prebuilt macOS app** — grab the latest `Ghostty.app` (Apple Silicon,
ad-hoc signed) from the [Releases](https://github.com/shenxingy/witty/releases)
page, unzip, and drop it into `/Applications`.

**Build from source** — requires [Zig](https://ziglang.org) `0.15.2` (see
`.zig-version` / `minimum_zig_version` in `build.zig.zon`):

```sh
# core build (and the underlying library)
zig build

# run the test suite (prefer a filter; the full suite is slow)
zig build test -Dtest-filter="<name>"
```

macOS app bundle (needs Xcode + gettext on `PATH` for `msgfmt`):

```sh
# 1) rebuild the xcframework with your changes
zig build -Demit-xcframework=true -Demit-macos-app=false -Dapp-runtime=none -Doptimize=ReleaseFast
# 2) build the app
xcodebuild -project macos/Ghostty.xcodeproj -scheme Ghostty \
  -configuration ReleaseLocal SYMROOT="$PWD/macos/build" build
# -> macos/build/ReleaseLocal/Ghostty.app
```

See [`CONTRIBUTING.md`](CONTRIBUTING.md) and [`HACKING.md`](HACKING.md) (from
upstream) for the full developer workflow.

## Built on Ghostty

witty is a fork of **[ghostty-org/ghostty](https://github.com/ghostty-org/ghostty)**
by Mitchell Hashimoto and the Ghostty contributors. All of Ghostty's
documentation, downloads, and terminal behavior apply unchanged:

- 🌐 Website & downloads: <https://ghostty.org>
- 📚 Documentation: <https://ghostty.org/docs>
- 💬 About Ghostty: <https://ghostty.org/docs/about>

The complete original project README is preserved below.

<details>
<summary><b>Original Ghostty README</b></summary>

## About

Ghostty is a terminal emulator that differentiates itself by being
fast, feature-rich, and native. While there are many excellent terminal
emulators available, they all force you to choose between speed,
features, or native UIs. Ghostty provides all three.

**`libghostty`** is a cross-platform, zero-dependency C and Zig library
for building terminal emulators or utilizing terminal functionality
(such as style parsing). Anyone can use `libghostty` to build a terminal
emulator or embed a terminal into their own applications. See
[Ghostling](https://github.com/ghostty-org/ghostling) for a minimal complete project
example or the [`examples` directory](https://github.com/ghostty-org/ghostty/tree/main/example)
for smaller examples of using `libghostty` in C and Zig.

For more details, see [About Ghostty](https://ghostty.org/docs/about).

### Download

See the [download page](https://ghostty.org/download) on the Ghostty website.

### Documentation

See the [documentation](https://ghostty.org/docs) on the Ghostty website.

### Contributing and Developing

If you have any ideas, issues, etc. regarding Ghostty, or would like to
contribute to Ghostty through pull requests, please check out our
["Contributing to Ghostty"](CONTRIBUTING.md) document. Those who would like
to get involved with Ghostty's development as well should also read the
["Developing Ghostty"](HACKING.md) document for more technical details.

### Roadmap and Status

Ghostty is stable and in use by millions of people and machines daily.

The high-level ambitious plan for the project, in order:

|  #  | Step                                                    | Status |
| :-: | ------------------------------------------------------- | :----: |
|  1  | Standards-compliant terminal emulation                  |   ✅   |
|  2  | Competitive performance                                 |   ✅   |
|  3  | Rich windowing features -- multi-window, tabbing, panes |   ✅   |
|  4  | Native Platform Experiences                             |   ✅   |
|  5  | Cross-platform `libghostty` for Embeddable Terminals    |   ✅   |
|  6  | Ghostty-only Terminal Control Sequences                 |   ❌   |

For the full roadmap detail, crash-reporter docs, and everything else, see the
upstream README at
[ghostty-org/ghostty](https://github.com/ghostty-org/ghostty#readme).

</details>

## License

Same as upstream Ghostty — see [LICENSE](LICENSE) (MIT).
