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
| **Geolocation (GPS)** | Geofence-Reminder: beim Betreten einer Zone (z.B. Kaufpark) an Parkscheibe erinnern | `navigator.geolocation` + `watchPosition` | ⏳ Geplant |
| **Push Notifications** | Wiederholte Erinnerung (parametrierbar) in der Geofence-Zone | `Notification` + `PushManager` | ⏳ Geplant |

## 4. Nicht geplant

| Feature | Grund |
|---|---|
| Accelerometer / DeviceMotion | Kein Anwendungsfall für diese App |
| Vibration | Kein Anwendungsfall |
| Battery | Kein Anwendungsfall |
| Background Sync / Periodic BG Sync | PWA-Limit: kein Location-Zugriff im Service Worker |
| Push über Server | Würde Server-Infrastruktur erfordern – ist als PWA-Limit akzeptiert |

## 5. Architektur

- **Single-File Vanilla JS** – `index.html` enthält HTML + CSS + JS (kein Framework, kein Bundler)
- **YAML-Parser** – minimaler Inline-Parser für `timers.yaml`
- **localStorage** – Timer-Konfiguration und Startzeiten
- **Service Worker** – Cache-first für `index.html`, `manifest.json`, `timers.yaml`
- **Wake Lock** – primär `navigator.wakeLock.request()`, Fallback via unsichtbares Loop-Video

## 6. Testmatrix (vorläufig)

| Feature | Chrome Win | Chrome Android | Safari iOS | Firefox |
|---|---|---|---|---|
| Timer + Farbzonen | ✅ | ✅ | ✅ | ✅ |
| Settings-UI | ✅ | ✅ | ✅ | ✅ |
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
- **FR: Push erst bei Fahrzeug-Stillstand:** Sinnvolle Idee, aber Implementierung nicht geplant (würde Accelerometer + GPS-Geschwindigkeit erfordern, zu komplex für Phase 1).
- **Service Worker:** `timers.yaml?t=...` wird nicht gecached – OK, da nur bei explizitem "Standard laden" verwendet.

## 8. Bekannte Einschränkungen

- **Wake Lock:** `navigator.wakeLock` existiert auf Safari nicht. Video-Fallback ist plattformabhängig und kann vom Browser unterdrückt werden.
- **Fullscreen:** `requestFullscreen()` muss durch User Gesture (Klick) ausgelöst werden – automatisches Verstecken der Statusleiste beim App-Start nicht möglich.
- **Geolocation im Hintergrund:** In einer PWA nicht standardisiert möglich – Geofence-Reminder erfordert offene App mit Wake Lock.
- **Push Notifications:** Ohne Server-Infrastruktur keine Push-Zustellung bei geschlossener App.
- **Systemprüfung:** Prüft nur API-Verfügbarkeit, nicht die tatsächliche Runtime-Funktion (z.B. `wakeLock in navigator` ist `true`, aber `request()` kann trotzdem fehlschlagen).
- **GPS-Permission:** Auf Android kann nur "nur während der Nutzung" gewählt werden, nicht "immer" – schränkt Geofence-Funktionalität ein.
- **GPS ohne HTTPS:** `navigator.geolocation` erfordert HTTPS – lokale Entwicklung über `http://localhost` ist eine Ausnahme.
