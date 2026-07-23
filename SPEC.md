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

## 7. Bekannte Einschränkungen

- **Wake Lock:** `navigator.wakeLock` existiert auf Safari nicht. Video-Fallback ist plattformabhängig und kann vom Browser unterdrückt werden.
- **Fullscreen:** `requestFullscreen()` muss durch User Gesture (Klick) ausgelöst werden – automatisches Verstecken der Statusleiste beim App-Start nicht möglich.
- **Geolocation im Hintergrund:** In einer PWA nicht standardisiert möglich – Geofence-Reminder erfordert offene App mit Wake Lock.
- **Push Notifications:** Ohne Server-Infrastruktur keine Push-Zustellung bei geschlossener App.
- **Systemprüfung:** Prüft nur API-Verfügbarkeit, nicht die tatsächliche Runtime-Funktion (z.B. `wakeLock in navigator` ist `true`, aber `request()` kann trotzdem fehlschlagen).
