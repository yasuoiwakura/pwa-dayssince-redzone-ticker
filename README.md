# Red Zone Reminder

Serverless PWA to track time elapsed since an event – with color zones (green → yellow → red) based on elapsed time.

## Features

- **Count-up timers** – counts days, hours, or minutes since last reset
- **Color zones** – per-timer thresholds: green (ok) → yellow (warning) → red (critical)
- **Screen Wake Lock** – keeps the display on (dashboards, workshop, bathroom, etc.)
- **Installable** – PWA with `display: standalone` for fullscreen mode
- **Offline-ready** – service worker with cache-first strategy
- **YAML config** – default timers in `timers.yaml`, adjustable via WebUI
- **No backend** – fully client-side, data stored in `localStorage`

## Live

[https://yasuoiwakura.github.io/pwa-dayssince-redzone-ticker/](https://yasuoiwakura.github.io/pwa-dayssince-redzone-ticker/)

## Quick start

```bash
python -m http.server 8080
# → http://localhost:8080
```

Or deploy to GitHub Pages / Netlify / any static host.

## Project structure

```
├── index.html      – App (HTML + CSS + JS inline)
├── manifest.json   – PWA manifest (installable)
├── sw.js           – Service worker (cache-first)
├── timers.yaml     – Default timer configuration
└── README.md
```

## Configuration

Default timers are loaded from `timers.yaml`. Once loaded, they are managed in `localStorage`. Use the gear icon (⚙) to add, edit, or remove timers.

```yaml
timers:
  - name: Example
    unit: hours           # hours | days | minutes
    show_minutes: true    # show sub-unit?
    yellow_after: 8       # yellow after (in timer unit)
    red_after: 12         # red after (in timer unit)
```

Math expressions like `24*3` are evaluated automatically.

## Tech stack

- **No dependencies** – plain vanilla JS
- **YAML parser** – minimal, inline, no library required
- **localStorage** – persists timer config and start times
- **Service Worker** – caches `index.html`, `manifest.json`, `timers.yaml`
- **Screen Wake Lock API** – `navigator.wakeLock.request('screen')`
- **Dark theme** – `#121212` background, monospace with `tabular-nums`
