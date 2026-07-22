Aufgabe: Erstelle "Red Zone Ticker" PWA,
Erstelle eine serverlose Progressive Web App (PWA) aus 3 Dateien (index.html, manifest.json, sw.js).

Kern-Anforderungen,
Funktionalität: Zeige 3 unabhängige "Time Since"-Timer (aufwärts zählend: Tage:Stunden:Minuten).,
Persistenz: Speichere den Start-Zeitpunkt jedes Timers im localStorage. Beim Neuladen muss die verstrichene Zeit korrekt berechnet werden (Date.now() - startTime).,
Wake Lock: Implementiere die navigator.wakeLock.request('screen') API, damit der Bildschirm im Bad nicht ausgeht. Handle visibilitychange, um den Lock bei Tab-Wechsel neu anzufordern.,
UI/UX:
Dunkles Theme (#121212), riesige rote Zahlen (#ff3333) für Dringlichkeit.,
Jeder Timer hat einen "RESET"-Button (setzt Startzeit auf Date.now()).,
Schriftart: Monospace (tabular-nums) gegen Springen der Ziffern.,
Responsive für Mobile/Tablet (Vollbild).,
,
PWA:
manifest.json: display: standalone, Theme Color Rot.,
sw.js: Cache-First Strategie für Offline-Nutzung.,
,

Datei-Struktur & Logik,
index.html,
,
Enthält HTML, CSS (inline) und JS (inline).,
JS Logik:
Funktion getStartTime(index): Liest aus localStorage oder setzt Date.now().,
Funktion update(): Berechnet diff = Date.now() - start, formatiert zu DD:HH:MM, aktualisiert DOM.,
Event Listener für Reset-Buttons.,
Wake Lock Implementierung (async/await mit try/catch).,
Service Worker Registrierung.,
,

manifest.json,
,
Name: "Red Zone Ticker".,
Start URL: ./index.html.,
Icons: Platzhalter URLs verwenden.,

sw.js,
,
Cache Name: redzone-v1.,
Install: Cache index.html, manifest.json.,
Fetch: Serve from cache, fallback network.,

Ausgabe,
Generiere den vollständigen Code für alle 3 Dateien ohne Erklärtext dazwischen.