# web-components

**Repository:** https://github.com/remixlabs/web-components
**Stack:** TypeScript, JavaScript, Elm, SolidJS (TSX), Vite, CircleCI
**Purpose:** Monorepo of 13 standalone registered web components for Remix Studio. Separate from turntable's bundled components — these are registered via manifests and served through standard Remix
search/catalogs.
**Explored layers:** L1–L4 (skeleton, manifests/API surface, all source implementations, build configs). L5 (READMEs/CSS/demos) skipped — low architectural value.

## Build & Deploy

- **Build:** `npm run build` → top-level `build.mjs` (zx) sequentially runs `npm install && npm run bundle` in each component dir
- **Per-component:** Vite builds to `dist/<name>.js`, then `bundle.js`/`bundle.mjs` zips dist + manifest into `dist/<name>.zip`
- **Index generation:** `make-index.ts` reads zip manifests → produces `webcomp-index` JSON (kind, timestamp, build, packages[])
- **Bundle script:** Identical across components — JSZip reads `manifest.json`, strips `url` field, zips `<tag>.js` + manifest → `dist/<tag>.zip`
- **Output formats:** Two patterns — ES modules (`formats: ["es"]`) for dev-test, vega-embed-simple; IIFE (`format: "iife"`) for all others. Maps to manifest `module: true/false`
- **Deploy (legacy):** `upload-overwrite.mjs` syncs `artifacts/assets/` to GCS bucket `gs://rmx-static/webcomponents`
- **Deploy (current):** Host 3 files (dist JS, thumbnail, manifest.json) to workspace files on `agt.files.remix.app`, then register in Studio
- **Deploy (R2):** image_base64_uploader uses Cloudflare R2 via `rclone` (`r2:agt-dev/dda-search/files/webcomps/...`), versioned paths
- **CI:** CircleCI (`rmx-ci-rcm-amd64` image, `rmx-prod-uploads` context) — job steps mostly commented out currently
- **Ignored dirs in build:** `map-polygons`, `lib`, `rmx-carousel`, `rmx-sortable`
- **RCM config:** package name `remix-extra-webcomps` v0.1.0
- **Vite plugins:** `vite-plugin-solid` (tui-chart, tui-grid), `vite-plugin-elm` (rmx-sortable); Elm 0.19.1
- **Dev servers:** Port 3100 (most components), port 3030 (SolidJS components), CORS enabled

## Architecture

### Manifest Schema

Every component has a `manifest.json` defining its integration contract with Remix Studio:

```
tag           — custom element name (e.g. "rmx-timer")
module        — boolean: true = ES module import, false = script tag
ins[]         — input params: { name, type, defaultValue? }
events[]      — output events: { name, payload? }
slots?        — slot configuration (SingleSlot, etc.)
url           — hosted JS file URL
thumbnail     — hosted thumbnail image URL
mix_package_version, mix_package_type, file, version, icon, description — catalog/bundle metadata
```

**Input types:** `string`, `number`, `bool`, `event` (invokable method), `{}` (object), `[{}]` (array of objects), `["card"]` (slotted children), `location`, `color`

### Hosting Pattern

- Primary: `https://agt.files.remix.app/remix-libraries/files/webcomps/<name>/v<N>/<name>.js`
- Alternate (catalog library): `https://agt.files.remix.app/remix_labs/_rmx_files/webcomps/<name>/v<N>/<name>.js`

### UI Framework Split

| Framework                   | Components                                                                         |
|-----------------------------|------------------------------------------------------------------------------------|
| **Lit** (v2–3)              | dev-test, emoji-picker, rmx-sortable, rmx-timer, rmx-video                         |
| **SolidJS** (solid-element) | rmx-tui-chart, rmx-tui-grid                                                        |
| **Vanilla JS**              | image-base64-uploader, map-polygons, rmx-carousel, rmx-map-icon, vega-embed-simple |
| **marked** (+ vanilla)      | rmx-markdown                                                                       |
| **Elm** (dev harness only)  | rmx-sortable test page                                                             |

## Component Catalog (13 active)

### dev-test

Test harness demonstrating all web component features: properties (title, count), invokable methods (reset), events (current-count, just-trigger), slot support. Lit-based. Reference implementation for
WC integration contract.

### emoji-picker

Single-line wrapper around `emoji-picker-element` library. Emits `emoji-click` event with emoji data payload.

### image-base64-uploader

Shadow DOM component for image upload. Supports drag & drop, file picker, and clipboard paste. Converts images to base64 data URLs. Emits `base64` event. Has `reset` method. Uses Material Design Icons
via CDN.

### map-polygons (`rmx-map`)

Full-featured Google Maps component. Supports markers, polygons, circles, and a drawing manager. Events: boundsChanged, markerClicked, polygonClicked, circleClicked, markerComplete, polygonComplete,
circleComplete. Dynamic Google Maps API loading with callback promise. Uses `@turntable/rmx-map` package name.

### rmx-carousel

Scroll-based carousel using native `<slot>` for children. IntersectionObserver for slide visibility tracking. Configurable: arrows, dots, track size/color/position, slide count. No shadow DOM — uses
`<template>` in constructor.

### rmx-map-icon

Simpler Google Maps component using AdvancedMarkerElement + PinElement (newer Maps API). Takes api-key, zoom, center, markers array, map-options. Emits `marker-clicked`. Dynamic script loading.
Auto-fits bounds.

### rmx-markdown

Shadow DOM component using `marked` library. Accepts `markdown` and `styles` attributes. Renders parsed HTML into shadow DOM with custom styles. No events emitted.

### rmx-sortable

Drag-to-reorder list using SortableJS + Lit. Uses `<slot>` for children. Emits `order` event with permutation array `[{id, oid}]`. Supports drag handles via `data-rmx-meta` attribute matching. Forces
`width: 100%` on children. Elm code in `src/Main.elm` is a development test harness.

### rmx-timer

SVG circular countdown timer. Configurable: start-time, hazard-threshold, style-config (15+ color/font/size properties), auto-start, size. Three visual states: normal → hazard → overtime. Events:
time-up, time-elapsed. Methods: start, stop, restart, getTime. Uses setInterval (1s).

### rmx-tui-chart

Toast UI Chart wrapper using SolidJS + `solid-element` + `noShadowDOM()`. Supports 16 chart types (Bar, Column, Line, Area, LineArea, ColumnLine, Bullet, BoxPlot, Treemap, Heatmap, Scatter,
LineScatter, Bubble, Pie, NestedPie, Radar). Reactive re-initialization on prop changes. Uses Zod (listed as dep but not visibly used in source).

### rmx-tui-grid

Toast UI Grid wrapper using SolidJS + `solid-element` + `noShadowDOM()`. Takes options (columns, data, etc.) and theme (presetName + extOptions). Same lifecycle pattern as tui-chart: onMount →
createEffect → onCleanup.

### rmx-video

Lit wrapper for HTML `<video>` element. Properties: src, poster, time, autoplay, controls, loop. Methods: play (with optional time seek), pause. Events: play, pause (with currentTime payload).

### vega-embed-simple

Vega/Vega-Lite visualization wrapper using `vega-embed` library. Takes `config` JSON attribute. Uses `rmx-uid` attribute for unique container IDs (avoids conflicts with multiple instances). No shadow
DOM.

## Key Patterns & Conventions

- **Registration:** All use `customElements.define("<tag>", Class)` — Lit via `@customElement` decorator, SolidJS via `customElement()` from `solid-element`
- **Events:** `CustomEvent` with `detail` payload; cross-shadow-DOM events use `bubbles: true, composed: true`
- **Invokable methods:** Defined as public methods on the class (e.g. `reset()`, `play()`, `pause()`); manifest declares them as `type: "event"` inputs
- **Shadow DOM:** Most components use shadow DOM (via Lit or manual `attachShadow`); TUI components explicitly disable it (`noShadowDOM()`) because the underlying libraries require direct DOM access
- **Google Maps loading:** Shared pattern of dynamic script injection with callback promise (`window.__initMap`)
- **Dev workflow:** Each component has `npm run dev` (Vite dev server) + `index.html` for standalone testing
- **Catalog export:** Upload dist JS + thumbnail + manifest.json to workspace files → create web component node in Studio → publish to library

## Deprecated Components (`old/`)

4 components, no longer maintained:

- `markdown-parser` — superseded by rmx-markdown
- `markdown-text-editor` — no longer used
- `rmx-date-calendar` — explicitly marked DEPRECATED
- `zoom-timer` — superseded by rmx-timer

## Open Questions

- Why is the CI pipeline mostly commented out? Is deployment now manual or handled elsewhere?
- The `rmx-tui-chart/package.json` has `"name": "rmx-tui-grid"` — intentional or copy-paste bug?
- Zod is a dependency of tui-chart and tui-grid but not visibly used in source — is it used for runtime validation elsewhere?
- What is the `rcm` tool (`rcm comp publish`)? Appears in commented-out code
- How does the `module: true` vs `module: false` distinction affect Studio runtime loading?
