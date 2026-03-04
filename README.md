# GeoSuite — Unified GIS Web Application

A professional, browser-based GIS application that combines **raster georeferencing** and **map digitizing** into a single unified tool. No server required — runs entirely in the browser from a single HTML file.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![HTML](https://img.shields.io/badge/built%20with-HTML%2FCSS%2FJS-orange.svg)
![Zero Dependencies](https://img.shields.io/badge/install-zero%20install-green.svg)

## Overview

GeoSuite is a dual-mode single-page application with two integrated workspaces:

| Mode | Purpose |
|------|---------|
| **GeoRef** | Canvas-based raster georeferencing with GCP management, affine transforms, and multi-format export |
| **Digitizer** | Leaflet-based interactive map with vector drawing, raster overlays, measurement tools, and GeoJSON I/O |

Both modes share a dark theme UI, a status bar, and an **AppBridge** that lets you send georeferenced images from GeoRef directly onto the Digitizer map.

```
+-----------------------------------------------------------+
|  HEADER: Logo | [GeoRef] [Digitizer] switcher | Help      |
+-----------------------------------------------------------+
|  #georef-mode   (canvas + side panel)                     |
|       OR                                                  |
|  #digitizer-mode (left sidebar + Leaflet map)             |
+-----------------------------------------------------------+
|  Shared STATUS BAR                                        |
+-----------------------------------------------------------+
```

## Features

### GeoRef Mode

- **Image Upload** — Drag & drop or file picker; supports JPG, PNG, TIFF, BMP, WebP
- **TIFF Rendering** — Parses GeoTIFF/TIFF via `geotiff.js`, handles 1-band grayscale and 3/4-band RGB/RGBA
- **Canvas Navigation** — Zoom in/out, fit-to-view, mouse wheel zoom with cursor pivot, pan tool
- **GCP Management** — Click on image to place Ground Control Points with color-coded crosshair markers
- **Quick Corner Bounds** — Enter West/East/South/North coordinates, auto-assign to 4 corner GCPs
- **Affine Transform** — Least-squares 6-parameter affine computed from 3+ GCPs
- **Settings** — Output CRS selection (WGS 84, UTM 46N/47N/48N), coordinate format, transform method, compression
- **Export Formats:**
  - GeoTIFF (`.tif`) — Custom binary encoder with ModelPixelScale, ModelTiepoint, GeoKeyDirectory tags
  - World File (`.jgw` / `.pgw` / `.tfw`) — 6-line affine transform sidecar
  - GCP File (`.points`) — QGIS-compatible ground control point file
  - GCP Table (`.csv`) — Spreadsheet-friendly coordinate table
  - Projection File (`.prj`) — WKT projection definition
  - **Send to Digitizer** — Overlays the georeferenced image on the Leaflet map

### Digitizer Mode

- **Basemaps** — OpenStreetMap (dark-filtered) and Esri Satellite imagery, switchable via layer control
- **Draw Tools** — Polygon, Polyline, Marker, Rectangle, Circle with full edit/delete support via Leaflet.Draw
- **GeoTIFF Overlay (Fixed)** — Reads embedded georeference metadata via `image.getBoundingBox()` and places the raster at its correct geographic position; falls back to viewport bounds with a warning if metadata is missing
- **Image Overlay** — Plain JPG/PNG overlaid on current map extent
- **Layer Management** — Interactive layer list with type-specific icons, name display, visibility toggle, zoom-to-extent, and delete
- **Opacity Control** — Global slider for all raster overlays
- **Measurement Tools:**
  - Measure Distance — Click points, double-click to finish; shows km, miles, and meters
  - Measure Area — Click polygon vertices, double-click to close; shows km², hectares, and m²
- **Feature Naming** — Click any drawn feature to open an editable name popup
- **GeoJSON Import/Export** — Import `.geojson`/`.json` files, export all drawn features

### Integration

- **AppBridge** — In-memory bridge that passes image data + computed geographic bounds between modes
- **Send to Digitizer** — Computes affine transform corner bounds, creates a PNG dataURL, stores in AppBridge, switches to Digitizer mode
- **Receive Overlay** — Reads AppBridge, creates `L.imageOverlay` with correct bounds, flies to bounds
- **Mode Switching** — Calls `map.invalidateSize()` to keep Leaflet rendering correctly

### UI/UX

- **Dark Theme** — Professional dark palette with CSS custom properties
- **Typography** — DM Sans for UI text, JetBrains Mono for data/coordinates
- **Leaflet Dark Overrides** — All Leaflet controls, popups, draw toolbar, and attribution styled to match
- **Toast Notifications** — Color-coded (success/error/info/warning) slide-in toasts
- **Status Bar** — Live status indicator with colored dot
- **Help Modal** — Covers both modes with usage instructions
- **Responsive** — Adapts layout for screens under 900px

## Quick Start

### Option 1: Open Directly

```bash
# Just open index.html in any modern browser
open index.html
```

### Option 2: Local Server (recommended for TIFF handling)

```bash
# Python
python -m http.server 8000

# Node.js
npx serve .

# Then open http://localhost:8000
```

### Option 3: GitHub Pages

Push this repository to GitHub and enable GitHub Pages in Settings — the app will be live at `https://<username>.github.io/<repo>/`.

## Usage Guide

### Georeferencing a Raster Image

1. Open the app — starts in **GeoRef** mode
2. Drag & drop a raster image (JPG, PNG, or TIFF) onto the canvas
3. Click on 4 known locations on the image to place GCPs (corners work best)
4. **Option A:** Manually enter Longitude/Latitude for each GCP
5. **Option B:** Enter the bounding coordinates in **Quick Corner Bounds** and click "Apply Bounds to 4 GCPs"
6. Go to the **Export** tab to download in your preferred format
7. Or click **Send to Digitizer** to overlay the image on the interactive map

### Digitizing Features on the Map

1. Switch to **Digitizer** mode using the header toggle
2. Optionally upload a GeoTIFF (auto-positioned) or plain image as a background
3. Use the Leaflet draw toolbar on the right to create polygons, lines, markers, rectangles, or circles
4. Click any feature to edit its name
5. Use **Measure Distance** or **Measure Area** from the Tools tab
6. Export all features as GeoJSON from the Data tab

### Loading a GeoTIFF with Georeference

1. In Digitizer mode, go to the **Layers** tab
2. Upload a `.tif` file under "GeoTIFF (.tif)"
3. If the file contains valid georeference metadata, it will be placed at the correct geographic location
4. If not, it falls back to the current viewport with a warning
5. Use the opacity slider to adjust transparency
6. Manage layers in the Layer List (visibility, zoom-to, delete)

## Technical Architecture

### Single-File Design

The entire application is contained in one `index.html` file (~2,500 lines) with no build step, no bundler, and no local dependencies. All external libraries are loaded via CDN.

### External Libraries (CDN)

| Library | Version | Purpose |
|---------|---------|---------|
| [Leaflet](https://leafletjs.com/) | 1.9.4 | Interactive map |
| [Leaflet.Draw](https://github.com/Leaflet/Leaflet.draw) | 1.0.4 | Vector drawing tools |
| [geotiff.js](https://geotiffjs.github.io/) | 2.1.3 | GeoTIFF parsing and metadata extraction |
| [DM Sans](https://fonts.google.com/specimen/DM+Sans) | — | UI typeface |
| [JetBrains Mono](https://fonts.google.com/specimen/JetBrains+Mono) | — | Monospace typeface for data |

### Key Components

```
index.html
├── <style>          — All CSS (~550 lines)
│   ├── Theme variables (CSS custom properties)
│   ├── Header, buttons, tabs, toast, modal, status bar
│   ├── GeoRef mode styles (canvas, GCP cards, settings, export)
│   ├── Digitizer mode styles (sidebar, layer list, tools)
│   └── Leaflet dark theme overrides
├── <body>           — All HTML (~400 lines)
│   ├── Header with mode switcher
│   ├── #georef-mode (canvas panel + side panel with 3 tabs)
│   ├── #digitizer-mode (sidebar with 3 tabs + map container)
│   ├── Status bar, toast container, help modal
└── <script>         — All JavaScript (~1,500 lines)
    ├── Shared utilities (toast, status, download, TIFF parser, math)
    ├── AppBridge (integration object)
    ├── Mode switching logic
    ├── GeoRef: state, file upload, canvas rendering, GCP management
    ├── GeoRef: affine transform, GeoTIFF binary encoder, all exports
    ├── Digitizer: Leaflet init, draw tools, layer management
    ├── Digitizer: GeoTIFF handler (fixed), image handler
    ├── Digitizer: measurement tools, GeoJSON I/O
    └── Integration: sendToDigitizer, receiveOverlay
```

### Math Utilities

- **`leastSquares(A, b)`** — Solves overdetermined systems via normal equations (A^T A)x = A^T b
- **`gaussianElimination(mat)`** — 3x3 solver with partial pivoting
- **`computeAffineTransform()`** — Produces 6 affine parameters from GCPs: `lon = a*col + b*row + c`, `lat = d*col + e*row + f`
- **`computeGeodesicArea(latlngs)`** — Spherical excess formula for geodesic area on WGS 84

### GeoTIFF Binary Encoder

The app includes a **custom GeoTIFF writer** that produces valid, uncompressed RGB GeoTIFFs readable by QGIS, ArcGIS, GDAL, and rasterio. It writes:

- Standard TIFF header (little-endian, magic 42)
- 13-entry IFD with ImageWidth, ImageLength, BitsPerSample, Compression, PhotometricInterpretation, StripOffsets, SamplesPerPixel, RowsPerStrip, StripByteCounts, PlanarConfiguration
- GeoTIFF extension tags: ModelPixelScaleTag (33550), ModelTiepointTag (33922), GeoKeyDirectoryTag (34735)
- GeoKey entries for GTModelTypeGeoKey (Geographic), GTRasterTypeGeoKey (PixelIsArea), GeographicTypeGeoKey (4326/WGS 84)

## Browser Compatibility

| Browser | Supported |
|---------|-----------|
| Chrome 90+ | Yes |
| Firefox 90+ | Yes |
| Edge 90+ | Yes |
| Safari 15+ | Yes |
| Mobile browsers | Partial (responsive layout, touch not optimized) |

Requires: ES6+, Canvas API, FileReader API, ArrayBuffer, Blob/URL APIs.

## Project Structure

```
geosuite/
├── index.html          # Complete application (single file)
├── README.md           # This file
├── LICENSE             # MIT License
├── CONTRIBUTING.md     # Contribution guidelines
├── CHANGELOG.md        # Version history
└── docs/
    ├── ARCHITECTURE.md # Technical deep dive
    └── API.md          # Internal function reference
```

## Known Limitations

- **No projection conversion** — Coordinates are assumed WGS 84 (EPSG:4326) for map display. UTM CRS options affect export metadata only.
- **Affine transform only** — Higher-order polynomial transforms are not yet implemented.
- **GeoTIFF compression** — Export always produces uncompressed TIFFs regardless of the compression setting (LZW/Deflate not implemented in the binary encoder).
- **Large files** — Very large rasters (>100 MP) may cause browser memory issues. Canvas `toDataURL()` and the binary encoder operate in-memory.
- **No persistent storage** — All data is session-only. Closing the tab loses all work. Export before closing.
- **Touch devices** — Canvas GCP placement and measurement tools are not optimized for touch input.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

## Credits

- Built with [Leaflet](https://leafletjs.com/), [Leaflet.Draw](https://github.com/Leaflet/Leaflet.draw), and [geotiff.js](https://geotiffjs.github.io/)
- GeoRef mode originally from [geonet-myanmar/georef](https://github.com/geonet-myanmar/georef)
- Dark theme design inspired by modern development tools
- Fonts: [DM Sans](https://fonts.google.com/specimen/DM+Sans) and [JetBrains Mono](https://fonts.google.com/specimen/JetBrains+Mono) via Google Fonts
