# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Digital photo frame ("Portarretrato") for an old Android tablet (Galaxy Tab 2): month calendar on the left, rotating photo slideshow plus clock on the right. The whole app is a single static file, [index.html](index.html) (inline CSS + JS). There is no build step, package manager, linter, or test suite; to run it, open the file in a browser (the target device runs it in Chrome / Fully Kiosk Browser). UI strings and code identifiers are in Spanish.

## Constraints that shape all edits

- **ES5 only, no libraries.** The target browser is very old: use `var`, function expressions, no arrow functions/`let`/`const`/template strings/`Promise` (the one `navigator.getBattery().then` is guarded by feature detection). Use `c['continue']()` and `['delete']` bracket forms for reserved-word methods, as the existing code does.
- **CSS needs both prefixed and unprefixed flexbox** (`-webkit-box`, `-webkit-flex`, `flex`), and sizes use `vh`/`vw` so the layout scales to any screen. Portrait layout is overridden in the `@media (orientation: portrait)` block.
- All user-tunable settings live in the `CONFIG` object at the top of the script (photo interval, 12/24h, night-dimming hours, `URL_SCRIPT`/`CLAVE` for Google Calendar, etc.).

## Architecture (inside the single IIFE)

- **Clock loop**: `tic()` runs every second; it repaints the clock, rebuilds the calendar when the date changes (`pintarMes`), and once per minute refreshes night mode (`revisarNoche`, a black overlay at 0.8 opacity) and the "next event" footer (`pintarProximo`). It also reloads the page at 4:00 a.m. daily to free memory.
- **Photo store**: photos picked via the settings panel are downscaled to `CONFIG.TAMANO_MAX` on a canvas, with EXIF orientation applied manually (`orientacionExif` / `reducir`) when the browser doesn't auto-rotate, and saved as JPEG data URLs in IndexedDB (`portarretrato` DB, `fotos` store). The `fotos` array holds descriptors, not image data: `{clave}` for IndexedDB photos, `{src}` for `CONFIG.FOTOS_LOCALES`, and `{drive:true, src, alt}` for Google Drive photos. Image bytes are read lazily in `siguienteFoto`.
- **Slideshow**: `siguienteFoto` draws from a shuffled queue (`cola`, refilled via `barajar`) and crossfades between two stacked layers (`#fotoA` / `#fotoB`) in `mostrar`. Unloadable photos are skipped, with a 60s retry once 5 consecutive attempts fail.
- **Settings panel**: tapping anywhere toggles fullscreen and the bottom panel (add / remove current / delete all photos); it auto-closes after 20s of inactivity (`mantenerPanel`). Removal and deletion only affect IndexedDB photos.
- **Optional Google Calendar/Drive sync**: when `CONFIG.URL_SCRIPT` is set, the page fetches events and Drive photo IDs from an Apps Script web app (`Codigo.gs`, which is *not* in this repo) via JSONP (`jsonp`), sending `CONFIG.CLAVE` as a shared secret. Responses (`{events:[{start,end,title,allDay}], photos:[ids]}`) are cached in localStorage (`pr_datos`) and applied via `aplicarDatos`, which replaces only the Drive entries of `fotos`.
- **Battery indicator**: `iniciarBateria` tries the standard Battery API, then the legacy `navigator.battery`, then Fully Kiosk's `window.fully` JS interface; it stays hidden if none exists.
