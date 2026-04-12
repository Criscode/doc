# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Static marketing website for **Criscode** — a mobile app studio. Hosted on OVH shared hosting (PHP 7.0 engine, configured via `.ovhconfig`). No build step, no package manager, no framework — everything is a single `index.html` file served directly.

## Development

Open `index.html` directly in a browser. There is no build, compile, or install step.

For a local dev server with auto-reload you can use any static server, e.g.:

```bash
npx serve .
# or
python3 -m http.server 8080
```

Deployment is a straight file upload to OVH shared hosting.

## Architecture

### Single-file i18n

All English and Polish translations live in a `T` object at the bottom of `index.html` (inside the `<script>` block). DOM elements that need translation carry a `data-i18n="key"` attribute. `applyLang(lang)` walks all `[data-i18n]` elements and sets `textContent` from the matching dict.

Language resolution priority (highest → lowest):
1. `?lang=en` / `?lang=pl` URL parameter — explicit user click, saved to `localStorage`, then stripped from URL via `history.replaceState`
2. `localStorage` key `criscode_lang` — persists across visits
3. `https://ipapi.co/json/` IP geolocation — first-visit only, fires after English renders

When adding new translatable strings: add a `data-i18n="your.key"` attribute to the element, then add `'your.key': '...'` to **both** `T.en` and `T.pl`.

### Screenshot carousel

The SafeDose product section contains a viewpager carousel.

- Screenshots live in `safedose/en/1.png … 6.png` and `safedose/pl/1.png … 6.png`.
- `buildCarousel(lang)` (called from `applyLang`) rebuilds the `<img>` elements pointing at the correct language folder and resets to slide 0.
- Slide width is hardcoded to **200 px** — this must match `.carousel-track img { width: 200px }` and the `goToSlide` transform calculation.
- The Pixel device frame is `pixel-frame.svg` (216×480 viewBox). It overlays the stage via `position: absolute; left: -8px; top: 0` on `.device-frame`, inside `.carousel-frame-wrapper` which has `padding: 18px 0 16px` to accommodate the SVG bezels. The screen cutout in the SVG is `x=8 y=18 w=200 h=446` — derived from the actual screenshot resolution (1280×2856).

### Assets

| File | Purpose |
|---|---|
| `logo.png` | Criscode brand mark (geometric "C") |
| `logo_safe_dose.png` | SafeDose product icon |
| `pixel-frame.svg` | Android Pixel device frame overlay |
| `safedose/{en,pl}/1-6.png` | App screenshots per language |

## Hosting

`.ovhconfig` sets PHP 7.0 engine — only relevant if `.php` files are ever added. The site is fully static today and ignores the PHP engine.
