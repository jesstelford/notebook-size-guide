# CLAUDE.md

Guidance for agentic coding tool continuing this project.

## Project

Interactive **to-scale notebook-size reference guide**: single static page comparing
notebook/paper dimensions across ISO, JIS, brand/format sizes, with region-aware shopping links.

- **Live:** https://jesstelford.github.io/notebook-size-guide/
- **Repo:** https://github.com/jesstelford/notebook-size-guide
- **All code in `docs/index.html`** — one self-contained file (HTML + CSS + vanilla JS).
  No framework, bundler, build step.

## Running & deploying

- **Local:** open `docs/index.html` in browser, or serve static (`npx serve docs`). No install/build.
- **Deploy:** GitHub Pages, `main` / `docs` folder (Settings → Pages → source "Deploy from a
  branch", folder `/docs`). `docs/index.html` = entry point.
- **Runtime deps (CDN, not vendored):** Google Fonts (Space Grotesk, IBM Plex Sans/Mono);
  one client-side `fetch` to `https://ipwho.is/` for region detection. No other network calls,
  no server, no secrets.

## Hard constraints (keep these)

- **Single file.** Everything in `docs/index.html` unless strong reason to split.
- **No browser storage.** No `localStorage` / `sessionStorage` / cookies — state in-memory only.
- **No build tooling** unless requested; must stay openable as bare file.
- Dependency-light, framework-free.

## Code style

- Plain browser JS in inline `<script>` — no transpile, modules, framework.
- Grouped by concern (see script order below). Derive state from single source of truth over
  tracking separately — e.g. filter buttons' selected state derives from `hidden` set, not stored.

## Layout inside `docs/index.html`

1. `<head>` — meta (incl. SEO/OG/Twitter tags; `og:image` = `docs/social.png`), fonts, **all
   CSS**. CSS variables define "ink-on-paper / drafting spec-sheet" theme.
2. `<body>`:
   - `header` — eyebrow (`.region` selector top-right, after ruled line) + `h1`
   - `.stage` — `svg#diagram` (to-scale drawing) + `#view3dBtn` (top-right, WebGL-gated)
     + `#nbControls` (top-left Cover/Pages controls, 3D only) + `#srStatus` live region
   - `.stage-foot` — `#filters` button row + `.foot-right` (`.sortby` dropdown + `.baseline` toggles)
   - `.cards#cards` — per-size info cards (JS-generated)
   - `.notes` — footnote cards (JIS vs ISO, Slim, shopping links, brand-size caveats)
3. `<script>` (all logic), this order:
   - **Marketplace/region:** `MARKETS`, `MARKET`, `amazonURL` / `camelURL`
   - **Units:** `IMPERIAL`, `UNIT`, `unitForMarket`, `fmtDim`, `updateUnits`
   - **Detection:** `countryFromLocale`, `detectCountry`
   - **Data:** `sizes[]`
   - **Diagram:** `renderDiagram()` (+ `sortBy`/`sortDesc`; dispatches to `view3d.rebuild()` while 3D active)
   - **Cards:** `prodHTML`, `cardHTML`, render + checkbox wiring, `applyCardOrder`
   - **Filters:** `GROUPS`, `syncChecks`, `refreshFilters`, click handler
   - **Region wiring:** `regionSel` (+ `syncRegionDisplay` short/full option text),
     `baselines`/`syncBaselineButtons`, `sortSel` handler, `applyMarket`, detection init
   - **3D view:** lazy Three.js import, `view3d` controller (scene/notebook builders,
     view-locked SVG label overlay, orbit + pinch + keyboard controls, mount/unmount)

## Data model

`sizes` = **single source of truth** (array of objects). Fields:

- `id` — unique string; used by checkboxes, filters, diagram.
- `name`, `tag` — display name + pill text (e.g. `"B6"` / `"JIS"`, `"Traveler's"` /
  `"Regular · non-standard"`). Part of `tag` before first `·` = short label in diagram callouts.
- `color` — hex; size's ink-wash / leader-line / pill colour.
- `w`, `h` — **real width/height in millimetres**. Drive to-scale drawing + all unit conversion.
- `mm` — human string incl. unit; shown as secondary reference (may contain `≈`).
- `in` — legacy inch string, **no longer displayed** (`fmtDim` computes units live).
- `alt` — small grey descriptor line.
- `use` — one-line use case.
- `approx` (optional bool) — prepends `≈` to computed dimension strings.
- `none` (optional HTML) — honest note when size has no clean match (Slim / Reporter's).
- `products[]` — `{ brand, model, size, specs: [[k,v], …], note?, search }`. `search` =
  **market-neutral** query string building both Amazon + Camel links.

**Current 25 sizes** (Sort dropdown orders cards + diagram callouts by **Area** (default; ties
broken by width), **Width** or **Height**, descending):
A4, Smythson Portobello, B5 JIS, B5 ISO, B5 Slim (approx), Traveler's Regular, A5,
Smythson Soho, Moleskine Large, Reporter's (approx), Baron Fig Confidant (approx),
Hobonichi Weeks, B6 JIS, Smythson Chelsea, B6 Slim Stalogy, B6 ISO, B6 Slim Midori,
A6 Slim Traveler's (approx), A6, A6 Slim Muji, Field Notes, B7 JIS, Traveler's Passport,
B7 ISO, A7.

- **Moleskine Large** (130×210) — A5-height, 18 mm narrower; covers whole Moleskine Large line
  (Classic, Cahier) + fountain-pen "A5 Slim" 130×210 footprint.
- **Baron Fig Confidant** (≈137×196, from 5.4×7.7 in spec) — smaller than A5 on both axes.
- **Hobonichi Weeks** (94×188) — only bespoke Hobonichi trim (not A/B); Weeks Mega shares it.
  A6 Original + A5 Cousin reuse existing A6/A5 entries (listed there as products).
- **A7** (74×105 ISO) + **B7** (JIS 91×128 / ISO 88×125). Genuine ISO-B7 notebooks barely exist
  at retail, so that entry links to closest brand substitute (Rhodia No. 12).
- **A6 Slim** — maker's narrow cut, not a standard: two entries, **Muji (92×147, verified)** +
  **Traveler's Company spiral-ring (≈94×152, quoted 6×3.7 in, `approx`)**. Near-A6 height,
  ~11–13 mm narrower; overlaps Moleskine/Leuchtturm-Pocket cluster — honest note says so.
- **Smythson** (leather, Featherweight 50 gsm paper) — three bespoke trims verified from
  smythson.com cm listings: **Soho 140×190**, **Chelsea 112×167**, **Portobello 210×260**
  (card `name` = trim only; brand shows in tag pill). **Panama (90×140)** duplicates Field Notes
  3.5×5.5 in footprint; **Wafer (70×105)** ≈ A7 — both listed as products under Field Notes / A7,
  not separate rectangles. ("Featherweight" = paper, not a size.)

`MARKETS` maps ISO country code → `{ code, label, flag, amazon (host), camel (subdomain
prefix | null) }`. `camel: null` ⇒ Camel unsupported ⇒ Camel link hidden. `XX` = ".com / other"
fallback.

## Core behaviours

- **Diagram (`renderDiagram`)** — all visible sizes = translucent rectangles sharing bottom-left
  corner (largest drawn behind). `viewBox` **recomputed each render from visible bounding box**,
  so hiding larger sizes zooms rest to fill container. Right-hand gutter holds non-overlapping
  leader-line callouts (name + dimensions), ordered by active **Sort** key with push-down
  anti-overlap pass. To-scale **baseline references** drawn at shared bottom-left corner: filled
  dark **credit card (85.6 × 54 mm)** and/or outlined **US Letter sheet (215.9 × 279.4 mm /
  8.5 × 11 in)**. Active set = `baselines` Set (`"card"` / `"letter"`), toggled by Baseline
  buttons; either, both, neither. Active refs factored into bounding-box computation so never
  clip when only small sizes visible. Callout dimensions use active unit.
- **Cards** — generated in `sortDesc` order (active **Sort** key). `applyCardOrder()` re-orders
  existing card nodes in place (keeping listeners) when Sort changes. Each card: visibility
  checkbox (updates `hidden` Set + diagram), inline header (checkbox + name + tag pill),
  dimensions (switchable primary unit + mm secondary), use case, collapsible product section.
- **Sort (`sortBy` + `sortDesc`)** — dropdown (**Area** default / **Width** / **Height**) orders
  cards, 2D callouts, 3D pile, descending; Area ties break by width. Changing it calls
  `applyCardOrder()` + `renderDiagram()` (routes to `view3d.rebuild()` in 3D). In 2D gutter,
  Height tracks each box's top edge; Area/Width stack callouts evenly.
- **Filters (`GROUPS` + `refreshFilters`)** — additive multi-select, two modes:
  - **All mode** (`hidden.size === 0`): `All` button active, group buttons off.
  - **Partial mode**: `All` off; group button active **iff every item in it visible**.
  - Button state **derived** from `hidden` every change — never tracked separately.
  - `All` only ever turns **ON**. First group click from All mode focuses down to that group;
    subsequent clicks additive (toggle whole categories on/off). Individual checkbox changes
    re-derive all button states.
  - Groups: `a4` (A4 + Smythson Portobello), `a5` (A5 + Moleskine Large + Baron Fig +
    Smythson Soho/Chelsea), `b5` (3 variants), `b6` (4 variants), `b7` (JIS + ISO),
    `a6` (A6 + both A6 Slims), `a7`,
    `ftr` (Field Notes + Traveler's Regular/Passport + Reporter's + Hobonichi Weeks).
- **Baseline (`baselines` + `syncBaselineButtons`)** — pair of independent toggle buttons
  (Credit Card / US Letter) in stage foot, controlling which to-scale references draw.
  `applyMarket()` sets region default (US → Letter, else Card); buttons override, both may be on.
  Work in **both views**: 2D draws filled card / outlined Letter frame; **3D renders non-book
  slabs** of ~credit-card thickness (card = dark, real rounded corners; Letter = white sheet,
  square corners), stacked with books by active Sort key, own callout labels.
- **3D view (`view3d`)** — optional WebGL scene (Three.js core lazy-imported from CDN on first
  toggle; button hidden without WebGL). Books = 4 prisms (coloured/textured spine + overhanging
  covers, recessed white pages); thickness from global Cover/Pages controls (top-left of stage,
  3D only). Pile stacks by active **Sort** key (biggest at bottom), matching cards + 2D callouts
  — `buildNotebooks()` reuses `sortDesc`. Labels = view-locked SVG overlay calibrated to 2D
  callout pixel size. Orbit = drag / arrow keys; zoom = wheel / pinch / +-. Pile sits
  left-of-centre via `camera.setViewOffset` (constant pixel shift — keeps orbit pivot visually
  fixed); do **not** aim camera at offset look-point, never set `camera.aspect` manually
  (setViewOffset owns it). Canvas height = `baseViewH` (60% of width, ≤ 75vh) but **grows**
  (`canvasHeight` / `resizeCanvas`) when callout column taller, reserving `topReserve()` at top
  so no callout hides behind controls; `frameScene` scales distance by growth factor +
  `applyViewOffset` shifts pile up, so pile keeps base size instead of ballooning on tall canvas.
- **Accessibility** — toggle-style buttons carry `aria-pressed`; WebGL canvas focusable
  (`role="img"`, arrow keys rotate, `+`/`-` zoom); label overlay `aria-hidden`; `#srStatus`
  (`role="status"`) announces 2D↔3D switches + load failure; per-size checkboxes have unique
  `aria-label`s. `--ink-faint` + five accent hexes darkened for WCAG AA small-text contrast —
  keep any new size colour ≥ 4.5:1 on white.
- **Region + units** — `detectCountry()` tries `ipwho.is` (non-intrusive; **no** geolocation
  prompt), falls back to browser locale, then `XX`. `applyMarket()` sets marketplace (Amazon +
  Camel link domains) + unit (**US/CA → inches, else → cm**), then refreshes links, dimension
  text, baseline default (US → Letter, else Card), diagram. Dropdown allows manual override;
  manual choice suppresses later auto-detection.

## Design principles & decisions (the "why")

- **Accuracy over convenience — never fabricate data.** No invented prices, ratings, review
  counts, specs. Amazon + Camel block automated reads, search snippets don't expose live prices,
  so pricing **linked out, not embedded**. (Embedded price-snapshot feature built then
  deliberately **removed** for this reason.) If value can't be verified, show honest
  "not available" / link out — don't guess.
- **JIS = primary B-series standard** (these are Japanese-notebook sizes); ISO = separate
  entry/annotation. A-sizes identical worldwide (one entry each).
- **"Slim" not a real standard** — brand-specific (Midori ≈110×176, Stalogy 100×180); B5 Slim
  barely exists (Semi-B5 substitute noted). Hardcover-dotted doesn't exist in Slim, so
  softcover/grid options accepted there.
- **"Reporter's" = binding style, not fixed size** — flagged `approx`; products list own real
  dimensions.
- **Brand sizes ≠ pure ISO** — Leuchtturm's Pocket / B6+ / Composition / Master run off-spec;
  always display each product's **real** dimensions over implying a match.
- **Price aggregator = CamelCamelCamel.** Camel covers US/UK/CA/DE/FR/IT/ES/JP/**AU**;
  **Keepa doesn't cover Australia**, why Camel chosen. Camel tracks price only (not ratings),
  so ratings/reviews left to Amazon listing.

## Known limitations / gotchas

- Region detection depends on `ipwho.is`; if blocked/rate-limited, silently falls back to locale
  — expected, not a bug.
- Country-flag emoji in region `<select>` render as letters on some platforms (Windows) —
  cosmetic only.
- Product prices/ratings intentionally **not** in page — links only.
- Camel link hidden for marketplaces Camel doesn't support.

## Possible next steps

- Candidate sizes discussed but deferred (add **only** with verified dimensions): Steno
  (152 × 229), US Letter / Half-Letter / Legal as *entries* (Letter currently only US scale
  reference, not a card), other Traveler's-style formats.
