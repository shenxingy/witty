# witty icon source

witty reuses Ghostty's Icon Composer document (`images/Ghostty.icon`) — the
premium screen/bevel/gloss/effects frame — but with witty's own identity:

- **Glyph:** `witty_mark.svg` (a bold `W` + terminal-prompt cursor underscore)
  replaces the ghost layer (`Ghostty.icon/Assets/Ghostty.png`, 475×541, white
  with a soft glow). Render: `rsvg-convert -w 475 -h 541 witty_mark.svg`, then
  composite a Gaussian-blurred white halo behind it.
- **Color:** the blue `Screen.png` and the in-app `AppIconImage` tiles are
  hue-shifted +55° (blue → violet), and the ghost-glow fill in `icon.json`
  (`GhosttyBlur`) is set to the matching violet (`extended-srgb:0.438,0,1`).

Bundle ID and the `ghostty` CLI are intentionally unchanged (see
`docs/witty-altscreen.md` / CHANGELOG) so upstream merges stay trivial.
