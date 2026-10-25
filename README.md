# ASCII Darkroom

Real-time ASCII art from your webcam, or from any image you drop in. 35 render styles,
15 palettes, 8 stackable post effects, PNG and plain-text export.

One HTML file. No build step, no bundler, no dependencies except two Google Fonts.

**[Live demo →](https://biryaniji.github.io/ascii-art-generator/)**

---

## Features

- **Live camera** at 30–60fps, with a front/back toggle and a freeze button
- **Any image or video** — click Upload, drag a file onto the stage, or just paste
- **35 styles** across glyphs, line work, dots, tiles and raster
- **15 palettes** — Phosphor, Amber, Noir, Paper, C64, PICO-8, Game Boy, Vapour,
  Blueprint, Sepia, Ice, Ember, Thermal, Neon, Kraft
- **26 one-click recipes** — Matrix code, Film noir, Amber CRT, Woodcut, Banknote,
  Topographic, Pop art, Oil portrait, Old film and more
- **Hover any style** to preview it on your own image before committing
- **8 post effects** — scanlines, bloom, RGB split, film grain, vignette, glitch,
  film dust, bleed
- **Full-screen mode** — `f`, the button, or a double-click on the canvas; `esc` or
  a tap to leave
- **Responsive** — single-column below 880px, works on phones and tablets
- **Export** PNG at 1×/2×/4×, or the raw characters as `.txt` / clipboard
- Runs entirely in the browser. Nothing is uploaded anywhere.

## Styles

| Group | Styles |
|---|---|
| **Text** | ASCII · Dense · Matrix · Blocks · Half block · Quadrant · Braille · Edge · Glitch |
| **Line** | Sketch · Crosshatch · Engraving · Contour · Truchet · Diagonal · Lines · Spiral · Ripple |
| **Dot** | Halftone · Stipple · Cross · Diamond · LED · CMYK |
| **Tile** | Mosaic · Hex · Bricks · Voxel · Weave · Stained · Triangles |
| **Raster** | Pixel · Dither · Oil paint · Warhol |

Twelve character sets (Standard, Dense 70, Minimal, Block shades, Vertical fill,
Binary, Katakana, Hex, Stipple, Slashes, Runes, Terminal) can be swapped into any of
the glyph styles.

## How it works

Every frame runs the same four steps:

1. **Downsample.** The frame is drawn into a canvas exactly as wide as the character
   grid — ask for 110 columns and it draws into a 110px-wide canvas. The browser's
   native scaler does the box-averaging, which is why this is fast.
2. **Luminance.** Each pixel becomes one number via the Rec. 709 weights
   (`0.2126R + 0.7152G + 0.0722B`), then brightness, contrast, gamma and invert
   are applied.
3. **Look up.** `Math.round(brightness × (ramp.length - 1))` picks a character from a
   ramp ordered by ink density — `" .:-=+*#%@"`. A glyph is just a brush with a fixed
   amount of ink in it.
4. **Draw.** `fillText` at the cell position, coloured from the palette gradient.

Monospace characters are about twice as tall as they are wide, so the row count is
scaled by `CHAR_AR = 0.52` — without that, every face comes out stretched.

Some styles leave that recipe behind:

- **Braille** samples at 2× width and 4× height, thresholds each of the 8 subpixels
  and packs them into a byte. Unicode lays the braille block out so the code point
  *is* the dot pattern (`U+2800 + bitmask`), giving 8× the effective resolution.
- **Quadrant** does the same trick with `▘▝▀▖▌▞▛▗▚▐▜▄▙▟█` over a 2×2 grid.
- **Edge** runs a Sobel operator and uses `atan2(gy, gx)` to pick between
  `/ \ | -` — reading shape instead of tone.
- **Contour** runs marching squares at seven brightness levels.
- **Dither** is Floyd–Steinberg: snap to the nearest palette colour, then push the
  error onto the neighbours you haven't drawn yet (7/16, 3/16, 5/16, 1/16).
- **CMYK** draws four halftone screens at 15° / 75° / 0° / 45° — the angles
  commercial printing uses so the dots form rosettes instead of moiré.
- **Oil** rotates each brush stroke to follow the local edge direction.

Rendering is bucketed by brightness level with a counting sort, so a frame sets
`fillStyle` a few dozen times instead of thirty thousand times.

## Running it locally

```bash
git clone https://github.com/biryaniji/ascii-art-generator.git
cd ascii-art-generator
open index.html        # macOS — or just double-click the file
```

No server needed. Chrome and Firefox treat `file://` as a secure origin, so the
camera works straight from disk. If your browser refuses, serve it:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Keyboard

| Key | Action |
|---|---|
| `f` | Full screen (or double-click the canvas) |
| `esc` | Leave full screen |
| `←` `→` | Previous / next style |
| `space` | Freeze / resume |
| `[` `]` | Decrease / increase detail |
| `s` | Save PNG |
| `c` | Copy text |
| `i` | Invert |

## Browser notes

Needs a reasonably current Chrome, Firefox, Safari or Edge — it leans on Canvas 2D
`filter`, `ellipse` and `roundRect`. Camera capture requires a secure context:
`https://`, `localhost`, or a local file. It will not work over plain `http://` on a
remote host.

Spiral, Ripple, CMYK and Oil paint issue a few thousand draw calls per frame. They're
fine on stills and in the hover previews, but on live camera at high column counts
they drop frames — freeze first, or keep detail around 80–100.

## Licence

MIT — see [LICENSE](LICENSE).
