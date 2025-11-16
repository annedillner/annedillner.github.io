# Photo Selector Web App — Spec v2.2  
Generated: 2025-11-16

This document defines the behaviour and tunables for the static photo-selection web app (`index.html`).  
The `index.html` file is generated from this spec.  
Do **not** edit the HTML manually unless the spec is updated and version bumped.

---

## A. Directory layout and image IDs

Each photo session lives in its own directory, for example:

```
/photo-session-a-2025-11-14/
  ├─ photos/
  ├─ index.html
  └─ spec.md
```

- `CONFIG.baseDir` = `"photos/"`.
- Image IDs are **filenames only** (`IMG_0012.jpg` etc.).
- Filenames must be unique; duplicates cause a blocking error.

---

## B. URL format (live and deep-linkable)

The browser’s address bar reflects state using debounced `history.replaceState`.

| Param | Example | Meaning |
|-------|---------|---------|
| `sel` | `?sel=001.jpg,013.jpg` | comma-separated selected filenames |
| `only` | `&only=1` | show only selected |
| `open` | `&open=013.jpg` | open that photo in overlay |
| `lang` | `&lang=sv` | UI language |
| `v` | `&v=1` | optional version flag |

### Validation
- Any unknown filename in `sel` → show error, ignore all `sel`.
- Unknown `open` → show error, skip overlay open.
- Duplicate filenames in `CONFIG.images` → blocking error.

---

## C. Persistence

- `localStorage` key: `pg:<location.pathname>#<baseDir>#<imagesHash>`
- Load order:  
  1. URL (if valid)  
  2. localStorage (same key)  
  3. empty
- All state changes update both URL and localStorage.
- Tabs for the same session sync via the `storage` event.

---

## D. Selection and top bar

- Selection = set of filenames.  
- Grid and overlay show subtle checkmarks.

**Top bar** (sticky, visible everywhere):
- “N photos selected” (always shown, `aria-live="polite"`)
- **Copy link** – copies full URL  
- **Copy filenames** – newline-separated list  
- **Show only selected** toggle  
- **Clear selection**  
- **Lang** toggle (EN/SV)

When “Show only selected” is on, the grid filters immediately and URL gains `only=1`.

---

## E. Overlay (single image view)

- Opens from grid or via `?open=<filename>`.
- Closing removes `open` from URL and restores focus to that grid image.
- Navigation: Prev/Next buttons, Left/Right keys, swipe (paged animation).
- Filename row (under top bar) shows filename + “Copy filename” button.
- Preloads ±1 neighbour for responsiveness.

---

## F. Layout, visuals, and colour

- Responsive square grid (`object-fit: contain`).
- Grid and overlay content centred horizontally.
- Spinner while image loads; placeholder if load fails.
- Entire UI greyscale — no colour accents, no pure white/black.
- Thin border outlines each cell to show checkmark ownership.

### Tunables (CSS variables via JS config)

```
baseDir: "photos/"
approxCellSizePx: 180
cellPaddingPx: 8
gapPx: 8
bgColor: "#777"
cellBorderPx: 1
cellBorderColor: "rgba(255,255,255,0.25)"
uiGrayHigh: "rgba(255,255,255,0.85)"
uiGrayMid:  "rgba(255,255,255,0.55)"
uiGrayLow:  "rgba(255,255,255,0.25)"

minColumns: 2
maxColumns: 12
rounding: "nearest"

gridMaxScale: 1.0
overlayMaxScale: 1.0
overlayPaddingPx: 16
controlSizePx: 40
controlInsetPx: 12
swipeAnimMs: 220
maxVisiblePreload: 2

selectionEnabled: true
showOnlySelectedDefault: false
filenameInOverlay: true

localeDefault: "en"
```

---

## G. Keyboard and accessibility

**Grid** (`role="grid"`):  
- Arrow keys move focus.  
- Home/End to row edges.  
- PageUp/PageDown scroll and move focus.  
- **Enter** opens overlay.  
- **Space** toggles selection.

**Overlay** (`role="dialog"`, `aria-modal="true"`):  
- **Esc** closes.  
- **Left/Right** navigate.  
- **Space** toggles selection.

Focus outline always visible (high-contrast).  
Live regions announce selection changes and “Link copied”.  
All buttons have `aria-label`.

---

## H. Colour, format, and performance

- Use JPEG or WebP (sRGB, embedded ICC).  
- App does no colour conversion.  
- Lazy loading with spinners; browser handles bitmap memory.

---

## I. Error handling

- Unknown images → banner: “These images aren’t part of this gallery: …”
- Unknown `open` → banner: “Cannot open ‘…’ — not part of this gallery.”
- Duplicates in config → blocking error on startup.

---

## J. Maintenance notes

- **Spec-first workflow:**  
  - Keep this `spec.md` beside `index.html`.  
  - Regenerate `index.html` by pasting this spec into ChatGPT and asking:  
    “Generate index.html from this spec.”  
  - Update version number if behaviour or tunables change.  
  - Optionally keep old versions as `spec_v2.2.md`, `spec_v2.3.md`, etc.

- **HTML provenance:**  
  - Generated files begin with a comment:

    ```html
    <!--
    Photo Selector Web App
    Generated from spec v2.2 on YYYY-MM-DD
    Spec source: spec.md
    -->
    ```
---

_End of Spec v2.2_
