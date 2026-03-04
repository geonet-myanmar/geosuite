# GeoSuite — Architecture Documentation

## Design Philosophy

GeoSuite is designed as a **zero-install, single-file web application**. The entire app lives in one `index.html` file with no build step, no bundler, no package manager, and no server requirement. This makes deployment trivial (GitHub Pages, any static host, or just double-clicking the file) and keeps the barrier to entry as low as possible for GIS users.

## High-Level Architecture

```
┌──────────────────────────────────────────────────────────┐
│                        BROWSER                           │
│                                                          │
│  ┌────────────────── index.html ──────────────────────┐  │
│  │                                                    │  │
│  │  ┌──────────┐  ┌───────────┐  ┌────────────────┐  │  │
│  │  │  <style>  │  │  <body>   │  │   <script>     │  │  │
│  │  │  ~550 LOC │  │  ~400 LOC │  │   ~1500 LOC    │  │  │
│  │  └──────────┘  └───────────┘  └────────────────┘  │  │
│  │                                                    │  │
│  │  ┌──────────────────────────────────────────────┐  │  │
│  │  │              SHARED LAYER                    │  │  │
│  │  │  Utilities │ AppBridge │ Mode Switcher       │  │  │
│  │  └──────────────────────────────────────────────┘  │  │
│  │         │                           │              │  │
│  │  ┌──────┴───────┐          ┌───────┴──────────┐   │  │
│  │  │  GEOREF MODE │          │  DIGITIZER MODE  │   │  │
│  │  │              │  Bridge  │                   │   │  │
│  │  │  Canvas      │ ──────► │  Leaflet Map      │   │  │
│  │  │  GCP Mgmt    │         │  Draw Tools       │   │  │
│  │  │  Affine Xfm  │         │  Layer Mgmt       │   │  │
│  │  │  Exports     │         │  Measurements     │   │  │
│  │  └──────────────┘         └───────────────────┘   │  │
│  │                                                    │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  CDN: Leaflet │ Leaflet.Draw │ geotiff.js │ Google Fonts │
└──────────────────────────────────────────────────────────┘
```

## Mode System

The app uses a simple `display: none/grid` toggle to switch between modes. Only one mode is visible at a time.

```javascript
// Modes are toggled by setting display property
document.getElementById('georef-mode').style.display = 'grid';   // or 'none'
document.getElementById('digitizer-mode').style.display = 'grid'; // or 'none'
```

**Leaflet is lazy-initialized** — the map is only created when the user first switches to Digitizer mode. This avoids unnecessary resource usage and the common Leaflet issue of initializing inside a hidden container.

When switching to Digitizer mode, `map.invalidateSize()` is called after a short delay to force Leaflet to recalculate its container dimensions.

## AppBridge — Integration Between Modes

The AppBridge is a simple in-memory object that enables one-way data passing from GeoRef to Digitizer:

```javascript
var AppBridge = {
  pendingOverlay: null  // {dataURL, bounds: [[s,w],[n,e]], name}
};
```

### Data Flow: GeoRef → Digitizer

```
GeoRef Mode                          Digitizer Mode
───────────                          ──────────────
1. User places GCPs
2. Affine transform computed
3. Corner bounds calculated:
   TL = transform(0, 0)
   TR = transform(w, 0)
   BL = transform(0, h)
   BR = transform(w, h)
4. Image → canvas → dataURL
5. Store in AppBridge.pendingOverlay ──→ 6. receiveOverlay() reads AppBridge
                                         7. L.imageOverlay(dataURL, bounds)
                                         8. map.fitBounds(bounds)
                                         9. AppBridge.pendingOverlay = null
```

The bridge is checked every time the user switches to Digitizer mode. If `pendingOverlay` is non-null, `receiveOverlay()` consumes it.

## GeoRef Mode — Internals

### State Object

```javascript
var grState = {
  image: null,       // HTMLImageElement (loaded raster)
  fileName: '',      // Original filename for export naming
  gcps: [],          // [{id, pixelX, pixelY, lon, lat}, ...]
  nextId: 1,         // Auto-incrementing GCP ID
  tool: 'gcp',       // Current tool: 'gcp' | 'pan'
  scale: 1,          // Canvas zoom level
  offsetX: 0,        // Canvas X translation
  offsetY: 0,        // Canvas Y translation
  isPanning: false,   // Drag state
  panStartX/Y: 0,    // Drag start position
  panOffsetX/Y: 0,   // Drag start offset
};
```

### Canvas Rendering Pipeline

```
1. Clear canvas
2. Fill dark background (#1a1d26)
3. Save context
4. Apply translation (offsetX, offsetY) and scale
5. Draw image at (0, 0) in image space
6. Restore context
7. For each GCP:
   a. Transform pixel coords to screen coords
   b. Draw crosshair lines (±12px)
   c. Draw circle (8px radius)
   d. Draw label text
8. Update zoom percentage display
```

### Affine Transform

The transform maps pixel coordinates to geographic coordinates using 6 parameters:

```
lon = a * col + b * row + c
lat = d * col + e * row + f
```

Computed via **least squares** (minimum 3 GCPs, overdetermined with more):

```
Normal equations: (A^T A) x = A^T b

where A = [[col1, row1, 1],     b_lon = [lon1,     b_lat = [lat1,
            [col2, row2, 1],              lon2,               lat2,
            ...]                          ...]                ...]
```

The 3x3 system is solved via Gaussian elimination with partial pivoting.

### GeoTIFF Binary Encoder

The custom encoder produces a valid GeoTIFF without any library dependency. Memory layout:

```
Offset 0:   TIFF Header (8 bytes)
            - Byte order: 'II' (little-endian)
            - Magic: 42
            - IFD offset: 8

Offset 8:   IFD (2 + 13×12 + 4 = 162 bytes)
            - Tag count: 13
            - 13 IFD entries (12 bytes each)
            - Next IFD offset: 0 (no more IFDs)

Offset 170: Extra Data
            - BitsPerSample: [8, 8, 8] (6 bytes)
            - ModelPixelScale: [sx, sy, 0] (24 bytes)
            - ModelTiepoint: [0, 0, 0, geoX, geoY, 0] (48 bytes)
            - GeoKeyDirectory: [1,1,0,3, 1024,0,1,2, ...] (32 bytes)

Offset ~280: Pixel Data (W × H × 3 bytes, RGB interleaved)
```

IFD Tags written:
| Tag | Name | Value |
|-----|------|-------|
| 256 | ImageWidth | W |
| 257 | ImageLength | H |
| 258 | BitsPerSample | [8,8,8] |
| 259 | Compression | 1 (None) |
| 262 | PhotometricInterpretation | 2 (RGB) |
| 273 | StripOffsets | pixel data offset |
| 277 | SamplesPerPixel | 3 |
| 278 | RowsPerStrip | H |
| 279 | StripByteCounts | W×H×3 |
| 284 | PlanarConfiguration | 1 (Chunky) |
| 33550 | ModelPixelScaleTag | [scaleX, scaleY, 0] |
| 33922 | ModelTiepointTag | [0, 0, 0, originX, originY, 0] |
| 34735 | GeoKeyDirectoryTag | [version, keys...] |

## Digitizer Mode — Internals

### Leaflet Configuration

- **Map**: Standard `L.map` with default zoom control
- **Basemaps**: OSM (with `.dark-tiles` CSS filter) and Esri Satellite
- **Feature Group**: `L.FeatureGroup` for drawn vectors
- **Measure Group**: Separate `L.FeatureGroup` for measurement overlays
- **Draw Control**: Polygon, Polyline, Marker, Rectangle, Circle enabled

### Layer Management

All overlays and features are tracked in a flat array:

```javascript
var digiLayers = [
  {
    id: 1,              // Unique ID
    name: 'raster.tif', // Display name
    type: 'geotiff',    // 'geotiff' | 'raster' | 'vector'
    subtype: null,       // For vectors: 'polygon' | 'polyline' | 'marker' | etc.
    leafletLayer: L.imageOverlay(...),  // The actual Leaflet layer
    visible: true,       // Visibility state
    bounds: [[s,w],[n,e]] // Geographic bounds (for zoom-to)
  },
  ...
];
```

### GeoTIFF Display Bug Fix

**Before (broken):**
```javascript
// Used map viewport bounds — WRONG
var bounds = map.getBounds();
L.imageOverlay(url, bounds, {opacity: 0.7}).addTo(map);
```

**After (fixed):**
```javascript
// Extract actual geographic bounds from TIFF metadata
var bbox = image.getBoundingBox();
// bbox = [west, south, east, north]
if (bbox && bbox.some(v => v !== 0)) {
  bounds = [[bbox[1], bbox[0]], [bbox[3], bbox[2]]];
} else {
  // Fallback with warning
  bounds = map viewport bounds;
  showToast('Warning: no georeference metadata', 'warning');
}
```

`image.getBoundingBox()` from geotiff.js reads the ModelPixelScale and ModelTiepoint (or ModelTransformation) tags to compute the actual geographic extent of the raster.

### Measurement System

Measurement tools use a state machine:

```
IDLE ──click──→ MEASURING ──dblclick──→ RESULT (permanent on map)
  ↑                │
  └──cancel/clear──┘
```

- **Distance**: Accumulates `latlng.distanceTo()` along the polyline segments (Haversine formula, built into Leaflet)
- **Area**: Uses `computeGeodesicArea()` with the spherical excess formula on WGS 84 (R = 6,371,000 m)

## CSS Architecture

### Theme Variables

All colors are defined as CSS custom properties on `:root`. To create a light theme, override these variables.

```css
:root {
  --bg-primary: #0f1117;     /* Deepest background */
  --bg-secondary: #181b24;   /* Panels, sidebars */
  --bg-tertiary: #1e2230;    /* Tab bars, nested panels */
  --bg-elevated: #252a38;    /* Hover states, cards */
  --border: #2d3348;         /* All borders */
  --text-primary: #e8e6e1;   /* Main text */
  --text-secondary: #9a9ba3; /* Secondary text */
  --text-muted: #5f6170;     /* Labels, placeholders */
  --accent: #c9915a;         /* Primary action color */
  /* ... */
}
```

### Leaflet Overrides

Leaflet's default styles are aggressively overridden with `!important` to enforce the dark theme. This is necessary because Leaflet injects inline styles and has high-specificity selectors.

Overridden components:
- Zoom control buttons
- Layer control panel and labels
- Draw toolbar buttons and action menus
- Popup content wrapper and tip
- Attribution bar
- Control bars and separators

### Responsive Design

At `max-width: 900px`:
- GeoRef mode switches from `grid-template-columns: 1fr 360px` to `grid-template-rows: 50vh 1fr`
- Digitizer mode switches from `grid-template-columns: 300px 1fr` to `grid-template-rows: auto 1fr`
- Sidebar gets a max-height constraint

## Performance Considerations

### Memory

- Raster images are held as `HTMLImageElement` references (browser-managed)
- GeoTIFF export creates a full W×H×3 `ArrayBuffer` — can be large for high-res images
- `canvas.toDataURL()` for AppBridge creates a PNG string in memory
- Each `L.imageOverlay` holds a reference to the data URL

### Rendering

- Canvas redraws on every pan/zoom action (60fps capable for typical map images)
- GCP markers are drawn procedurally (no DOM elements on canvas)
- Leaflet handles its own rendering efficiently via tile layers and SVG/Canvas renderer

### Recommendations for Large Files

- Use Chrome for best memory handling
- Close other tabs when working with >50 MP rasters
- Export GeoTIFF before attempting other operations on very large images
- Consider tiling large images externally and serving via tile server for the Digitizer
