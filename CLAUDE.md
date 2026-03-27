# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-page HTML restaurant/bar/coffee guide for Tokyo — 23 curated places with editorial vignettes, Google Maps links, Tabelog links, and an embedded interactive Google Map.

## Tech Stack

- Single static `index.html`, no build step, no dependencies
- Google Fonts (Cormorant Garamond + DM Sans)
- Google Maps JavaScript API for the embedded map with colored markers
- Deployed via GitHub Pages

## Development

No build or install commands. Just open `index.html` in a browser or serve it locally:

```
python3 -m http.server
```

Note: The embedded Google Map requires the page to be served (not opened as a `file://` URL) for the API key to work.

## Architecture

Everything lives in `index.html` — HTML, CSS, and JS are all inline. There is no separation into multiple files.

### Map

- 23 markers with 3 color categories: restaurants (terracotta `#b8432f`), bars (purple `#6b5b95`), coffee (brown `#5c3d2e`)
- Custom map styling matching the warm paper palette (`#f5f2ed`)
- Clickable markers with InfoWindows; bottom-center legend

### Sections (sticky nav)

1. **Restaurants** (17 entries) — izakaya, kaiseki, pizza, yakiniku, yakitori, sushi, fusion, Italian, oyakodon
2. **Bar** (3 entries) — cocktail bars
3. **Coffee** (3 entries) — specialty coffee

### Design

- Warm editorial aesthetic: paper-toned background, Cormorant Garamond headings, DM Sans body
- Each entry: name, cuisine/type tag, neighborhood tag, Maps link (place IDs), Tabelog link (where available), vignette, price
- Color-coded tags: restaurants in terracotta, coffee in brown
- Mobile responsive

## Files

- `index.html` — the complete page, ready to serve
- `tokyo-restaurant-guide.md` — markdown text content (slightly outdated, missing some entries)

## Known Issues

- The Google Maps API key is a demo key. For production, replace it and restrict by HTTP referrer to the GitHub Pages domain.
- Savoy's Tabelog link points to an old listing; the Google Maps link is correct.
- Some entries (Glitch Coffee, three bars) lack Tabelog links.
