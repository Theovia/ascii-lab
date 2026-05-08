# ASCII Lab

Cursor-reactive ASCII portrait studies. Single-file HTML, WebGL2 fragment shaders, glyph atlas, no build.

Live: **[atelier.eigenatlas.com](https://atelier.eigenatlas.com)**

Part of the [Eigen Atlas](https://eigenatlas.com) open lab.

---

## What this is

A WebGL2 fragment shader that turns any image into a cursor-reactive ASCII portrait at 60Hz. Each pixel of the output canvas:

1. Computes the cell it falls in (e.g. 4 px wide).
2. Samples the source image at the cell center.
3. Maps luminance to a glyph index in a 42-character ramp.
4. Reads the corresponding glyph from a pre-rendered atlas texture.
5. Composites it back to the canvas in the source's true RGB or a chosen palette.

Cursor proximity drives a global affine transform (rotation + translation + zoom) on the image lookup UV — only inside an elliptical face mask. The figure follows the cursor with its head while the background stays still.

## Variants

| File | What it explores |
| --- | --- |
| `v0-foundation.html` | Canvas 2D baseline. Brightness ramp, true RGB, cursor displacement. |
| `v1-edges.html` | Sobel + Difference-of-Gaussians edge detection. Char direction quantized to `\| \ _ /`. Acerola-style line-art. |
| `v2-webcam.html` | `getUserMedia` live ASCII at 30 fps with the same shader pipeline. |
| `v3-audio.html` | WebAudio FFT drives cursor radius (bass), char glitch (mids), hue (highs). |
| `v4-shader.html` | WebGL2 port. Two-pass: ASCII → FBO → bloom + chromatic aberration + CRT scanlines. Sub-pixel cells, 4K-ready. |
| `atelier.html` | Light cream + indigo ink edition. Editorial layout. |
| `specimen.html` | Museum specimen plate layout. Hover-reveal Sobel edge plate inside a circular lens. |
| `procession.html` | Sticky canvas + scroll-driven dissolve between 5 paintings. Per-cell hash threshold. **Deployed at atelier.eigenatlas.com.** |
| `site.html` | Full landing structure prototype. |
| `index.html` | Gallery cards linking all variants. |

All files are single HTML — no build step, no package manager. Open them with `open file.html` and they run.

## Technique

### Fragment shader pipeline

```
fragPx → cellId → cellCenterUV → tilted lookupUV (face mask only)
       → image sample (true RGB)
       → luminance → ramp index
       → atlas sub-UV → glyph alpha
       → final = mix(bg, charColor, glyph)
```

### Glyph atlas

A 64-pixel monospace cell is rendered to an offscreen 2D canvas for each character in the ramp:

```
" .'`,:;!iI?ltjf1234567890[]{}|()*#$@%MW&8B"
```

The canvas is uploaded as a single RGBA texture (8 columns × 6 rows). The shader samples a sub-region per cell.

### Face mask

```glsl
vec2 maskD = (cellCenterUV - vec2(0.5, 0.38)) / vec2(0.24, 0.32);
float faceMask = 1.0 - smoothstep(0.75, 1.15, length(maskD));
vec2 lookupUV = mix(staticUV, tiltedUV, faceMask);
```

Background stays still. Face follows cursor.

### Scroll dissolve (procession)

Per-cell hash threshold + scroll progress decide which of two source textures the cell samples:

```glsl
float thr = hash(cellId);
bool useTo = u_progress > thr;
vec4 src = useTo ? texture(u_texTo, uv) : texture(u_texFrom, uv);
```

Result: paintings rewrite themselves cell-by-cell as you scroll.

## Stack

- WebGL2 fragment shaders (GLSL ES 3.00)
- Canvas 2D for atlas rasterization + the v0-v3 baselines
- Plus Jakarta Sans + Newsreader serif + Geist Mono
- Cloudflare Pages for deploy
- Wikimedia Commons CC-PD-Art for source paintings

## Run locally

```bash
open procession.html
```

Or any of the other variants. They are all self-contained.

## Deploy

```bash
wrangler pages deploy . --project-name=ascii-lab
```

Currently published to:
- `https://ascii-lab.pages.dev`
- `https://atelier.eigenatlas.com`

## Source paintings

Bundled in `paintings/`. All Public Domain via Wikimedia Commons.

| File | Painting | Author |
| --- | --- | --- |
| `01-vermeer.jpg` | Girl with a Pearl Earring | Johannes Vermeer · c. 1665 |
| `02-botticelli.jpg` | The Birth of Venus | Sandro Botticelli · c. 1485 |
| `03-vangogh.jpg` | The Starry Night | Vincent van Gogh · 1889 |
| `04-hokusai.jpg` | The Great Wave off Kanagawa | Katsushika Hokusai · c. 1831 |
| `05-monalisa.jpg` | La Gioconda (Mona Lisa) | Leonardo da Vinci · c. 1503-1519 |

## License

Code: MIT.

Source images: Public Domain (artists deceased > 100 years).

---

**Eigen Atlas** — sistemas inteligentes en la frontera.
