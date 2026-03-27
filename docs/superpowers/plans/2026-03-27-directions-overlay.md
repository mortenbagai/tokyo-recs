# Directions Overlay Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a "Directions" modal to restaurant/bar/wine/coffee entries that shows an embedded Google Maps route from any hotel in the guide.

**Architecture:** Single shared modal in the DOM, populated dynamically from the existing `places` array. Directions links injected by JS (not hard-coded in HTML) for all non-hotel entries. Google Maps Embed API iframe displays the route. Hotel selection persisted in `localStorage`.

**Tech Stack:** Vanilla JS, CSS, Google Maps Embed API (free, no billing charges). All inline in `index.html`.

**Spec:** `docs/superpowers/specs/2026-03-27-directions-overlay-design.md`

---

### Task 1: Replace the demo API key with the billing-enabled key

**Files:**
- Modify: `index.html:1469` (the Maps JS API script tag)

- [ ] **Step 1: Replace the API key**

Find:
```html
<script async src="https://maps.googleapis.com/maps/api/js?key=AIzaSyDoOngUdwQvcia2lExJYWqGh6ezRi9KhPk&callback=initMap"></script>
```

Replace with:
```html
<script async src="https://maps.googleapis.com/maps/api/js?key=AIzaSyBDS_uyZVxZK8wxFNESVP3RITS3UAgoLcs&callback=initMap"></script>
```

- [ ] **Step 2: Verify the map still loads**

Run: `python3 -m http.server` and open `http://localhost:8000` in a browser.
Expected: The embedded Google Map with all markers loads correctly, same as before.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Replace demo Maps API key with billing-enabled key"
```

---

### Task 2: Add modal CSS styles

**Files:**
- Modify: `index.html` — add styles inside the existing `<style>` block, before the closing `</style>` tag (currently line 478)

- [ ] **Step 1: Add the modal CSS**

Insert the following CSS just before the `</style>` closing tag (before line 478):

```css
  /* Directions modal */
  .directions-backdrop {
    display: none;
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.4);
    z-index: 1000;
    align-items: center;
    justify-content: center;
  }
  .directions-backdrop.open {
    display: flex;
  }
  .directions-modal {
    background: var(--paper);
    border-radius: 12px;
    width: 440px;
    max-width: calc(100vw - 32px);
    max-height: calc(100vh - 64px);
    overflow: hidden;
    box-shadow: 0 8px 32px rgba(0,0,0,0.18);
    display: flex;
    flex-direction: column;
  }
  .directions-header {
    padding: 16px 20px;
    border-bottom: 1px solid var(--divider);
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .directions-header h3 {
    font-family: 'Cormorant Garamond', serif;
    font-size: 20px;
    font-weight: 600;
    color: var(--ink);
    margin: 0;
  }
  .directions-close {
    background: none;
    border: none;
    font-size: 20px;
    color: var(--muted);
    cursor: pointer;
    padding: 4px 8px;
    line-height: 1;
  }
  .directions-close:hover { color: var(--ink); }
  .directions-hotel {
    padding: 12px 20px;
    border-bottom: 1px solid var(--divider);
  }
  .directions-hotel label {
    display: block;
    font-family: 'DM Sans', sans-serif;
    font-size: 11px;
    text-transform: uppercase;
    letter-spacing: 1px;
    color: var(--muted);
    margin-bottom: 6px;
  }
  .directions-hotel select {
    width: 100%;
    padding: 8px 10px;
    border: 1px solid var(--divider);
    border-radius: 6px;
    font-size: 14px;
    font-family: 'DM Sans', sans-serif;
    background: white;
    color: var(--ink);
  }
  .directions-modes {
    padding: 12px 20px;
    display: flex;
    gap: 8px;
    border-bottom: 1px solid var(--divider);
  }
  .directions-mode-btn {
    flex: 1;
    text-align: center;
    padding: 8px;
    border-radius: 6px;
    font-size: 13px;
    font-family: 'DM Sans', sans-serif;
    font-weight: 500;
    cursor: pointer;
    border: none;
    background: var(--tag-bg);
    color: var(--ink);
    transition: background 0.15s, color 0.15s;
  }
  .directions-mode-btn.active {
    background: var(--hotel);
    color: white;
  }
  .directions-embed {
    width: 100%;
    height: 350px;
    border: none;
    display: block;
  }
  .directions-placeholder {
    height: 350px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: 'DM Sans', sans-serif;
    font-size: 14px;
    color: var(--muted);
    text-align: center;
    padding: 20px;
  }
```

- [ ] **Step 2: Verify styles don't break existing layout**

Run: Refresh `http://localhost:8000` in the browser.
Expected: Page looks identical — the modal styles have no effect until the modal HTML is added.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Add CSS styles for directions modal"
```

---

### Task 3: Add the modal HTML

**Files:**
- Modify: `index.html` — add modal markup just before the closing `</body>` tag (currently line 1471)

- [ ] **Step 1: Add modal HTML**

Insert the following just before `</body>` (before line 1471):

```html
<div class="directions-backdrop" id="directions-backdrop">
  <div class="directions-modal" onclick="event.stopPropagation()">
    <div class="directions-header">
      <h3 id="directions-title">Directions</h3>
      <button class="directions-close" onclick="closeDirections()">&times;</button>
    </div>
    <div class="directions-hotel">
      <label>From</label>
      <select id="directions-hotel-select" onchange="onHotelChange()">
        <option value="">Select your hotel...</option>
      </select>
    </div>
    <div class="directions-modes">
      <button class="directions-mode-btn active" data-mode="transit" onclick="setMode('transit')">🚇 Transit</button>
      <button class="directions-mode-btn" data-mode="driving" onclick="setMode('driving')">🚗 Car</button>
    </div>
    <div id="directions-embed-container">
      <div class="directions-placeholder">Select your hotel above to see directions</div>
    </div>
  </div>
</div>
```

- [ ] **Step 2: Verify modal is hidden by default**

Run: Refresh `http://localhost:8000`.
Expected: Page looks identical — modal has `display:none` via CSS.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Add directions modal HTML markup"
```

---

### Task 4: Add the modal JavaScript

**Files:**
- Modify: `index.html` — add a new `<script>` block after the `initMap` script closing tag (after line 1468 `</script>`)

- [ ] **Step 1: Add the directions JS**

Insert the following `<script>` block after the existing `</script>` tag that closes `initMap` (after the line `</script>` around line 1468), and before the Maps API `<script async ...>` tag:

```html
<script>
const MAPS_EMBED_KEY = 'AIzaSyBDS_uyZVxZK8wxFNESVP3RITS3UAgoLcs';

// Wait for initMap to run so `places` is available — it's defined inside initMap.
// We need to hoist `places` to be accessible outside. See Task 5 for the one-line fix.
// For now, this script sets up the modal functions.

let directionsDestSlug = null;
let directionsMode = 'transit';

function getPlaces() {
  return window._places || [];
}

function getHotels() {
  return getPlaces().filter(p => p.cat === 'h').sort((a, b) => a.name.localeCompare(b.name));
}

function getPlaceBySlug(slug) {
  return getPlaces().find(p => p.slug === slug);
}

function openDirections(slug) {
  const place = getPlaceBySlug(slug);
  if (!place) return;

  directionsDestSlug = slug;
  directionsMode = 'transit';

  // Set title
  document.getElementById('directions-title').textContent = 'Directions to ' + place.name;

  // Populate hotel dropdown
  const select = document.getElementById('directions-hotel-select');
  const savedHotel = localStorage.getItem('directions-hotel') || '';
  select.innerHTML = '<option value="">Select your hotel...</option>';
  getHotels().forEach(h => {
    const opt = document.createElement('option');
    opt.value = h.slug;
    opt.textContent = h.name;
    if (h.slug === savedHotel) opt.selected = true;
    select.appendChild(opt);
  });

  // Reset mode buttons
  document.querySelectorAll('.directions-mode-btn').forEach(btn => {
    btn.classList.toggle('active', btn.dataset.mode === 'transit');
  });

  // Show modal
  document.getElementById('directions-backdrop').classList.add('open');
  document.body.style.overflow = 'hidden';

  // If hotel was restored from localStorage, load the embed
  if (savedHotel && getPlaceBySlug(savedHotel)) {
    updateEmbed();
  } else {
    showPlaceholder();
  }
}

function closeDirections() {
  document.getElementById('directions-backdrop').classList.remove('open');
  document.body.style.overflow = '';
  // Clear iframe to stop any ongoing loads
  document.getElementById('directions-embed-container').innerHTML =
    '<div class="directions-placeholder">Select your hotel above to see directions</div>';
}

function onHotelChange() {
  const slug = document.getElementById('directions-hotel-select').value;
  if (slug) {
    localStorage.setItem('directions-hotel', slug);
    updateEmbed();
  } else {
    showPlaceholder();
  }
}

function setMode(mode) {
  directionsMode = mode;
  document.querySelectorAll('.directions-mode-btn').forEach(btn => {
    btn.classList.toggle('active', btn.dataset.mode === mode);
  });
  const hotelSlug = document.getElementById('directions-hotel-select').value;
  if (hotelSlug) updateEmbed();
}

function updateEmbed() {
  const hotelSlug = document.getElementById('directions-hotel-select').value;
  const hotel = getPlaceBySlug(hotelSlug);
  const dest = getPlaceBySlug(directionsDestSlug);
  if (!hotel || !dest) return;

  const src = 'https://www.google.com/maps/embed/v1/directions'
    + '?key=' + MAPS_EMBED_KEY
    + '&origin=' + hotel.lat + ',' + hotel.lng
    + '&destination=' + dest.lat + ',' + dest.lng
    + '&mode=' + directionsMode;

  document.getElementById('directions-embed-container').innerHTML =
    '<iframe class="directions-embed" src="' + src + '" allowfullscreen loading="lazy"></iframe>';
}

function showPlaceholder() {
  document.getElementById('directions-embed-container').innerHTML =
    '<div class="directions-placeholder">Select your hotel above to see directions</div>';
}

// Close on backdrop click
document.getElementById('directions-backdrop').addEventListener('click', closeDirections);

// Close on Escape
document.addEventListener('keydown', e => {
  if (e.key === 'Escape') closeDirections();
});

// Inject "Directions" links into all non-hotel entries
function injectDirectionsLinks() {
  const nonHotelPlaces = getPlaces().filter(p => p.cat !== 'h');
  nonHotelPlaces.forEach(p => {
    const entry = document.getElementById(p.slug);
    if (!entry) return;
    const meta = entry.querySelector('.entry-meta');
    if (!meta) return;
    // Don't add duplicate
    if (meta.querySelector('.directions-link')) return;
    const link = document.createElement('a');
    link.className = 'map-link directions-link';
    link.href = '#';
    link.textContent = '↗ Directions';
    link.addEventListener('click', e => {
      e.preventDefault();
      openDirections(p.slug);
    });
    meta.appendChild(link);
  });
}
</script>
```

- [ ] **Step 2: Verify no JS errors on page load**

Run: Refresh `http://localhost:8000` and check the browser console.
Expected: No errors. The `injectDirectionsLinks()` function is defined but not yet called (it needs `places` to be available — that's Task 5).

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Add directions modal JavaScript logic"
```

---

### Task 5: Hoist `places` array and wire up injection

**Files:**
- Modify: `index.html` — inside the `initMap` function, expose `places` to the global scope and call `injectDirectionsLinks()`

- [ ] **Step 1: Expose `places` to global scope**

Inside the `initMap` function, find the line (around line 1369):
```js
  const places = [
```

Replace with:
```js
  window._places = [
```

- [ ] **Step 2: Update local references to `places`**

In the same `initMap` function, find the line (around line 1405 after the array closes):
```js
  places.forEach(p => {
```

Replace with:
```js
  window._places.forEach(p => {
```

- [ ] **Step 3: Call injectDirectionsLinks at the end of initMap**

Find the closing of `initMap` — the line:
```js
  map.controls[google.maps.ControlPosition.BOTTOM_CENTER].push(legend);
}
```

Replace with:
```js
  map.controls[google.maps.ControlPosition.BOTTOM_CENTER].push(legend);

  // Inject directions links into non-hotel entries
  injectDirectionsLinks();
}
```

- [ ] **Step 4: Verify end-to-end flow**

Run: Refresh `http://localhost:8000`.

Expected:
1. Every restaurant, bar, wine, and coffee entry now shows a "↗ Directions" link in its meta row
2. Hotel entries do NOT have this link
3. Clicking "↗ Directions" on any entry opens the modal with the correct restaurant name
4. The hotel dropdown is populated with all 14 hotels in alphabetical order
5. Selecting a hotel + transit/car mode loads the Google Maps embed with the route
6. Switching modes updates the route
7. Closing via ✕, backdrop click, or Escape all work
8. Reopening the modal on a different entry remembers the last hotel selection

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Wire up directions modal: hoist places array and inject links"
```

---

### Task 6: Final verification and deploy

- [ ] **Step 1: Test on mobile viewport**

In browser dev tools, toggle responsive mode to ~375px width.

Expected:
- Modal takes nearly full width with 16px margins on each side
- Hotel dropdown, mode toggle, and map embed all fit within the modal
- Modal is scrollable if content overflows viewport height

- [ ] **Step 2: Test localStorage persistence**

1. Open directions for any restaurant, select "Grand Hyatt Tokyo"
2. Close the modal
3. Open directions for a different restaurant
Expected: "Grand Hyatt Tokyo" is pre-selected in the dropdown

- [ ] **Step 3: Test both modes**

1. Open directions for any restaurant, select a hotel
2. Toggle between Transit and Car
Expected: Iframe reloads with different route each time. Transit shows train/subway route, Car shows driving route.

- [ ] **Step 4: Push to deploy**

```bash
git push origin main
```

Site deploys automatically via GitHub Pages.
