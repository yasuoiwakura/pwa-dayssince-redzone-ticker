# Red Zone Reminder – Spezifikation

## 1. App-Features

| Feature | Beschreibung | Status |
|---|---|---|
| Count-up Timer | Zeit seit letztem Reset zählen | ✅ Implementiert |
| Einheiten | Tage / Stunden / Minuten | ✅ Implementiert |
| Sub-Unit | Kleinere Einheit anzeigen (z.B. h bei Tagen) | ✅ Implementiert |
| Farbzonen | Grün → Gelb → Rot pro Timer konfigurierbar | ✅ Implementiert |
| YAML-Konfiguration | Standard-Timer aus `timers.yaml` laden | ✅ Implementiert |
| Settings-UI | Timer anlegen/bearbeiten/löschen | ✅ Implementiert |
| Dark Theme | #121212 Hintergrund, monospace | ✅ Implementiert |
| Geofence (GPS) | Checkin-Timer mit Zonen-Eintritt via GPS | ✅ Implementiert |
| Adaptives GPS-Polling | Dynamisches Intervall (3s – 60s) basierend auf Distanz | ✅ Implementiert |
| Schedule Timer (Pillen) | Mehrere tägliche Zeiten, Count-up ab Fälligkeit | ⏳ Geplant |
| Notification-Typen | Push / Audio / Vibration / Stumm, pro Timer überschreibbar | ⏳ Geplant |

## 2. PWA-Features

| Feature | Spezifikation | Plattform | Laufzeit | PWA |
|---|---|---|---|---|
| **Screen Wake Lock** | Bildschirm anhalten (Kiosk/Dashboard) | ⚠️ `navigator.wakeLock` nur Chromium | ⚠️ `request()` scheitert ohne User Gesture, nicht auf iOS | ✅ Video-Fallback |
| **Fullscreen (Kiosk)** | Statusleiste ausblenden per `requestFullscreen` | ✅ `document.fullscreenEnabled` | ⚠️ `requestFullscreen()` nur bei User Gesture | ✅ Über Statusbar-Modus "hide" |
| **Service Worker** | Offline-Caching (Cache-first) | ✅ `serviceWorker` | ✅ Registriert, Cache v2 aktiv | ✅ Aktiv |
| **Installierbar (A2HS)** | `display: standalone`, `beforeinstallprompt` | ⚠️ Nicht Firefox/Safari | ✅ über `beforeinstallprompt` | ✅ Install-Mode auto/show/hide |
| **localStorage** | Persistenz ohne Backend | ✅ | ✅ | ✅ Wird genutzt |
| **Cache API** | SW-Cache für Offline-Fallback | ✅ | ✅ | ✅ Aktiv |
| **Systemprüfung** | Debug-Dialog "Systemprüfung" mit Plattform-Checks | ✅ Alle API-Checks | ⚠️ Prüft nur API-Existenz, nicht Runtime | ✅ Implementiert |

## 3. Geplante Features

| Feature | Spezifikation | API | Status |
|---|---|---|---|
| **Schedule Timer** | Mehrere tägliche Zeiten (z.B. Pille 8:00/14:00/20:00), Count-up ab verpasster Zeit, Farbzonen | `setInterval` + `tick()` | ⏳ Geplant |
| **Notification-Type: Audio** | Eigener Ton über `OscillatorNode` bei Alarm (Media-Lautstärke) | `AudioContext` | ⏳ Geplant |
| **Notification-Type: Vibration** | Haptisches Feedback via `vibrate`-Pattern | `navigator.vibrate` | ⏳ Geplant |
| **Default Notification-Typ** | Settings-Dropdown: Push / Audio / Vibration / Stumm | `localStorage` | ⏳ Geplant |
| **Per-Timer Notification-Override** | YAML-Feld `notification_type` überschreibt globalen Default | YAML + `localStorage` | ⏳ Geplant |
| **Bald-Farbe (Blau)** | Optionale Vorwarnung N Minuten vor Fälligkeit in Blau | `tick()` | 💡 Idee |

## 4. Nicht geplant

| Feature | Grund |
|---|---|
| Accelerometer / DeviceMotion | Kein Anwendungsfall für diese App |
| Battery | Kein Anwendungsfall |
| Background Sync / Periodic BG Sync | PWA-Limit: kein Location-Zugriff im Service Worker |
| Push über Server | Würde Server-Infrastruktur erfordern – ist als PWA-Limit akzeptiert |

## 5. Architektur

- **Single-File Vanilla JS** – `index.html` enthält HTML + CSS + JS (kein Framework, kein Bundler)
- **YAML-Parser** – minimaler Inline-Parser für `timers.yaml`
- **localStorage** – Timer-Konfiguration, Startzeiten, Schedule-Taken-States, Notification-Einstellungen
- **Service Worker** – Cache-first für `index.html`, `manifest.json`, `timers.yaml`
- **Wake Lock** – primär `navigator.wakeLock.request()`, Fallback via unsichtbares Loop-Video
- **Notification-Typen** – `reg.showNotification()` (Push), `AudioContext` (Audio), `navigator.vibrate()` (Vibration), Default + Per-Timer-Override

## 6. Testmatrix (vorläufig)

| Feature | Chrome Win | Chrome Android | Safari iOS | Firefox |
|---|---|---|---|---|---|
| Timer + Farbzonen | ✅ | ✅ | ✅ | ✅ |
| Geofence (Checkin) | ✅ | ✅ | ✅ | ✅ |
| GPS Auto-Polling | ✅ | ✅ | ✅ | ✅ |
| Settings-UI | ✅ | ✅ | ✅ | ✅ |
| Schedule Timer (Pillen) | ⏳ | ⏳ | ⏳ | ⏳ |
| Notification: Push | ✅ | ✅ | ✅ | ✅ |
| Notification: Audio | ⏳ | ⏳ | ⏳ | ⏳ |
| Notification: Vibration | ⏳ | ⏳ | ❌ | ⏳ |
| Wake Lock (API) | ✅ | ⚠️ | ❌ | ⚠️ |
| Wake Lock (Video) | ⚠️ | ⚠️ | ⚠️ | ⚠️ |
| Fullscreen | ✅ | ✅ | ❌ | ✅ |
| Service Worker | ✅ | ✅ | ✅ | ✅ |
| Installierbar | ✅ | ✅ | ❌ | ❌ |
| Systemprüfung | ✅ | ✅ | ✅ | ✅ |
| Geolocation (GPS) | ⏳ | ⏳ | ⏳ | ⏳ |
| Push Notifications | ⏳ | ⏳ | ⏳ | ⏳ |

## 7. Bugs & TODOs

### 7.1 GPS: Kein User-Feedback bei fehlender Berechtigung

- GPS ist standardmäßig nicht aktiv – es gibt aber **keine Info an den User**.
- Die Systemprüfung zeigt Geolocation als "✅ Verfügbar" an, obwohl die **Runtime-Berechtigung fehlt**.
- Der OS-Permission-Dialog erscheint erst beim **Klick auf das kleine 📍-Icon unten** – kein Hinweis darauf.
- Auf Android kann man nur **"diesmal" oder "wenn App im Vordergrund"** wählen, nicht "immer".
- **TODO:** Systemtest muss Runtime-Permission prüfen (nicht nur API-Existenz). User-freundlicher Permission-Hinweis in der UI.

### 7.2 Geofence-Timer: Statusanzeige unvollständig

Der Checkin-Timer muss folgende Zustände abbilden:

| Zustand | Icon/Farbe | Beschreibung |
|---|---|---|
| Kein GPS verfügbar | 🔴 Rot | `navigator.geolocation` nicht vorhanden |
| Keine Erlaubnis | 🔒🔴 Rot | Permission denied |
| Außerhalb Geofence | ⚫ Grau | GPS aktiv, Distanz > Radius |
| Im Geofence (unbestätigt) | 🟡 Gelb (blinkend) | Alarm aktiv |
| Im Geofence + bestätigt | 🟢 Grün | `acked` gesetzt |
| Nach Verlassen | ⚫ Grau | Zurückgesetzt |

**TODO:** `updateGeoUI()` und `tick()` müssen diese Zustände abbilden.

### 7.3 Wake Lock (Video-Fallback) defekt

- Video-Workaround funktioniert **nicht im Browser (Edge/Windows)**.
- Funktioniert **nicht auf iPhone 7** (iOS Safari).
- **TODO:** Fallback-Mechanismus überprüfen oder alternative Strategie (kein Video) implementieren.

### 7.4 Keine echte Push-Benachrichtigung

- Auch wenn die UI den Geofence-Alarm anzeigt (roter Banner), gibt es **keine System-Notification**.
- Der `new Notification()`-Aufruf wird durch `Notification.permission` blockiert (default = `default`).
- **Keine Permission-Abfrage** für Notification wurde je durchgeführt.
- **TODO:** Permission-Abfrage für Notification beim ersten Geofence-Alarm + Permissions-Menü.

### 7.5 GPS-Position per "Hier" setzen

- **FR:** Für lokale Tests soll man in den Settings die aktuelle GPS-Position per Knopfdruck übernehmen können (ohne Koordinaten manuell einzugeben).
- Verhindert, dass private Standorte (z.B. Zuhause) in Git landen.
- **TODO:** Button "Hier" in der Location-Edit-Maske – ruft `getCurrentPosition()` ab und befüllt Lat/Lng-Felder.

### 7.6 Permissions-Menü fehlt

- Es gibt keinen zentralen Ort, um Zugriffsrechte anzufordern: GPS, Push Notifications, ggf. Wake Lock.
- **TODO:** Settings um "Berechtigungen"-Abschnitt erweitern, der fehlende Berechtigungen anzeigt und per Button anfordert.

### 7.7 Sonstige TODOs

- **"Standard laden" Cache-Busting:** ✅ Implementiert seit 2026-07-23 (`?t=Date.now()`).
- **Service Worker:** `timers.yaml?t=...` wird nicht gecached – OK, da nur bei explizitem "Standard laden" verwendet.

### 7.8 FR: Schedule Timer (Pillen-Reminder)

Ein Timer mit mehreren täglichen Zeiten (`times: ["08:00", "14:00", "20:00"]`), angezeigt als ein Eintrag:

| Zustand | Value | Farbe | Beschreibung |
|---|---|---|---|
| Nächste Zeit > jetzt | `→ 14:00` | Grau `#888` | Noch nicht fällig |
| Fällig, nicht genommen | Count-up | Gelb → Rot | Minuten seit verpasster Zeit |
| Genommen | `✔ 08:00` | Grün `#33ff33` | Für diesen Slot quittiert |

- **Storage:** `redzone_schedule_taken` – pro Timer-Index + Datum → Liste der genommenen Uhrzeiten
- **✓-Button** markiert den aktuell fälligen Slot als genommen
- **Notify** beim Überschreiten einer `times[]`-Uhrzeit
- **Optional später:** Blaue Vorwarnung N Minuten vor Fälligkeit

### 7.9 FR: Notification-Type konfigurierbar

**Globaler Default** in Settings (Dropdown):

| Typ | API | Anmerkung |
|---|---|---|
| `push` | `reg.showNotification()` | Aktuelles Verhalten, Chrome hängt URL an |
| `audio` | `OscillatorNode` (AudioContext) | Kurzer Sinus-Ton, läuft über Media-Lautstärke |
| `vibration` | `navigator.vibrate([200,100,200])` | Android-only, nicht auf iOS |
| `stumm` | – | Nur In-App-Banner |

**Per-Timer-Override** via YAML:

```yaml
timers:
  - name: Pille
    notification_type: audio
```

- **Storage Default:** `redzone_notification_default` in `localStorage`
- **Auflösung:** `t.notification_type ?? localStorage.default ?? 'push'`

### 7.10 FR: Systemtest Runtime-Prüfung

Systemtest prüft aktuell nur API-Existenz, nicht Runtime:
- `geolocation` → `navigator.permissions.query({ name: 'geolocation' })` für tatsächlichen Permission-Status
- `wakeLock` → `request()` testen ob's wirklich funktioniert
- `Notification` → `Notification.permission` anzeigen

### 7.11 FR: Notification requireInteraction + Vibrate

- `requireInteraction: true` – Notification bleibt stehen statt nach 8s zu verschwinden
- `vibrate`-Pattern in der Notification-Option (nur Android)

```javascript
reg.showNotification(title, {
  body: msg, icon: 'icon.svg',
  requireInteraction: true,
  vibrate: [200, 100, 200]
});
```

## 8. Bekannte Einschränkungen

- **Wake Lock:** `navigator.wakeLock` existiert auf Safari nicht. Video-Fallback ist plattformabhängig und kann vom Browser unterdrückt werden.
- **Fullscreen:** `requestFullscreen()` muss durch User Gesture (Klick) ausgelöst werden – automatisches Verstecken der Statusleiste beim App-Start nicht möglich.
- **Geolocation im Hintergrund:** In einer PWA nicht standardisiert möglich – Geofence-Reminder erfordert offene App mit Wake Lock.
- **Push Notifications:** Ohne Server-Infrastruktur keine Push-Zustellung bei geschlossener App.
- **Systemprüfung:** Prüft nur API-Verfügbarkeit, nicht die tatsächliche Runtime-Funktion (z.B. `wakeLock in navigator` ist `true`, aber `request()` kann trotzdem fehlschlagen).
- **GPS-Permission:** Auf Android kann nur "nur während der Nutzung" gewählt werden, nicht "immer" – schränkt Geofence-Funktionalität ein.
- **GPS ohne HTTPS:** `navigator.geolocation` erfordert HTTPS – lokale Entwicklung über `http://localhost` ist eine Ausnahme.

## 9. FR: Webhook-Integration (Event-Logging & Sync)

### 9.1 Ziel

Jede Useraktion, die einen Task erledigt (Reset, Ack, Schedule-Taken), ruft einen konfigurierbaren Webhook auf. Der Server quittiert mit den letzten bekannten Aktionen pro Timer – so baut sich eine geräteunabhängige Datenbasis für Auswertungen (z.B. Abweichungen Soll/Ist) auf.

### 9.2 Payload (POST)

```json
{
  "app": "redzone-reminder",
  "instance": "<instance-id aus localStorage>",
  "secret": "<hash aus localStorage>",
  "action": "reset" | "ack" | "schedule_taken",
  "timer": "<timer-name>",
  "timestamp": "<ISO-8601>",
  "details": {
    "scheduled_time": "08:00",
    "actual_time": "08:05"
  }
}
```

Bei `schedule_taken` zusätzlich `scheduled_time` (Soll) und `actual_time` (Ist) für Abweichungsanalyse.

### 9.3 Server-Response

Der Server sendet bei jeder Anfrage (auch bei leeren) die letzten bekannten Aktionen jedes Timers zurück – z.B. als Sync für andere Instanzen:

```json
{
  "status": "ok",
  "last_actions": {
    "Parkscheibe": { "action": "ack", "timestamp": "..." },
    "Pille": { "action": "schedule_taken", "timestamp": "...", "scheduled_time": "08:00" }
  }
}
```

### 9.4 Offline-Queue (optional, später)

Wenn `fetch()` fehlschlägt, wird die Action in `localStorage` (`redzone_webhook_queue`) zwischengespeichert und beim nächsten erfolgreichen Webhook-Aufruf nachgesendet.

### 9.5 Konfiguration

**YAML (App-Ebene + Timer-Ebene):**

```yaml
webhook:
  url: "https://homelab.local/webhook"
  # secret wird NICHT in YAML gespeichert

timers:
  - name: Pille
    webhook_url: "https://homelab.local/webhook/pille"  # override
```

**Secret** – liegt in `localStorage` (`redzone_webhook_secret` + `redzone_webhook_instance_id`), wird im Settings-Menü gesetzt. So bleibt es aus dem YAML (und damit aus Git/öffentlicher Instanz) raus.

**Auflösung:** `t.webhook_url ?? config.webhook.url ?? null` → wenn `null`, kein Webhook für diesen Timer.

**Settings-UI:** Textfeld für Webhook-URL (global) + Secret + Instance-ID, darunter Liste der Timer-spezifischen URLs.

### 9.6 Offene Fragen

- **Öffentliche Instanz:** YAML inkl. Webhook-URL wäre öffentlich einsehbar – daher muss URL entweder ebenfalls in localStorage (Settings-UI) oder über eine Umgebungsvariable gelöst werden.
- **Datenschutz:** Aktionen + Zeitstempel sind ggf. personenbezogen – DSGVO-konforme Lösung nötig falls öffentlich betrieben.
- **Backend:** Einfaches Python-Skript im HomeLab vs. InfluxDB für Zeitreihen-Auswertung. Webhook-Ansatz ist agnostisch und erlaubt beides.
- **Secret-Hash:** Client-seitig gehashtes Secret (SHA-256) im Authorization-Header, Server vergleicht gegen konfigurierten Hash – kein Klartext-Secret über die Leitung.
