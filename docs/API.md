# GeoSuite — Internal Function Reference

This document lists all public JavaScript functions in `index.html` organized by module. These are all global functions accessible from the browser console or from `onclick` handlers.

---

## Shared Utilities

### `showToast(message, type)`
Displays a slide-in notification in the top-right corner.

| Parameter | Type | Description |
|-----------|------|-------------|
| `message` | `string` | Text to display |
| `type` | `string` | One of `'success'`, `'error'`, `'info'`, `'warning'` |

Auto-dismisses after 4 seconds.

---

### `updateStatus(type, text)`
Updates the shared status bar at the bottom of the app.

| Parameter | Type | Description |
|-----------|------|-------------|
| `type` | `string` | CSS class for the dot: `'ready'`, `'warning'`, or `''` |
| `text` | `string` | Status text to display |

---

### `downloadBlob(blob, filename)`
Triggers a browser file download.

| Parameter | Type | Description |
|-----------|------|-------------|
| `blob` | `Blob` | The file content |
| `filename` | `string` | Suggested filename for the download |

---

### `toggleHelp()`
Toggles the help modal overlay visibility. No parameters.

---

### `parseTiffToCanvas(arrayBuffer)`
Parses a TIFF/GeoTIFF ArrayBuffer and renders it to an off-screen canvas.

| Parameter | Type | Description |
|-----------|------|-------------|
| `arrayBuffer` | `ArrayBuffer` | Raw TIFF file bytes |

**Returns:** `Promise<{canvas, image, width, height}>` where `canvas` is an `HTMLCanvasElement` with the rendered image, `image` is the geotiff.js `GeoTIFFImage` object, and `width`/`height` are pixel dimensions.

Handles 1-band (grayscale), 3-band (RGB), and 4-band (RGBA) TIFFs.

---

### `leastSquares(A, b)`
Solves an overdetermined linear system via normal equations.

| Parameter | Type | Description |
|-----------|------|-------------|
| `A` | `number[][]` | Matrix of coefficients (n rows x m cols) |
| `b` | `number[]` | Right-hand side vector (n elements) |

**Returns:** `Float64Array` of m solution values, or `null` if the system is singular.

---

### `gaussianElimination(mat)`
Solves a square augmented matrix `[A|b]` using Gaussian elimination with partial pivoting.

| Parameter | Type | Description |
|-----------|------|-------------|
| `mat` | `number[][]` | Augmented matrix (n x n+1) |

**Returns:** `Float64Array` of n solution values, or `null` if singular.

---

### `computeGeodesicArea(latlngs)`
Computes the geodesic area of a polygon on a sphere using the spherical excess formula.

| Parameter | Type | Description |
|-----------|------|-------------|
| `latlngs` | `{lat, lng}[]` | Array of vertex coordinates |

**Returns:** `number` — Area in square meters (always positive).

Uses Earth radius R = 6,371,000 m.

---

## Mode Switching

### `switchMode(mode)`
Switches between GeoRef and Digitizer modes.

| Parameter | Type | Description |
|-----------|------|-------------|
| `mode` | `string` | `'georef'` or `'digitizer'` |

Behavior:
- Toggles `display` on mode containers
- Initializes Leaflet on first switch to Digitizer (lazy init)
- Calls `map.invalidateSize()` after switching to Digitizer
- Checks for and processes any pending AppBridge overlay

---

## GeoRef Mode

### `grHandleFile(file)`
Processes an uploaded file for the GeoRef canvas. Detects TIFF by extension and delegates to `parseTiffToCanvas()`, otherwise reads as a data URL.

| Parameter | Type | Description |
|-----------|------|-------------|
| `file` | `File` | The file from input or drag & drop |

---

### `grInitCanvas()`
Initializes the canvas after an image is loaded. Hides the upload overlay, shows the toolbar and info bar, calls `grZoomFit()`. No parameters.

---

### `grResizeCanvas()`
Resizes the canvas to match its container and re-renders. Called on `window.resize` and after mode switches. No parameters.

---

### `grZoomFit()`
Fits the image to the canvas viewport with 8% padding. No parameters.

---

### `grZoomIn()` / `grZoomOut()`
Zooms the canvas by a factor of 1.25x. No parameters.

---

### `grRender()`
Full canvas redraw: background, image (with current transform), and all GCP markers. No parameters.

---

### `setTool(tool)`
Sets the active canvas tool.

| Parameter | Type | Description |
|-----------|------|-------------|
| `tool` | `string` | `'gcp'` (place control points) or `'pan'` (drag to pan) |

Updates button active states and cursor.

---

### `addGcp(px, py)`
Creates a new GCP at the given pixel coordinates.

| Parameter | Type | Description |
|-----------|------|-------------|
| `px` | `number` | Pixel X coordinate on the image |
| `py` | `number` | Pixel Y coordinate on the image |

Assigns an auto-incrementing ID, adds to `grState.gcps`, re-renders list and canvas.

---

### `removeGcp(id)`
Removes a GCP by its ID.

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `number` | The GCP ID to remove |

---

### `clearAllGcps()`
Removes all GCPs and resets the ID counter. No parameters.

---

### `updateGcpCoord(id, field, value)`
Updates a geographic coordinate for a GCP.

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `number` | GCP ID |
| `field` | `string` | `'lon'` or `'lat'` |
| `value` | `string` | New value (parsed to float, or `null` if empty) |

---

### `renderGcpList()`
Rebuilds the GCP list HTML in the side panel. No parameters. Reads from `grState.gcps`.

---

### `updateGcpStatus()`
Updates the status bar and export tab status text based on how many GCPs have complete coordinates. No parameters.

---

### `applyCornerBounds()`
Reads the Quick Corner Bounds inputs (West, East, South, North) and assigns them to the first 4 GCPs sorted by position (NW, NE, SW, SE). No parameters.

Requires exactly 4 GCPs placed and all 4 bound fields filled.

---

### `computeAffineTransform()`
Computes the 6-parameter affine transform from complete GCPs.

**Returns:** `{a, b, c, d, e, f}` or `null` if fewer than 3 complete GCPs.

Where:
```
lon = a * pixelX + b * pixelY + c
lat = d * pixelX + e * pixelY + f
```

---

### `buildGeoTiffBuffer(rgbPixels, w, h, transform)`
Constructs a valid GeoTIFF ArrayBuffer from raw pixel data.

| Parameter | Type | Description |
|-----------|------|-------------|
| `rgbPixels` | `Uint8ClampedArray` | RGBA pixel data from `ImageData.data` |
| `w` | `number` | Image width |
| `h` | `number` | Image height |
| `transform` | `object` | Affine transform `{a, b, c, d, e, f}` |

**Returns:** `ArrayBuffer` — A valid, uncompressed RGB GeoTIFF.

---

### `exportGeoTIFF()`
Exports the georeferenced image as a GeoTIFF file. Requires 3+ complete GCPs. No parameters.

---

### `exportWorldFile()`
Exports a world file (`.jgw`, `.pgw`, `.tfw`, etc.) with affine parameters. No parameters.

---

### `exportGcpFile()`
Exports a QGIS-compatible `.points` GCP file. No parameters.

---

### `exportGcpCsv()`
Exports GCP coordinates as a CSV table. No parameters.

---

### `exportPrjFile()`
Exports a WKT projection definition file (`.prj`). No parameters.

---

### `grGetOutputName(ext)`
Generates an output filename based on the input filename.

| Parameter | Type | Description |
|-----------|------|-------------|
| `ext` | `string` | File extension including dot (e.g., `'.tif'`) |

**Returns:** `string` — e.g., `'mymap_georef.tif'`

---

### `grSwitchTab(tab)`
Switches the GeoRef side panel tab.

| Parameter | Type | Description |
|-----------|------|-------------|
| `tab` | `string` | `'gcps'`, `'settings'`, or `'export'` |

---

## Integration

### `sendToDigitizer()`
Computes affine transform corner bounds, creates a PNG data URL from the current image, stores it in `AppBridge.pendingOverlay`, and switches to Digitizer mode. No parameters.

Requires 3+ complete GCPs and a loaded image.

---

### `receiveOverlay()`
Reads `AppBridge.pendingOverlay`, creates an `L.imageOverlay` on the map, adds it to the layer list, and flies to the overlay bounds. No parameters.

Called automatically when switching to Digitizer mode if a pending overlay exists.

---

## Digitizer Mode

### `initLeaflet()`
Initializes the Leaflet map, basemaps, draw controls, feature group, measurement group, and all event handlers. No parameters.

Called once on first switch to Digitizer mode.

---

### `handleDigiGeoTiff(file)`
Loads a GeoTIFF file, renders it to canvas, extracts geographic bounds from metadata, and adds as an image overlay at the correct position.

| Parameter | Type | Description |
|-----------|------|-------------|
| `file` | `File` | The GeoTIFF file |

Falls back to viewport bounds with a warning if no georeference metadata exists.

---

### `handleDigiImage(file)`
Loads a plain image file and overlays it on the current map viewport.

| Parameter | Type | Description |
|-----------|------|-------------|
| `file` | `File` | The image file (JPG, PNG, etc.) |

---

### `updateDigiLayerList()`
Rebuilds the layer list HTML in the Digitizer sidebar. No parameters. Reads from `digiLayers` array.

---

### `toggleLayerVisibility(id)`
Toggles a layer's visibility on the map.

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `number` | Layer ID from `digiLayers` |

---

### `zoomToLayer(id)`
Fits the map to a layer's geographic extent.

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `number` | Layer ID from `digiLayers` |

---

### `removeLayer(id)`
Removes a layer from the map and the layer list.

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `number` | Layer ID from `digiLayers` |

---

### `showFeaturePopup(layer)`
Opens a Leaflet popup with an editable name input for the given feature layer.

| Parameter | Type | Description |
|-----------|------|-------------|
| `layer` | `L.Layer` | The Leaflet layer to edit |

---

### `saveFeatureName(btn)`
Saves the name from the popup input to the feature's properties and updates the layer list. Called from the popup's Save button.

| Parameter | Type | Description |
|-----------|------|-------------|
| `btn` | `HTMLElement` | The clicked button (used for context) |

---

### `startMeasureDistance()`
Activates the distance measurement tool. Click points on the map, double-click to finish. No parameters.

---

### `startMeasureArea()`
Activates the area measurement tool. Click polygon vertices, double-click to close and calculate. No parameters.

---

### `cancelMeasure()`
Cancels any active measurement and clears temporary overlays. No parameters.

---

### `clearMeasurements()`
Removes all measurement overlays (lines, polygons, popups) from the map. No parameters.

---

### `digiExportGeoJSON()`
Exports all drawn features as a GeoJSON file download. No parameters.

---

### `handleDigiGeojsonImport(file)`
Imports features from a GeoJSON file, adds them to the draw group and layer list.

| Parameter | Type | Description |
|-----------|------|-------------|
| `file` | `File` | The GeoJSON file |

---

### `digiSwitchTab(tab)`
Switches the Digitizer sidebar tab.

| Parameter | Type | Description |
|-----------|------|-------------|
| `tab` | `string` | `'layers'`, `'tools'`, or `'data'` |

---

## Global State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `currentMode` | `string` | Active mode: `'georef'` or `'digitizer'` |
| `leafletInitialized` | `boolean` | Whether Leaflet has been initialized |
| `grState` | `object` | GeoRef mode state (image, GCPs, canvas transform) |
| `GCP_COLORS` | `string[]` | 12 color hex codes for GCP markers |
| `AppBridge` | `object` | Integration bridge with `pendingOverlay` property |
| `map` | `L.Map` | Leaflet map instance (initialized lazily) |
| `drawn` | `L.FeatureGroup` | Feature group for drawn vectors |
| `measureGroup` | `L.FeatureGroup` | Feature group for measurement overlays |
| `digiLayers` | `object[]` | Array of all layers in the Digitizer |
| `digiNextId` | `number` | Auto-incrementing layer ID counter |
| `activeMeasureTool` | `string\|null` | Active measurement: `'distance'`, `'area'`, or `null` |
| `measurePoints` | `L.LatLng[]` | Points collected during active measurement |
