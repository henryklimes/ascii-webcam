# ASCII Webcam

Real-time webcam-to-ASCII renderer. Single-file vanilla JS, no build, no dependencies. All processing is client-side via `getUserMedia` and `<canvas>`.


Per frame (`requestAnimationFrame`):

1. `drawImage(video, …)` into an off-screen canvas sized to `cols × rows` (downsample). `rows = cols * (videoH / videoW) * 0.5`; the `0.5` compensates for the ~2:1 height/width ratio of monospace cells.
2. `getImageData` → per-pixel luminance `Y = 0.299R + 0.587G + 0.114B`.
3. Adjust: `Y = (Y - 128) * contrast + 128 + brightness`, clamp to `[0,255]`, optional invert.
4. Map to a glyph: `ramp[(Y / 255) * (ramp.length - 1)]`.
5. `fillText` each glyph onto the output canvas. In color mode `fillStyle` is set to the source pixel `rgb()`; otherwise a single fixed color is used. Space glyphs are skipped.

Output is rasterized to `<canvas>` (not DOM text) so color mode and high column counts stay performant. Cell height is fixed at `12px`; cell width is `12 * 0.5`.

## Config (runtime, in-UI)

| Control      | Range / values                                         | State key      |
| ------------ | ------------------------------------------------------ | -------------- |
| Resolution   | 40–220 columns                                         | `resolution`   |
| Contrast     | 0.4–2.5                                                | `contrast`     |
| Brightness   | -100–100                                               | `brightness`   |
| Ramp         | standard, blocks, dense (70), minimal, binary, dots    | `ramp`         |
| Render mode  | mono, green, amber, color                              | `color`        |
| Mirror       | bool (default on)                                      | `mirror`       |
| Invert       | bool                                                   | `invert`       |

Ramps and color palettes are defined in the `RAMPS` and `COLORS` maps at the top of the script.

## Actions

- **Freeze** — halt the render loop on the current frame.
- **Snapshot** — `canvas.toDataURL("image/png")` → download.
- **Copy as text** — re-derives glyphs from the current frame and writes plain ASCII to the clipboard.

## Shortcuts

| Key     | Action          |
| ------- | --------------- |
| `Space` | Freeze / resume |
| `S`     | Snapshot        |
| `C`     | Copy as text    |

## Requirements

Browser with `getUserMedia` and Canvas 2D (Chrome, Firefox, Safari, Edge). Falls back to an error message if `mediaDevices.getUserMedia` is missing or permission is denied (`NotAllowedError`).

## Notes

No network I/O — frames are never uploaded or persisted. The camera track is stopped (`track.stop()`) on **Stop camera**.
