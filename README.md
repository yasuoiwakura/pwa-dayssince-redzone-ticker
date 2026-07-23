# Red Zone Reminder

Track time elapsed since an event. Timer cards shift from green → yellow → red as time passes – so you never miss a deadline.

Built as a serverless PWA. No signup, no backend, no dependencies.

## Quick start

```bash
python -m http.server 8080
# → http://localhost:8080
```

Or deploy to GitHub Pages / Netlify / any static host.

## Features

- **Count-up timer** – days, hours, or minutes since last reset
- **Schedule timer** – multiple daily slots (e.g. pill reminder), 3-line buttons with Soll/Ist/Delta, color zones per slot
- **Geofence checkin** – GPS-based zone entry detection, auto-alarm when entering a location
- **Color zones** – per-timer thresholds: green → yellow → red (schedule: gray → blue → yellow → red → purple → green)
- **Notifications** – push with repeat intervals for schedule slots
- **One-tap reset** – checkmark button resets a timer to now (or acks a geofence/schedule slot)
- **Dark theme** – optimized for dashboards, workshops, bedside
- **Kiosk mode** – keeps screen on, hides status bar (gear icon → Statusleiste → Vollbild)
- **Installable** – "Install as app" for standalone fullscreen mode
- **Offline-ready** – works without internet after first load
- **Import/export** – config via `timers.yaml`, editable in WebUI

## Configuration

Default timers are loaded from `timers.yaml`. Once loaded, they're managed in `localStorage`. Open the settings (gear icon) to add, edit, or remove timers.

```yaml
wake_lock: true           # screen wake lock on/off
status_bar: normal        # normal | blend | hide
gps_interval: 60          # GPS poll interval in seconds

locations:
  - name: Home
    radius: 50
  - name: Office
    lat: 52.42
    lng: 9.60
    radius: 100

timers:
  - name: Example
    unit: hours           # hours | days | minutes
    show_minutes: true    # show sub-unit?
    yellow_after: 8       # yellow after (in timer unit)
    red_after: 12         # red after (in timer unit)

  - name: Geofence Checkin
    type: checkin
    location: Home
    repeat_minutes: 1
    notify: true

  - name: Pille
    type: schedule
    times: ["08:00", "14:00", "20:00"]
    ready_before: 10
    yellow_after: 10
    red_after: 30
    max_overdue: 180
    repeat_minutes: 5
    repeat_max: 6
    notify: true
```

Math expressions like `24*3` are evaluated automatically.

## Systemprüfung

Open Settings → **Systemprüfung** to check which PWA features your device supports. This helps debug platform-specific limitations (Wake Lock, Fullscreen, GPS, etc.).

## Project structure

```
├── index.html      – App (HTML + CSS + JS inline)
├── manifest.json   – PWA manifest (installable)
├── sw.js           – Service worker (cache-first)
├── timers.yaml     – Default timer configuration
├── SPEC.md         – Feature specification & status
└── README.md
```

## Tech stack

- **No dependencies** – plain vanilla JS
- **YAML parser** – minimal, inline, no library required
- **localStorage** – persists timer config and start times
- **Service Worker** – caches `index.html`, `manifest.json`, `timers.yaml`
- **Screen Wake Lock API** – `navigator.wakeLock.request('screen')` + video fallback
- **Fullscreen API** – `requestFullscreen` for kiosk mode
- **Geolocation API** – geofence checkin with adaptive polling

## Planned

- **Webhook event logging** – POST events (reset, ack, schedule_taken) to a configurable webhook URL
  - **Option A: Google Forms (DAU)** – zero backend, events land in a Google Sheet via POST to a form URL
  - **Option B: Python server** – Flask backend with SQLite, bidirectional sync (`last_actions` response)
  - Shared offline queue in localStorage, settings UI for URL + secret + instance ID
  - See [SPEC.md §10](SPEC.md#10-webhook-implementation-event-logging) for full spec
