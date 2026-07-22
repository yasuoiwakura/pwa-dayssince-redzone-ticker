# Red Zone Reminder

Serverlose PWA zum Tracken der Zeit seit einem Ereignis – mit farblichen Zonen (grün → gelb → rot) je nach verstrichener Zeit.

## Features

- **Countup-Timer** – zählt Tage, Stunden oder Minuten seit dem letzten Reset
- **Farbzonen** – individuell pro Timer konfigurierbar: Grün (ok) → Gelb (Achtung) → Rot (kritisch)
- **Screen Wake Lock** – Bildschirm bleibt eingeschaltet (für Dashboards, Werkstatt, Bad etc.)
- **Installierbar** – als PWA mit `display: standalone` für Vollbild-Modus
- **Offline-fähig** – Service Worker mit Cache-First-Strategie
- **Konfiguration via YAML** – Timer-Standards in `timers.yaml`, überschreibbar über das WebUI
- **Kein Backend** – läuft komplett clientseitig, Speicherung in `localStorage`

## Schnellstart

```bash
python -m http.server 8080
# → http://localhost:8080
```

Oder einfach auf GitHub Pages / Netlify / jeden Static-Hoster deployen.

## Projektstruktur

```
├── index.html      – App (HTML + CSS + JS inline)
├── manifest.json   – PWA-Manifest (installierbar)
├── sw.js           – Service Worker (Cache-First)
├── timers.yaml     – Standard-Timer-Konfiguration
└── README.md
```

## Konfiguration

Die Standard-Timer werden aus `timers.yaml` geladen. Einmal geladen, werden sie in `localStorage` gemanagt. Über das Zahnrad-UI (⚙) können Timer hinzugefügt, bearbeitet und entfernt werden.

```yaml
timers:
  - name: Beispiel
    unit: hours           # hours | days | minutes
    show_minutes: true    # kleinere Einheit anzeigen?
    yellow_after: 8       # Gelb ab (in der Einheit)
    red_after: 12         # Rot ab (in der Einheit)
```

Rechenausdrücke wie `24*3` werden automatisch ausgewertet.

## Technik

- **Keine Abhängigkeiten** – reines Vanilla JS
- **YAML-Parser** – minimalistisch, inline, keine Library nötig
- **localStorage** – persistiert Timer-Konfiguration und Startzeiten
- **Service Worker** – cached `index.html`, `manifest.json` und `timers.yaml`
- **Screen Wake Lock API** – `navigator.wakeLock.request('screen')`
- **Dark Theme** – `#121212` Hintergrund, Monospace-Schrift mit `tabular-nums`
