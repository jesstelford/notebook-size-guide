# AGENTS.md

Guidance for agentic coding tool continuing this project. Terse on purpose: durable
constraints, the non-obvious "why", and landmarks — everything else is discoverable in
`docs/index.html` (read it; the code is banner-commented by concern).

## Project

Interactive **to-scale notebook-size reference guide**: single static page comparing
notebook/paper dimensions across ISO, JIS, brand/format sizes, with region-aware shopping links.

- **Live:** https://jesstelford.github.io/notebook-size-guide/
- **Repo:** https://github.com/jesstelford/notebook-size-guide
- **All code in `docs/index.html`** — one self-contained file (HTML + CSS + inline vanilla JS).
  No framework, bundler, build step.

## Hard constraints (keep these)

- **Single file.** Everything in `docs/index.html` unless strong reason to split.
- **No browser storage.** No `localStorage` / `sessionStorage` / cookies — state in-memory only.
- **No build tooling** unless requested; must stay openable as a bare file.
- **Dependency-light, framework-free.** Runtime deps are CDN-only, not vendored: Google Fonts,
  one `fetch` to `https://ipwho.is/` (region), Three.js core lazy-imported on first 3D toggle.
  No other network calls, no server, no secrets.
- **Never fabricate data.** Dimensions/specs are verified only; if a value can't be verified,
  link out / show an honest "not available" — don't guess.

## Running & deploying

- **Local:** open `docs/index.html` in a browser, or `npx serve docs`. No install/build.
- **Deploy:** GitHub Pages, `main` / `docs` folder (Settings → Pages → "Deploy from a branch",
  `/docs`). `docs/index.html` = entry point; social image = `docs/social.png`.

## Landmarks

All logic is one inline `<script>`, grouped with `/* ---------- NAME ---------- */` banners in
this order: marketplace/region → units → detection → data (`sizes[]`) → diagram → cards →
filters → region-wiring → 3D view. Read the code for specifics; the non-obvious invariants:

- **`sizes[]` is the single source of truth** (field legend + data rule commented above the
  array). Cards, filters, diagram and the 3D pile all derive from it + the `hidden` Set + the
  `sortBy` key — never track selection/order state separately.
- **`renderDiagram()` is the single sync hook.** Every size/unit/sort toggle calls it, and it
  **dispatches to `view3d.rebuild()` while 3D is active** — that guard keeps 2D and 3D in step.
- **The URL is the source of truth for view options** (`VIEW` / `buildURL` / `writeURL`).
  `VIEW` is parsed once up-front and `hidden` / `sortBy` / `baselines` / `state3d` initialise
  **directly from it**, so a shared link is correct on the first paint — never restore in a
  post-render pass. Writes always use `replaceState` (camera debounced via `writeURLSoon`).
  A bare URL must keep loading exactly as it does today.
- **Sort** (`sortBy`/`sortDesc`; dropdown Area/Width/Height, Area ties break by width) orders
  cards, 2D callouts and the 3D pile together.
- **Diagram** `viewBox` is recomputed each render from the visible bounding box (hiding sizes
  zooms the rest); baseline refs (`baselines` Set — credit card / US Letter) factor into bounds.
- **3D landmines:** the pile sits left-of-centre via `camera.setViewOffset` (constant pixel
  shift) — it **owns `camera.aspect`; never set aspect manually, and never aim the camera at an
  offset look-point** (that reintroduces the orbit-pivot drift bug). The canvas grows past
  `baseViewH` to fit the callout column and reserves `topReserve()` for the top controls.
- **Accessibility invariants:** toggle-style buttons need `aria-pressed`; the canvas stays
  keyboard-operable (focusable, arrow-rotate, `±`-zoom); keep any new size `color` ≥ 4.5:1 on white.

## Design principles & decisions (the "why" — not visible in source)

- **Accuracy over convenience.** Prices/ratings are **linked out, not embedded** — Amazon/Camel
  block automated reads and don't expose live prices. (An embedded price-snapshot feature was
  built then deliberately **removed** for this reason.)
- **JIS is the primary B-series standard** (these are Japanese-notebook sizes); ISO is a separate
  entry. A-sizes are identical worldwide (one entry each).
- **"Slim" / "Reporter's" aren't real standards** — brand-specific cuts / a binding style; flag
  `approx` and show each product's real dimensions. Same for brand sizes ≠ pure ISO (Leuchtturm
  Pocket / B6+ / Composition run off-spec).
- **Price aggregator = CamelCamelCamel, not Keepa** — chosen because **Keepa doesn't cover
  Australia**. Camel covers US/UK/CA/DE/FR/IT/ES/JP/AU; the Camel link is hidden where
  unsupported (`camel: null` in `MARKETS`).

## Gotchas

- Region detection depends on `ipwho.is`; if blocked/rate-limited it silently falls back to
  browser locale, then `XX` — expected, not a bug.
- Country-flag emoji in the region `<select>` render as letters on some platforms (Windows) —
  cosmetic only.

## Possible next steps

- Deferred candidate sizes (add **only** with verified dimensions): Steno (152 × 229); US Letter
  / Half-Letter / Legal as *entries* (Letter is currently only the US scale reference, not a
  card); other Traveler's-style formats.

## Maintaining this file

Keep this file for knowledge useful to almost every future agent session in this project.
Do not repeat what the codebase already shows; point to the authoritative file or command instead.
Prefer rewriting or pruning existing entries over appending new ones.
When updating this file, preserve this bar for all agents and keep entries concise.
