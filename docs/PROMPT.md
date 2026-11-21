# Prompt für Claude Code: Lauf-App „Laufbuch" für iPhone

> **So nutzt du diesen Prompt:** Lege auf deinem Mac einen leeren Ordner an (z. B. `~/Projekte/Laufbuch`), öffne darin das Terminal, starte `claude` und füge alles ab der Linie unten ein. Voraussetzung: ein Mac mit aktuellem Xcode und dein iPhone 11 per Kabel verbunden.

---

## Rolle und Ziel

Du bist ein erfahrener iOS-Entwickler (Swift, SwiftUI, CoreLocation, HealthKit, MapKit). Baue mit mir eine native iPhone-App namens **„Laufbuch"**, mit der ich meine täglichen Läufe aufzeichne. Sie soll mindestens so gut funktionieren wie adidas Running oder Strava, in einigen Punkten besser: genauere Distanz, klare deutsche Sprachansagen und eine aussagekräftige Auswertung. Ich bereite mich auf einen Marathon vor (Berlin 2027) und laufe fast täglich 5–10 km in Berlin-Mitte, mit vielen Ampelstopps.

Die App ist für meinen persönlichen Gebrauch auf einem **iPhone 11**. Es gibt keinen App-Store-Release, kein Backend, keine Accounts und keine Werbung. Alle Daten bleiben auf dem Gerät.

## Arbeitsweise

1. **Plane zuerst.** Lege mir vor dem Coden einen kurzen Plan vor: Architektur, Dateistruktur, Phasen. Warte dann auf mein OK.
2. **Arbeite in Phasen** (siehe unten). Am Ende jeder Phase soll die App bauen und lauffähig sein. Sag mir dann, was ich auf dem iPhone testen soll.
3. **Lege eine `CLAUDE.md`** im Projekt an. Sie enthält Ziel, Architektur, Konventionen, Build-Befehle und den Stand der Phasen.
4. **Baue und teste selbst** mit `xcodebuild`, im Simulator und mit Unit-Tests. Behebe Build-Fehler, bevor du mir eine Phase als fertig meldest.
5. **Frag nach**, wenn etwas unklar ist, statt zu raten. Das gilt vor allem für Xcode-Signing und Capabilities, die ich in Xcode selbst einstellen muss. Erkläre mir solche Schritte Klick für Klick, ich bin kein iOS-Profi.
6. **Commits:** Lege ein Git-Repo an und committe nach jeder abgeschlossenen Teilaufgabe mit aussagekräftiger Nachricht.

## Technische Vorgaben

- **Sprache und UI:** Swift 5.9+ und SwiftUI, Mindestversion **iOS 17**. Das iPhone 11 bekommt aktuelle iOS-Versionen, prüfe die installierte Version und richte das Deployment-Target danach aus.
- **Architektur:** MVVM mit klar getrennten Services (`LocationService`, `WorkoutEngine`, `VoiceCoach`, `HealthKitService`, `PersistenceService`). Die Rechenlogik muss ohne UI und ohne GPS testbar sein.
- **Speicherung:** SwiftData lokal auf dem Gerät.
- **Keine externen Abhängigkeiten,** nur Apple-Frameworks. Ausnahme nur nach Rückfrage.
- **Einheiten:** Kilometer, Pace in min/km, Zeiten als h:mm:ss, deutsches Datumsformat. Die gesamte UI ist auf Deutsch.
- **Akku:** Die App muss einen Lauf von 4–5 Stunden (Marathon) durchhalten. Achte auf sinnvolle GPS-Einstellungen und vermeide unnötige UI-Updates bei gesperrtem Bildschirm.

## Kernfunktionen (Pflicht)

### 1. Laufaufzeichnung
- Großer Start-Button. Vor dem Start zeigt die App die GPS-Signalqualität an (schwach/gut/sehr gut) und wartet optional auf gutes Signal.
- 3-Sekunden-Countdown mit Ansage. Danach Pause, Fortsetzen und Beenden. Beenden erst nach langem Drücken, damit ich nicht versehentlich stoppe.
- **Hintergrund-Tracking:** Die Aufzeichnung läuft bei gesperrtem Bildschirm und mit anderen Apps im Vordergrund weiter (Background Mode „Location updates", `CLLocationManager` mit `activityType = .fitness`, `allowsBackgroundLocationUpdates = true`, `pausesLocationUpdatesAutomatically = false`).
- **Live-Anzeige:** Distanz, Laufzeit, aktuelle Pace (geglättet, nicht springend), Ø-Pace, Pace des aktuellen Kilometers, Höhenmeter und eine Karte mit der bisherigen Strecke.
- **Sperrbildschirm:** eine Live Activity bzw. Dynamic Island mit Zeit, Distanz und Pace, falls technisch machbar. Sonst mindestens eine gut lesbare Anzeige beim Entsperren.

### 2. Genaue Distanz und Pace (hier besser sein als die anderen)
- **GPS-Filter:** Punkte mit `horizontalAccuracy` > 20 m verwerfen, ebenso unrealistische Sprünge (Geschwindigkeit > 7 m/s). Einen einfachen Kalman-Filter oder eine gleitende Glättung für die Pace einbauen.
- **Auto-Pause** (in den Einstellungen abschaltbar): Bei Stillstand, etwa an Ampeln, pausiert die Zeit automatisch und läuft beim Weiterlaufen von selbst weiter. Dazu eine kurze Ansage „Pausiert" / „Weiter". Gesamtzeit und Bewegungszeit werden getrennt gespeichert.
- **Pace pro Kilometer (Splits):** exakt beim Überschreiten jedes vollen Kilometers, mit Interpolation zwischen zwei GPS-Punkten statt am nächsten Punkt. Der letzte angebrochene Kilometer wird anteilig berechnet.
- Höhenmeter mit Barometer (`CMAltimeter`) statt nur GPS-Höhe, das ist deutlich genauer.
- Kadenz (Schritte pro Minute) über `CMPedometer`.
- **Unit-Tests** für Distanzberechnung, Split-Interpolation, Auto-Pause-Logik und Pace-Glättung, mit simulierten GPS-Tracks inklusive Ausreißern und Ampelstopps.

### 3. Sprachansagen (Voice Coach)
- Deutsche Stimme (`AVSpeechSynthesizer`, beste verfügbare Stimme für `de-DE`, wählbar in den Einstellungen). Sprechtempo und Lautstärke sind einstellbar.
- **Nach jedem Kilometer** kommt eine Ansage, zum Beispiel: *„3 Kilometer. Zeit: 19 Minuten 40. Letzter Kilometer: 6 Minuten 25. Durchschnitt: 6 Minuten 33 pro Kilometer."* Die Bausteine sind einzeln abschaltbar.
- Optional zusätzlich zeitbasierte Ansagen, etwa alle 5 Minuten.
- **Musik und Podcasts** werden während der Ansage leiser und danach wieder lauter (`AVAudioSession` mit `.playback`, Modus `.voicePrompt`, Option `.duckOthers`). Sie werden nicht gestoppt.
- Ansagen funktionieren bei gesperrtem Bildschirm und über Bluetooth-Kopfhörer.
- Optional gibt es Pace-Warnungen bei einer Zielpace: *„Du bist 15 Sekunden zu schnell."*
- **Zusammenfassung am Ende**, gesprochen und als Bildschirm: Gesamtdistanz, Zeit, Ø-Pace, schnellster und langsamster Kilometer, Höhenmeter und neue Bestleistungen.

### 4. Auswertung nach dem Lauf
- Karte der Strecke, **farbig nach Pace** (grün = schnell, rot = langsam), mit Kilometer-Markierungen.
- Tabelle der Splits: km, Pace, Höhenmeter, Kadenz, mit Balkendiagramm (Swift Charts).
- Diagramme für Pace-Verlauf und Höhenprofil über die Distanz.
- Kennzahlen: Distanz, Bewegungszeit, Gesamtzeit, Ø-Pace, beste Pace, Kalorien (Schätzung, klar so gekennzeichnet), Kadenz.
- Notiz und Gefühl (1–5 Sterne oder Emoji) zum Lauf speichern, Lauf umbenennen und löschen.

### 5. Verlauf, Statistik und Bestleistungen
- Liste aller Läufe, gruppiert nach Woche, mit Wochensumme in km.
- **Statistik:** km pro Woche und Monat als Diagramm, Anzahl Läufe, Ø-Pace-Trend, längster Lauf, Laufserie (Tage in Folge).
- **Bestleistungen:** schnellster 1 km, 5 km, 10 km, Halbmarathon und Marathon, jeweils auch innerhalb längerer Läufe gefunden. Neue Bestleistungen werden gefeiert.
- **10-%-Hinweis:** Die App warnt, wenn die Wochen-km mehr als ca. 10 % über der Vorwoche liegen.
- Ein **Wochenziel** in km mit Fortschrittsring.

### 6. Export und Anbindung
- **Apple Health:** Jeder Lauf wird als Workout mit Route gespeichert (`HKWorkoutBuilder`, `HKWorkoutRouteBuilder`). Damit erscheinen die Läufe auch in Apple Fitness.
- **GPX-Export** pro Lauf über das Teilen-Menü. So kann ich Läufe bei Strava hochladen.
- **CSV-Export** aller Läufe (Datum, Start, km, Zeit, Ø-Pace, Höhenmeter, schnellster km, schnellste 5 km, Notiz).

## Erweiterungen (nach den Pflichtfunktionen, jeweils nach Rückfrage)

1. **Herzfrequenz über Bluetooth-Brustgurt** (Standard-Heart-Rate-Service 0x180D über CoreBluetooth), mit Pulszonen und Zonenansage.
2. **Intervall- und Tempotraining:** Workouts selbst anlegen (z. B. 5 × 1 km in 5:45 mit 2 min Trabpause), inklusive Sprachführung.
3. **Marathon-Trainingsplan:** ein Plan bis zum Berlin-Marathon (Ende September 2027) mit Wochenumfang, langem Lauf und Ruhetagen, angepasst an meine tatsächlichen Läufe.
4. **Strava-Upload** direkt aus der App über die Strava-API mit OAuth. Ich lege dafür selbst eine API-App bei Strava an, erkläre mir die Schritte. Den Client-Secret niemals im Code hartkodieren, sondern im Schlüsselbund speichern.
5. **Apple-Watch-App** als Begleiter. Ich habe derzeit keine Watch, also nur vorbereiten, nicht bauen.
6. **Wetter** beim Lauf speichern (WeatherKit), falls ohne großen Aufwand möglich.

## Phasenplan

- **Phase 1 – Grundgerüst:** Projekt, Navigation (Tabs: Laufen, Verlauf, Statistik, Einstellungen), SwiftData-Modelle (`Run`, `Split`, `RoutePoint`), Berechtigungsdialoge mit deutschen Texten (Standort „Immer/Beim Verwenden", Bewegung, Health).
- **Phase 2 – Aufzeichnung:** LocationService, GPS-Filter, WorkoutEngine mit Distanz, Pace, Splits und Auto-Pause, Live-Anzeige, Hintergrundbetrieb, Unit-Tests.
- **Phase 3 – Voice Coach:** km-Ansagen, Ducking, Zusammenfassung, Einstellungen.
- **Phase 4 – Auswertung:** Detailansicht mit Karte, Splits, Diagrammen.
- **Phase 5 – Verlauf und Statistik:** Bestleistungen, Wochenziel, 10-%-Hinweis.
- **Phase 6 – Export:** Apple Health, GPX, CSV.
- **Phase 7 – Feinschliff:** Live Activity, App-Icon, Dark Mode, Akku-Test, Fehlerfälle (GPS-Verlust, App-Absturz mitten im Lauf: der Lauf muss wiederherstellbar sein).
- Danach die Erweiterungen nach Absprache.

## Qualität und Tests

- Ein **GPX-Simulationsmodus** für den Simulator (Xcode-Location-Simulation), dazu eine Beispiel-GPX-Datei einer 5-km-Runde in Berlin-Mitte mit Ampelstopps.
- Unit-Tests für die gesamte Rechenlogik. Bei jeder Phase lässt du `xcodebuild test` laufen.
- Robust bei GPS-Verlust, etwa im Tunnel oder zwischen hohen Häusern: Lücken werden überbrückt, nicht als Teleport gezählt.
- Wird die App vom System beendet, ist der laufende Lauf zwischengespeichert und wird beim Neustart angeboten.

## Installation auf meinem iPhone

Erkläre mir am Ende Schritt für Schritt:
- wie ich in Xcode mein Apple-Konto als Team eintrage und die App auf mein iPhone 11 installiere,
- den Entwicklermodus auf dem iPhone (Einstellungen → Datenschutz & Sicherheit → Entwicklermodus),
- welche Einschränkungen ein **kostenloses Apple-Konto** hat. Die App muss dann etwa alle 7 Tage neu aufgespielt werden, und manche Capabilities funktionieren vielleicht nicht. Was bringt mir dagegen das Apple Developer Program?

Fang jetzt mit dem Plan für Phase 1 an.
