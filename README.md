# UQ Graduation Slam Book

An interactive, page-flipping graduation “slam book” built with vanilla HTML, CSS, and JavaScript. The project renders high-resolution spreads as a 3D book that readers open by clicking through the cover and subsequent pages.

---

## Project Structure

| Path | Purpose |
| --- | --- |
| `index.html` | Self-contained implementation of the 3D book, including inline styles and JavaScript that drive the flipping interactions. |
| `index.css` | Optional standalone stylesheet showcasing an alternative typography-focused layout (not currently imported by `index.html`). |
| `Images/` | All cover and interior page assets (`cover.png`, `Page1.png`, etc.). Replace or extend this folder to update the visual content. |

---

## Public Surface Area

### DOM Components

| Element / Class | Description | Key Attributes / Notes |
| --- | --- | --- |
| `div.book-container` | Root element that defines the book’s footprint and sets the 3D `perspective`. Adjust its width/height (or media queries) to change the physical size of the book. |
| `div.book-cover` | Represents both the front and back cover. Contains two `<img>` tags with classes `cover-front` and `cover-back`. Applying the `.flipped` class rotates the cover around the Y-axis. |
| `div.book-page` | Represents a sheet that holds two printed sides (`.page-front` and `.page-back`). Each sheet is paired with an ID (`page2`, `page4`, …) so JavaScript can reference it. |
| `.page-front`, `.page-back` | Hold the actual artwork per side. Back faces are pre-rotated with `transform: rotateY(180deg)` to ensure realistic flipping. |
| `.page-img` | Applied to each `<img>` tag to standardize sizing and `object-fit: cover`. Swap the underlying `src` value to change the art while preserving layout. |
| `.show` utility class | Added by JavaScript to pages that should be visible/interactive before they are flipped. Removes the initial `opacity: 0; visibility: hidden; pointer-events: none` state. |

### JavaScript Interaction API

All logic in `index.html` lives inside a single `<script>` block. The public “API” surfaces through DOM IDs and the state booleans that gate animations.

| Identifier | Type | Responsibility |
| --- | --- | --- |
| `cover`, `page2`, `page4`, `endPage` | DOM references resolved via `document.getElementById()`. Every interactable sheet **must** exist in the markup with a unique ID that matches the variable used in the script. (Note: `endPage` is referenced but not yet defined in the DOM—add a matching `div` if you want a final spread.) |
| `isCoverFlipped`, `isPage2Flipped`, `isPage4Flipped` | Boolean guards that prevent a page from re-flipping or replaying the reveal animation. Extend this pattern when adding additional spreads. |
| `cover.addEventListener('click', …)` | Flips the cover once and, after a ~600 ms delay, applies `.show` to `#page2` so the first interior page is ready. |
| `page2.addEventListener('click', …)` | Flips the first interior sheet. Once flipped, it shows `#page4`. |
| `page4.addEventListener('click', …)` | Flips the second interior sheet. Designed to reveal `endPage` (add that DOM node if needed). |

There are no global utility functions; the event listeners are the primary extension points. To add additional interactions, create new DOM nodes, booleans, and listeners that mirror the existing pattern.

---

## Interaction Lifecycle

1. **Initial render** – Only the cover is visible. Interior pages have `opacity: 0` until the script toggles `.show`.
2. **Cover click** – Adds `.flipped` to `#cover`, animating it with `transform: rotateY(-179deg)`. After 600 ms, `#page2` receives `.show`.
3. **Interior page clicks** – Each `div.book-page` flips once. After it flips, the script waits 600 ms before revealing the next sheet so physical stacking looks natural.
4. **End page (optional)** – Prepare an element with `id="endPage"` plus `div.book-page` markup to support the final reveal referenced in the script.

---

## Usage Instructions

### Launch the Experience

1. Ensure the `Images/` assets exist relative to `index.html`. Filenames are case-sensitive.
2. Open `index.html` directly in any modern browser, or serve the folder with a static server (e.g., `npx serve .`) to avoid CORS restrictions when loading local textures.
3. Click the cover to begin, then click each subsequent page to continue flipping.

### Customize Page Artwork

1. Replace `Images/cover.png`, `Images/Page1.png`, etc., with your own artwork while keeping the same names.
2. Alternatively, edit the `src` attributes within `index.html` to point to new image paths.
3. Maintain consistent aspect ratios to avoid stretching; the defaults assume roughly 390×510 px.

### Add Additional Spreads (Example)

1. **Markup**

```
<div class="book-page" id="page6">
  <div class="page-front">
    <img class="page-img" src="Images/Page6.png" alt="Page 6" />
  </div>
  <div class="page-back">
    <img class="page-img" src="Images/Page7.png" alt="Page 7" />
  </div>
</div>
```

2. **Script**

```html
<script>
const page6 = document.getElementById('page6');
let isPage6Flipped = false;

page4.addEventListener('click', () => {
  if (!isPage4Flipped) {
    page4.classList.add('flipped');
    isPage4Flipped = true;
    setTimeout(() => {
      page6.classList.add('show');
    }, 600);
  }
});

page6.addEventListener('click', () => {
  if (!isPage6Flipped) {
    page6.classList.add('flipped');
    isPage6Flipped = true;
  }
});
</script>
```

This mirrors the existing flip logic: each newly added sheet maintains its own “flipped” boolean and reveals the next sheet via `.show`.

---

## Styling Notes

- Media queries in `index.html` make the `book-container` responsive down to ~400 px widths. Adjust or consolidate the breakpoints for cleaner behavior on small phones.
- The standalone `index.css` file contains an alternative styling system (different typography, `.book` wrapper, `.page` classes). To use it, add `<link rel="stylesheet" href="index.css">` in `index.html` and align the markup with the CSS selectors in that file.
- All transforms rely on `transform-origin: left center` to emulate a physical book hinge. Changing the origin will dramatically affect the animation.

---

## Known Gaps / Extension Hooks

- **Missing `endPage` element** – The script references `const endPage = document.getElementById('endPage');` but no corresponding markup exists. Add a final `div.book-page` with that ID to avoid `null` references when `page4` completes.
- **Accessibility** – Consider adding keyboard handlers and ARIA roles so flipping is possible without a mouse and screen readers know which page is active.
- **Preloading** – For high-resolution imagery, add a preloader or service worker to avoid flashes of unloaded textures on slower networks.

With these touchpoints, you can confidently extend the slam book, integrate it into a larger site, or adapt the flipping mechanics for other interactive spreads.
