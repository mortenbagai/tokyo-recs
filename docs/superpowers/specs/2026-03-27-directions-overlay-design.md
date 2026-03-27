# Directions Overlay Design

## Overview

Add a "Directions" button to every restaurant, bar, wine, and coffee entry that opens a modal overlay. The user selects which hotel they're staying at, chooses transit or car mode, and sees a Google Maps embedded route with full directions, estimated time, and transfers.

## Scope

- Applies to: restaurant, bar, wine, and coffee entries (categories `r`, `b`, `w`, `c`)
- Does NOT apply to: hotel entries (they are the origin, not destination)
- All changes inline in `index.html` — no new files or build steps

## API Key

- Replace the existing demo key (`AIzaSyDoOngUdwQvcia2lExJYWqGh6ezRi9KhPk`) with the billing-enabled key (`AIzaSyBDS_uyZVxZK8wxFNESVP3RITS3UAgoLcs`) in both:
  - The Maps JavaScript API `<script>` tag
  - The Embed API iframe `src` URLs (constructed in JS)
- **Prerequisite:** Enable the "Maps Embed API" in the Google Cloud Console: https://console.cloud.google.com/apis/library/maps-embed-backend.googleapis.com

## UI Components

### 1. Directions Button

- A new `<a class="map-link">` with text "↗ Directions" added to each non-hotel entry's `entry-meta` div
- Positioned after the existing Map/Tabelog links
- On click: opens the directions modal, passing the entry's slug to identify the destination

### 2. Modal Overlay

One shared modal in the DOM (not one per entry). Structure:

```
+--------------------------------------+
| Directions to [Restaurant Name]    ✕ |
+--------------------------------------+
| From:                                |
| [  Select your hotel...         v ]  |
+--------------------------------------+
| [ 🚇 Transit ]  [ 🚗 Car ]          |
+--------------------------------------+
|                                      |
|     Google Maps Embed iframe         |
|     (route with directions)          |
|                                      |
+--------------------------------------+
```

**Header:** Restaurant/bar name + close button (✕). Clicking backdrop also closes.

**Hotel selector:** `<select>` dropdown populated from the `places` array filtered to `cat === "h"`. Sorted alphabetically. Last selection persisted in `localStorage` under key `directions-hotel` so the user only picks once per browser.

**Mode toggle:** Two buttons, "Transit" (default) and "Car". Active state uses the `--hotel` teal color (`#2a7d6e`). Selecting a mode immediately updates the iframe.

**Map embed:** `<iframe>` using the Google Maps Embed API directions mode:
```
https://www.google.com/maps/embed/v1/directions
  ?key=API_KEY
  &origin={hotelLat},{hotelLng}
  &destination={destLat},{destLng}
  &mode=transit|driving
```

The iframe `src` is set only when both a hotel and mode are selected, to avoid unnecessary loads. Default iframe height: 350px. Width: 100% of modal.

### 3. Modal Behavior

- **Open:** Triggered by clicking "↗ Directions" on any non-hotel entry. JS reads the entry's `id` attribute (slug), looks up the matching place in the `places` array to get destination name and coordinates.
- **Close:** Click ✕ button, click backdrop, or press Escape.
- **State:** Hotel selection persisted in `localStorage`. Mode defaults to "transit" on each open.
- **Body scroll lock:** `document.body.style.overflow = 'hidden'` while modal is open.

## Styling

- Modal max-width: 440px, centered vertically and horizontally
- Backdrop: semi-transparent black (`rgba(0,0,0,0.4)`)
- Border-radius: 12px on the modal
- Font families match the page: Cormorant Garamond for the title, DM Sans for controls
- Responsive: on viewports < 500px, modal goes to `calc(100vw - 32px)` width

## Data Flow

1. User clicks "↗ Directions" on an entry
2. JS finds the destination in `places` by slug match
3. Modal opens with destination name in header
4. If `localStorage` has a saved hotel, pre-select it in the dropdown
5. With hotel selected + mode set → construct Embed URL → set iframe `src`
6. User changes hotel or mode → iframe `src` updates immediately

## Edge Cases

- If no hotel is selected yet: show a placeholder message in the iframe area ("Select your hotel above to see directions")
- The `places` array already contains lat/lng for all entries, so no geocoding is needed
