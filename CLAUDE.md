# Different Side of You – photo kiosk (NRG / IQOS event engagement)

Portrait 4K touch-screen photobooth. A guest picks a "frame" effect, takes 3 shots, and gets a
3-photo strip (printed as 2 pieces) plus a QR to download it. Plain HTML/CSS/JS, **no build step,
no framework, no npm dependencies**. Everything lives in `index.html`.

The team speaks Bahasa Indonesia; reply to them in Indonesian unless asked otherwise.

## Run it

```bash
npx http-server photo-different-kiosk -p 5700 -c-1
```

Open http://localhost:5700 in **Chrome**. Use `localhost`/https: the camera is blocked on `file://`.
On the TV: Chrome kiosk mode, portrait 2160×3840 (see README.txt, written in Indonesian, for kiosk/admin setup).

## Files

- `index.html` – the whole app (CSS + markup + one JS IIFE).
- `assets/` – IQOS fonts, UI art (`bg-a/bg-b.webp` backgrounds, `btn.webp` glitch button, `hero.webp`,
  `qrbox.webp`), filter art (`fx-bg.webp` background behind the guest in all 3 filters, `track-glass.webp`
  TRACK glass shapes, `grid-light.png` GRID light streaks), `strip-frame.png` (print template),
  `bg-source.webp` (client key visual, only used by the alternate "Art for the Curious" skin).
  `download-frame.webp` (9:16 guest-download background), `sample-guest.webp` (photo on the frame cards).
- `lab/` – **Filter Lab** dev page (http://localhost:5700/lab/): renders the 3 filters from the live app code
  (hidden iframe + `window.__pd`) next to the client references, with photo upload. Not needed on the kiosk.
- `lib/qrcode.js` – offline QR generator.
- `README.txt` – operator guide (Indonesian).

## Flow (6 screens, `<section data-screen=…>`)

`start` → `consent` → `lens` (choose frame) → `brief` → `capture` (one CAPTURE tap runs all 3 shots: countdown → shot → 1 s freeze → next countdown; `S.run` cancels stale timers; Upload Photo for testing, also on `lens`) → `result` (stacked 2-piece strip + QR, FINISH → start).
Navigation: `go(name)`; buttons use `data-go`. Idle reset after `cfg.idle` seconds.

## Layout model

- Fixed design stage **1080×1920**, scaled to the screen by `fit()`; `RES` = 2 on 4K so canvases render at 2×.
- Wider screens show **black side bars** (no ambient fill; intentional).
- Headlines are `.stag` (cascaded lines), auto-shrunk by `fitStagger()`.
- UI skin = **Taste Experience** look: CSS override block "Taste Experience UI skin" + `btn.webp`
  inside every `.cta`. A frozen copy with the alternate "Art for the Curious" skin (pill buttons) lives in
  the sibling folder `photo-different-kiosk_art-for-the-curious` (port 5701). The client has switched
  skins twice – keep effect/flow changes in this folder; port to the backup only if asked.

## The three effects (`LENSES` → `FX`)

All effects are real-time canvas filters: `FX[key](ctx, frame, w, h, timeMs, seed)`. Every render uses a
**1.2 aspect ratio** (preview 880×733, strip photo 520×433, card slots) – keep it, masks/zoom depend on it.

Reworked Nov 2026 with the client in several rounds (Filter Lab). Common base for all three: natural photo
(light cool grade), background replaced by `assets/fx-bg.webp` (`navyFill` draws it; gradient only as fallback)
via `paintBg` + `bgMask`, and `personGlow` (blue halo outside the silhouette + blue light wrap on hair/shoulders).

| Card | key | function | Look |
|---|---|---|---|
| 1 FACET | `facet` | `fxFacet` | Faint GRID quadtree (`drawGridLines`, 55%), thin glitches (`thinGlitch`: hairline slice shifts + small clusters of 1–3 px glitch bars, never on faces), 8 thin cyan light streaks. Triangle shards were removed (client: too heavy). |
| 2 GRID | `grid` | `fxGrid` | Quadtree grid (`drawGridLines`, 100%, 75% fainter over people), top layer `grid-light.png` in Screen. No shards. |
| 3 TRACK | `scan` | `fxScan` | Brighter 45° beams on the background, `track-glass.webp` in Screen over the whole photo (fixed layout, may touch people — client approved), cyan→violet streaks, thin (0.8u) glowing rounded face boxes + 2 tracking dots. |

Shared helpers: `drawSrc` (cover-crop + mirror + auto-frame zoom), `getBuf` (cached offscreen canvases),
`bgMask` (white = background), `personShape`, `paintBg`, `navyFill`, `personGlow`/`rimLight`/`lightWrap`,
`drawShards`, `drawGridLines`, `glowStreak`, `vignette`, `fInFace`, `rnd` (seeded hash random).

## Vision pipeline (important)

- `Vision` loads **MediaPipe Tasks** from jsDelivr + Google Storage at start-up (needs internet):
  selfie segmenter (person mask) + BlazeFace (boxes + 6 keypoints). Runs on a 320×267 copy of the frame.
- **Auto-framing**: `AZ` zoom region steered from face boxes (`steer()`), applied inside `drawSrc` (max 2.2×).
  Vision itself works on the raw frame (`drawSrc(..., raw=true)`); `Vision.faces()/mask()/bgAt()` return
  coordinates mapped into the zoomed frame.
- **Remove background** (`cutout()`, admin toggle `cfg.removeBg`, default on): guest pasted over
  `fx-bg.webp` (same art as the filter background); result frame has `cut:true`.
- Safety: if a detected face lands on "background" in the mask, the mask is ignored for that frame
  (`bgOk`) so the guest is never erased.
- Offline fallback: skin-tone face finder (`findFaces`) + ellipse masks.

## Output

`showResult()` → `drawStrip()` (600×1800 design, rendered at 2×; client template `assets/strip-frame.png`
"Moments for the curious" laid over the photos, each 1.2 photo cover-cropped into its ~1.4 window) → display as two
stacked strips → **print file** = 4×6 sheet with 2 strips side by side (2400×3600 JPG, `buildSheet`/`drawSheet`,
`S.blob`, admin "Download print 4x6"). **Guest download (QR upload)** is a different 9:16 picture, 1080×1920
(`buildDownload`, `S.dlBlob`): `assets/download-frame.webp` as background, the 3 photos on top in `DL_SLOTS`
(boxes from the client's placement guide, `lab/guide-download.webp`).
Photo window layout is adjustable in the **Jarak foto** panel (admin button or Ctrl/Alt+L): `cfg.layout` holds three
sets – `preview` (screens), `printL`, `printR` (each half of the print sheet) – of `{x,left,size,top,gap}` fractions
(`LAY_DEF` = the template's windows; re-measure if the template changes). `refreshResult()` redraws the result and print file → `publish()` POSTs to `cfg.uploadUrl`
(expects `{"url": "..."}`) and renders the QR. No upload server exists yet – without one the QR only
contains a session ID.

## Admin panel

Hold top-left corner 2 s or Ctrl/Alt+M. Settings in `localStorage` key `nrg_different_v1`
(camera, mirror, remove bg, upload URL, fallback QR link, idle seconds, rotation).

## Testing tips

- `window.__pd` debug hook: `go`, `chooseLens`, `capture`, `S` (state), `cam`, `fx`, `vision`, `cutout`.
- **Upload Photo** on the capture screen feeds a still image through the full pipeline (no webcam needed).
- Render an effect directly: `__pd.fx.grid(ctx, frame, 1080, 900, performance.now(), 1)` with
  `frame = {src, w, h, live:true, photo:true, mirror:false}` (pass it through `__pd.cutout(frame, -1)` first
  to include background removal; call `__pd.vision.update(frame, true)` before that).
- Keep per-frame cost low (target < ~50 ms at 4K preview); measure after changes.

## Conventions / gotchas

- Single file, ES5-style JS (no modules except the dynamic MediaPipe import). Keep it that way.
- When patching code with JS `String.replace`, remember `$$` in the replacement becomes `$`
  (this broke `fitStagger` once) – use `split/join` or a replacer function.
- Effects must stay deterministic per `seed` (shot 1/2/3 look different but stable while live).
- Don't add an AI image-generation step unless asked: it was scoped and the team chose real-time filters.
