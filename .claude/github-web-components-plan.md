# web-components — Exploration Plan

**Repository:** https://github.com/remixlabs/web-components
**Branch:** main
**Stack:** TypeScript, JavaScript, Elm (rmx-sortable), TSX (tui-chart/tui-grid), Vite (build), CircleCI (CI)
**Structure style:** Monorepo — 13 standalone web components under `webcomps/`, plus 4 deprecated components under `old/`
**Total files:** 226

## Layer 1 — Skeleton & Overview (must read)

Root-level config, CI, and build tooling:

- `README.md`
- `package.json`
- `tsconfig.json`
- `build.mjs`
- `make-index.ts`
- `rcm.config.mjs`
- `upload-overwrite.mjs`
- `.circleci/config.yml`

## Layer 2 — Entry Points & API Surface (high priority)

Per-component manifests and entry HTML — defines the public interface of each web component:

- `webcomps/dev-test/manifest.json`
- `webcomps/dev-test/package.json`
- `webcomps/emoji-picker/manifest.json`
- `webcomps/emoji-picker/package.json`
- `webcomps/image_base64_uploader/manifest.json`
- `webcomps/map-polygons/package.json`
- `webcomps/rmx-carousel/manifest.json`
- `webcomps/rmx-carousel/package.json`
- `webcomps/rmx-map-icon/manifest.json`
- `webcomps/rmx-map-icon/package.json`
- `webcomps/rmx-markdown/public/manifest.json`
- `webcomps/rmx-markdown/package.json`
- `webcomps/rmx-sortable/manifest.json`
- `webcomps/rmx-sortable/package.json`
- `webcomps/rmx-timer/manifest.json`
- `webcomps/rmx-timer/package.json`
- `webcomps/rmx-tui-chart/public/manifest.json`
- `webcomps/rmx-tui-chart/package.json`
- `webcomps/rmx-tui-grid/public/manifest.json`
- `webcomps/rmx-tui-grid/package.json`
- `webcomps/rmx-video/manifest.json`
- `webcomps/rmx-video/package.json`
- `webcomps/vega-embed-simple/manifest.json`
- `webcomps/vega-embed-simple/package.json`

## Layer 3 — Core Modules (medium priority)

Source code of each active web component (the actual implementations):

- `webcomps/dev-test/src/dev-test.ts`
- `webcomps/dev-test/types/dev-test.d.ts`
- `webcomps/emoji-picker/src/emoji-picker.ts`
- `webcomps/image_base64_uploader/image_base64_uploader.js`
- `webcomps/map-polygons/src/rmx-map.js`
- `webcomps/rmx-carousel/src/rmx-carousel.js`
- `webcomps/rmx-map-icon/src/rmx-map-icon.js`
- `webcomps/rmx-markdown/src/rmx-markdown.ts`
- `webcomps/rmx-sortable/src/rmx-sortable.ts`
- `webcomps/rmx-sortable/src/index.js`
- `webcomps/rmx-sortable/src/Main.elm`
- `webcomps/rmx-timer/src/rmx-timer.js`
- `webcomps/rmx-tui-chart/src/rmx-tui-chart.tsx`
- `webcomps/rmx-tui-chart/src/App.tsx`
- `webcomps/rmx-tui-chart/src/index.tsx`
- `webcomps/rmx-tui-grid/src/rmx-tui-grid.tsx`
- `webcomps/rmx-tui-grid/src/App.tsx`
- `webcomps/rmx-tui-grid/src/index.tsx`
- `webcomps/rmx-video/src/rmx-video.ts`
- `webcomps/vega-embed-simple/src/vega-embed-simple.js`

## Layer 4 — Build & Config (medium priority)

Per-component Vite configs, tsconfigs, deploy scripts, and bundled output:

- `webcomps/dev-test/vite.config.ts`
- `webcomps/dev-test/tsconfig.json`
- `webcomps/dev-test/tsconfig.node.json`
- `webcomps/emoji-picker/vite.config.js`
- `webcomps/emoji-picker/tsconfig.json`
- `webcomps/map-polygons/vite.config.js`
- `webcomps/rmx-carousel/vite.config.js`
- `webcomps/rmx-map-icon/vite.config.js`
- `webcomps/rmx-markdown/vite.config.js`
- `webcomps/rmx-markdown/tsconfig.json`
- `webcomps/rmx-sortable/vite.config.js`
- `webcomps/rmx-sortable/elm.json`
- `webcomps/rmx-sortable/tsconfig.json`
- `webcomps/rmx-timer/vite.config.js`
- `webcomps/rmx-tui-chart/vite.config.ts`
- `webcomps/rmx-tui-chart/tsconfig.json`
- `webcomps/rmx-tui-grid/vite.config.ts`
- `webcomps/rmx-tui-grid/tsconfig.json`
- `webcomps/rmx-video/vite.config.js`
- `webcomps/rmx-video/tsconfig.json`
- `webcomps/vega-embed-simple/vite.config.ts`
- `webcomps/image_base64_uploader/copy_to_r2.sh`
- `webcomps/dev-test/bundle.js`
- `webcomps/emoji-picker/bundle.js`
- `webcomps/rmx-map-icon/bundle.js`
- `webcomps/rmx-timer/bundle.js`
- `webcomps/rmx-video/bundle.js`
- `webcomps/vega-embed-simple/bundle.mjs`

## Layer 5 — Supporting (low priority, skip if budget tight)

READMEs, demo HTML, CSS, assets, and deprecated components:

- `webcomps/dev-test/README.md`
- `webcomps/emoji-picker/README.md`
- `webcomps/map-polygons/README.md`
- `webcomps/rmx-carousel/README.md`
- `webcomps/rmx-markdown/README.md`
- `webcomps/rmx-sortable/README.md`
- `webcomps/rmx-tui-chart/README.md`
- `webcomps/rmx-tui-grid/README.md`
- `webcomps/rmx-map-icon/README.md`
- `webcomps/rmx-video/README.md`
- `webcomps/vega-embed-simple/README.md`
- `webcomps/dev-test/src/assets/harness.css`
- `webcomps/rmx-markdown/src/style.css`
- `webcomps/rmx-tui-chart/src/App.module.css`
- `webcomps/rmx-tui-chart/src/index.css`
- `webcomps/rmx-tui-grid/src/App.module.css`
- `webcomps/rmx-tui-grid/src/index.css`
- `webcomps/rmx-video/src/index.css`
- Per-component `index.html` files (demo/dev pages)
- `old/markdown-parser/*` (deprecated)
- `old/markdown-text-editor/*` (deprecated)
- `old/rmx-date-calendar/*` (deprecated, marked DEPRECATED)
- `old/zoom-timer/*` (deprecated)

## Excluded (do not read)

- `package-lock.json` (root)
- `webcomps/*/package-lock.json` (per-component lock files)
- `webcomps/rmx-tui-chart/pnpm-lock.yaml`
- `webcomps/rmx-tui-grid/pnpm-lock.yaml`
- `*.png`, `*.gif`, `*.svg`, `*.ico` (thumbnails, icons, logos)
- `webcomps/rmx-tui-chart/src/manifest.local.json` (local dev only)
- `webcomps/rmx-tui-grid/src/manifest.json` (duplicate of public/)
- `webcomps/rmx-tui-chart/src/manifest.json` (duplicate of public/)
- `webcomps/rmx-tui-chart/public/rmx-tui-chart.js` (pre-built output)
- `webcomps/rmx-tui-grid/public/rmx-tui-grid.js` (pre-built output)
- `old/rmx-date-calendar/dist/*` (deprecated build output)
- `webcomps/*/src/vite-env.d.ts` (Vite boilerplate type declarations)
- `webcomps/rmx-sortable/index.orig.html` (original HTML, superseded by index.html)
- `.gitignore` files
