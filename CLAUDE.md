# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the site

No build system. Open `index.html` directly in a browser, or serve with any static server:

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

## Architecture

Everything lives in a single `index.html` — inline CSS, then React 18 loaded via CDN (UMD), with Babel Standalone for in-browser JSX compilation. No bundler, no npm.

**Data layer** — two delimited blocks near the top of the `<script type="text/babel">` section:

- `SITE_CONFIG` (between `// __SITE_CONFIG_START__` and `// __SITE_CONFIG_END__`) — artist name, contact info, bios, stats
- `PRODUCTS` (between `// __PRODUCTS_START__` and `// __PRODUCTS_END__`) — array of product objects

Adding a new piece = push one object onto `PRODUCTS`. Every UI surface (map hotspots, shop grid, product modal, cart) reads from this single array.

**Product object shape:**
```js
{
  id: 'unique_id',
  name: "Display Name",
  series: "Series Name",
  price: 15000,                    // INR
  status: 'available' | 'sold' | 'reserved',
  emoji: '🏠',                    // fallback when images is empty
  images: [],                      // paths like 'images/id/filename.jpg'
  hotspot: { x: 13, y: 8 },       // % position on map background
  building: { x: 13, y: 38 },     // % position (building base, currently unused in rendering)
  desc: "...",
  materials: "...",
  dims: "...",
  tags: ["tag1", "tag2"],
}
```

**Map background** — `map-bg.mp4` (video, preferred) over `map-bg.png` (fallback). `MapBackground` component layers them; video autoplay is muted + looped. Hotspot `x`/`y` values are percentages tuned to the current `map-bg` composition — adjust when the background changes.

**Images** — stored in `images/<product-id>/` (UUID-named JPEGs). When a product's `images` array is empty, the UI shows an emoji + striped placeholder. Populate `images` to show real photos.

**Pages** — fixed-position divs that slide up (`transform: translateY`) over the map. Nav links toggle their `visible` class. Pages: Shop, Custom Order, About. The product detail view is a bottom-sheet modal (`modal-overlay` + `modal-sheet`).

**Cart** — slide-in right drawer. State lives in `App` via `useState`. `const BAKED = false` is a placeholder flag, currently unused.

**CSS tokens** (`:root`): `--parchment`, `--parchment-dark`, `--ink`, `--ink-soft`, `--forest`, `--gold`, `--stone`. All themed values should use these variables.

**Responsive breakpoint**: `@media (max-width: 768px)` — mobile collapses nav to hamburger, switches modal to single column, cart drawer goes full-width.

## Folder layout

```
index.html           ← active site (edit this)
map-bg.mp4 / .png    ← hero background assets
images/              ← product photos, one subfolder per product id
v1/, v2/             ← archived earlier design iterations (do not edit)
handoff_extracted/   ← original Claude Design handoff bundle (reference only)
index_old1.html      ← archived snapshot
```
