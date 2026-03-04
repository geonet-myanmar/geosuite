# Changelog

All notable changes to GeoSuite are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [1.0.0] - 2026-03-04

### Added

- **Unified dual-mode application** — GeoRef mode and Digitizer mode in a single HTML file
- **Dark theme** — Professional dark palette with CSS custom properties, DM Sans + JetBrains Mono fonts
- **Mode switcher** — Header toggle between GeoRef and Digitizer modes
- **AppBridge integration** — Send georeferenced images from GeoRef to Digitizer with computed geographic bounds

#### GeoRef Mode
- Canvas-based image viewer with zoom, pan, fit-to-view, and mouse wheel zoom with cursor pivot
- Drag & drop file upload supporting JPG, PNG, TIFF, BMP, WebP
- TIFF/GeoTIFF rendering via geotiff.js (1-band grayscale and 3/4-band RGB/RGBA)
- GCP (Ground Control Point) management with color-coded crosshair markers
- Quick Corner Bounds — auto-fill 4 corner GCPs from bounding coordinates
- Affine transform computation via least-squares with Gaussian elimination
- Settings panel — CRS selection, coordinate format, transform method, compression
- Export: GeoTIFF (.tif) with custom binary encoder
- Export: World File (.jgw/.pgw/.tfw)
- Export: GCP File (.points) for QGIS
- Export: GCP Table (.csv)
- Export: Projection File (.prj) with WKT definitions
- Export: Send to Digitizer (new)

#### Digitizer Mode
- Leaflet map with OpenStreetMap and Esri Satellite basemaps
- Dark-themed Leaflet controls — zoom, layers, draw toolbar, popups, attribution
- Draw tools — polygon, polyline, marker, rectangle, circle with edit/delete
- Interactive layer list with type icons, visibility toggles, zoom-to-extent, delete
- Opacity slider for raster overlays
- Measure Distance tool — click points, double-click to finish, shows km/mi/m
- Measure Area tool — click polygon, double-click to close, shows km²/ha/m²
- Feature click popup with editable name and save button
- GeoJSON import and export

### Fixed

- **GeoTIFF geographic positioning** — Now uses `image.getBoundingBox()` from geotiff.js to extract actual embedded georeference metadata instead of incorrectly using `map.getBounds()`. Falls back to viewport bounds with a warning toast when no metadata exists.
- **Multi-band TIFF rendering** — Properly handles 1-band grayscale, 3-band RGB, and 4-band RGBA GeoTIFFs

### Changed

- Complete UI redesign from plain white panel to dark theme with sidebar layout
- Digitizer sidebar reorganized into 3 tabs: Layers, Tools, Data
- GeoTIFF overlay now auto-fits map to raster bounds after loading
- Vector features now tracked in a layer management system instead of a simple text list

### Removed

- Original plain white `<div class="panel">` UI
- `alert()`-based measurement placeholders
- `prompt()`-based feature naming (replaced with popup UI)
