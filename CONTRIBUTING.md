# Contributing to GeoSuite

Thank you for your interest in contributing to GeoSuite! This document provides guidelines and instructions for contributing.

## How to Contribute

### Reporting Bugs

1. Check the [existing issues](../../issues) to avoid duplicates
2. Open a new issue with:
   - A clear, descriptive title
   - Steps to reproduce the bug
   - Expected vs actual behavior
   - Browser name and version
   - Screenshots if applicable
   - Sample files that trigger the issue (if relevant)

### Suggesting Features

1. Open an issue with the `enhancement` label
2. Describe the use case — what problem does this feature solve?
3. Provide mockups or examples if possible

### Submitting Changes

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Make your changes to `index.html`
4. Test thoroughly in at least 2 browsers (Chrome + Firefox recommended)
5. Commit with a clear message: `git commit -m "Add: description of change"`
6. Push to your fork: `git push origin feature/your-feature-name`
7. Open a Pull Request with:
   - Description of what changed and why
   - Screenshots/recordings for UI changes
   - Testing steps for reviewers

## Development Guidelines

### Code Style

- **Single-file architecture** — All changes go into `index.html`. Do not split into separate JS/CSS files.
- **Vanilla JS** — No frameworks or build tools. Keep it zero-install.
- **Function declarations** — Use `function name()` over arrow functions for top-level functions (broader browser compat).
- **Descriptive names** — Prefix GeoRef functions with `gr` and Digitizer functions with `digi` to avoid collisions.
- **Comments** — Use `// ====` section dividers for major sections. Inline comments only where logic isn't obvious.

### CSS Conventions

- Use CSS custom properties (defined in `:root`) for all colors
- Follow the existing naming: `.georef-*` for GeoRef, `.digi-*` for Digitizer, shared classes unscoped
- Keep specificity low — avoid `!important` except for Leaflet overrides (required)

### Testing Checklist

Before submitting a PR, verify:

- [ ] App loads without console errors
- [ ] GeoRef: Upload JPG/PNG/TIFF — image renders on canvas
- [ ] GeoRef: Place 4 GCPs, use Quick Corner Bounds — coordinates auto-fill
- [ ] GeoRef: Export GeoTIFF — file downloads, opens in QGIS/GDAL
- [ ] GeoRef: Send to Digitizer — mode switches, image overlays correctly
- [ ] Digitizer: Upload GeoTIFF with georeference — placed at correct location
- [ ] Digitizer: Upload plain image — placed at viewport with warning
- [ ] Digitizer: Draw polygon/line/marker/rectangle/circle — features appear
- [ ] Digitizer: Click feature, edit name, save — name updates
- [ ] Digitizer: Measure Distance — popup shows km/mi/m
- [ ] Digitizer: Measure Area — popup shows km²/ha/m²
- [ ] Digitizer: Import GeoJSON — features load on map
- [ ] Digitizer: Export GeoJSON — valid file downloads
- [ ] Layer list: Visibility toggle, zoom-to, delete all work
- [ ] Mode switching: Both modes remain functional, map doesn't break
- [ ] Responsive: No layout breakage below 900px

### Commit Message Format

```
Type: Short description

Longer explanation if needed.
```

Types:
- `Add` — New feature
- `Fix` — Bug fix
- `Update` — Enhancement to existing feature
- `Refactor` — Code restructuring without behavior change
- `Docs` — Documentation only
- `Style` — CSS/visual changes only

### Areas Where Help is Welcome

- **Touch/mobile optimization** — Canvas GCP placement and measurement tools on touch devices
- **GeoTIFF compression** — Implementing LZW or Deflate in the binary encoder
- **Higher-order transforms** — 2nd/3rd order polynomial transforms in GeoRef
- **Projection support** — Client-side reprojection (proj4js integration)
- **Undo/redo** — State history for both modes
- **Localization** — i18n support for the UI
- **Accessibility** — ARIA labels, keyboard navigation, screen reader support
- **Performance** — Web Worker support for large raster processing

## Code of Conduct

Be respectful, constructive, and inclusive. We are all here to build useful geospatial tools.

## Questions?

Open an issue with the `question` label or start a discussion.
