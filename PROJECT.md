---
status: active
doc_type: project-notes
owner: engineering
last_reviewed: 2026-07-27
app_id: n/a
project_state: active
source_of_truth:
  - CustomTools/index.html
  - AGENTS.md
---

# sports-data-compare Project Notes

## Scope

- Type: single-file H5 web tool (all CSS and JS inlined into one HTML file, no build step).
- Purpose: compare heart-rate data from FIT files and GPS track data from GPX files exported by different sport watches.
- Status: active.
- Entry file: `index.html` (~1.9 MB, 1809 lines; browser tab title `运动数据对比工具`).
- How it runs: open `index.html` directly in a web browser (double-click). No compilation or server is required.

## Current Behavior

- The UI is split into two panels selected by a tab bar. Switching tabs toggles the active panel and triggers a delayed resize of the GPS map or the heart-rate chart so each renders correctly when first shown.
- Heart-rate comparison (`HRCompare` module):
  - Upload `.fit` files by drag-and-drop or click, up to 10 files.
  - Per-file overview card shows average / maximum / minimum heart rate, duration, valid point count, raw zero-value count, and patched-point count.
  - Timeline alignment: relative time starts at 0; the comparison window spans the longest uploaded file's duration (`maxDuration = Math.max(...)`, clamped to a minimum window), and every series renders its full points.
  - Gap filling: missing/blank values are forward-filled then back-filled, and the number of patched points is reported.
  - ECharts curve chart with `dataZoom` wheel zoom and drag pan, crosshair tooltip, PNG export, and reset-view control.
  - Pearson correlation matrix across files, color-coded (>=90% green, >=70% orange, <70% red).
- GPS track comparison (`GPSCompare` module):
  - Upload `.gpx` files by drag-and-drop or click.
  - OpenLayers map using Amap (Gaode) raster tiles that need no API key: a road layer and a satellite layer.
  - Track points are converted WGS-84 -> GCJ-02 (`Utils.wgs84ToGcj02`) before rendering so tracks align with the Amap tiles.
  - The map auto-fits its view to the bounds of the loaded tracks.
  - Toggle between standard (road) and satellite base maps.
- Inline module order inside `index.html`: OpenLayers CSS, custom CSS, ECharts library, OpenLayers library, `Utils`, `FitReader`, `GpxReader`, `HRCompare`, `GPSCompare`, then the tab-switching script.

## Build And Verification

- No build step. Verify locally by opening `index.html` in a browser; edits are made directly in that single file.
- Deployment target is GitHub Pages at `https://zh40s05.github.io/sports-data-compare/`; the repository was made public to enable Pages, and pushing to `master` updates the published page.
- This directory is a Git submodule; commits and pushes are handled inside `CustomTools/` separately from the parent repository.

## Local Decisions

- Use OpenLayers 10 for maps instead of Leaflet; Leaflet must not be reintroduced.
- Use Amap (Gaode) tiles because they are free and require no API key.
- Parse FIT data with a self-written parser (`FitReader.parseFIT`) that extracts only the timestamp and heart_rate from record message (#20), rather than pulling in a FIT library.
- Parse GPX with the browser's built-in `DOMParser`.
- Keep the single-file, fully-inlined architecture; do not split into a multi-file build system unless the user explicitly asks.
- Inline the chart and map libraries so the heart-rate feature remains fully usable offline.
- Git handling: the legacy `CLAUDE.md` documented a destructive rebuild-and-force-push single-commit flow; per `AGENTS.md`, agents should not run those destructive Git operations automatically and must only do so on explicit user request after explaining the impact.

## Dependencies And Reuse

- Inlined ECharts 5.5 (~1 MB minified) for the heart-rate curve chart.
- Inlined OpenLayers 10 (~900 KB) for the GPS map.
- Self-written `FitReader` module (~250 lines) implementing the full FIT base types (0-16) and compressed timestamps, exposing `parseFIT()`.
- Browser `DOMParser` for GPX parsing (`GpxReader` module).
- `Utils` module: color helpers, Pearson correlation, time formatting, WGS-84 -> GCJ-02 conversion, and HTML escaping.
- Data formats parsed:
  - FIT: binary; only record message (#20) `timestamp` and `heart_rate` are extracted.
  - GPX: XML; `<trkpt>` `lat` / `lon` / `ele` / `time` are extracted.
- Network dependency: only the Amap map tiles require network access; ECharts and OpenLayers are inlined.

## Open Issues

- Map tiles require network; when offline the GPS base map is unavailable (renders gray) although track lines still display, while the heart-rate feature works fully offline.
- Heart-rate comparison accepts at most 10 `.fit` files per session.
