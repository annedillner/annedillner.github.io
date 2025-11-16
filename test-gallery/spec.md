# Photo Selector Web App — Spec v2.3.1
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

## D. Selection, grid, and top bar

- Selection = set of filenames.  
- Grid and overlay show subtle checkmarks.

**Tile interactions**
- **Single tap/click on the photo opens the overlay.**
- **Selection is toggled only via a dedicated corner selection box** — a real `<input type="checkbox">` per tile. Toggling selection does **not** open the overlay.

**Keyboard in grid**
- Arrow keys move focus between tiles.
- **Enter/Space on the tile** opens overlay.
- **Enter/Space on the selection box (checkbox)** toggles selection.
- Home/End to row edges. PageUp/PageDown scroll and move focus roughly a row.

**Top bar** (sticky, visible everywhere)
- “N photos selected” (always shown, `aria-live="polite"`)
- **Show only selected** — a **checkbox**. Checked = filter on (`only=1`), unchecked = show all.
- **Copy link / Share** — On devices that support Web Share API, offer **Share**; otherwise **Copy link**.
- **Copy filenames** — newline-separated list, **disabled when selection is empty**.
- **Clear selection** — **disabled when selection is empty**.
- **Lang** toggle (EN/SV).

When “Show only selected” is on, the grid filters immediately and URL gains `only=1`.

---

## E. Overlay (single image view)

- Opens from grid or via `?open=<filename>`.
- **Clicking/tapping the darkened background closes the overlay.**
- Closing removes `open` from URL and **restores focus to that grid image**.
- Navigation: Prev/Next buttons, Left/Right keys, **swipe (touch)**.
- **Paged slide animation** plays on any navigation (buttons/keys/swipe). Duration controlled by `swipeAnimMs`.
- (Optional convenience) Clicking the left/right thirds of the image area navigates to previous/next respectively.
- Filename row (under top bar) shows filename + “Copy filename” button.
- Preloads ±1 neighbour for responsiveness.

---

## F. Layout, visuals, and colour

- UI chrome uses neutral grayscale tones (no color accents).  
- **Photos retain original colors** — **do not** apply any CSS color filters to the page or images.
- Responsive square grid (`object-fit: contain`).
- Grid and overlay content centred horizontally.
- Spinner while image loads; placeholder if load fails.
- Thin border outlines each cell to show checkmark ownership.
- **Focused grid tile shows a darker background** for clarity.

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
focusTileBg: "rgba(0,0,0,0.28)"
```

---

## G. Keyboard and accessibility

**Tabbing**
- All interactive elements are tabbable in normal DOM order (top bar controls → banner → each grid **tile** → each tile’s **checkbox** → …).  
- Arrow keys move focus inside the grid but do **not** change Tab order.

**Grid** (`role="grid"`):
- Arrow keys move focus.  
- Home/End to row edges.  
- PageUp/PageDown scroll and move focus.  
- **Enter/Space on the tile** opens overlay.  
- **Enter/Space on the selection box** toggles selection.

**Overlay** (`role="dialog"`, `aria-modal="true"`):
- **Esc** or **background click** closes.  
- **Left/Right** navigate.  
- **Space** toggles selection.

Focus outline always visible (high-contrast).  
Live regions announce selection changes and “Link copied”.  
All buttons have `aria-label`.

---

## H. Colour, format, performance, and interaction hygiene

- Use JPEG or WebP (sRGB, embedded ICC). App does no colour conversion.
- Lazy loading with spinners; browser handles bitmap memory.
- **No unintended selection/drag**:
  - Disable text selection and image dragging for interactive surfaces (grid tiles, selection boxes, overlay stage).
  - Allow selection where copying makes sense (e.g., filename row).
  - Suggested CSS/attrs: `user-select: none` on interactive surfaces (override with `user-select: text` where needed); images `draggable="false"`, `-webkit-user-drag: none`; overlay stage `touch-action: none`.

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
  - Optionally keep old versions as `spec_v2.3.md`, `spec_v2.3.1.md`, etc.

- **HTML provenance:**  
  - Generated files begin with a comment:

    ```html
    <!--
    Photo Selector Web App
    Generated from spec v2.3.1 on YYYY-MM-DD
    Spec source: spec.md
    -->
    ```

- **Delivery requirement:**  
  - When this spec is regenerated in ChatGPT, **also provide it as an attached, downloadable `spec.md` file** in the reply (not just inline text).

---

_End of Spec v2.3.1_
