# CLAUDE.md – Laufbuch

Diese Datei ist das Gedächtnis des Projekts für Claude Code. Halte sie nach jeder Phase aktuell.

## Ziel

Laufbuch ist eine native iPhone-App zum Aufzeichnen täglicher Läufe (5–10 km, Berlin-Mitte, viele Ampelstopps) als Vorbereitung auf den Berlin-Marathon 2027. Sie ist für den persönlichen Gebrauch auf einem iPhone 11 gedacht: kein App Store, kein Backend, alle Daten lokal. Der vollständige Anforderungskatalog steht in `docs/PROMPT.md` und ist verbindlich.

## Arbeitsregeln

- Erst planen, dann auf das OK des Nutzers warten, dann coden.
- Phasenweise arbeiten. Jede Phase endet mit einer App, die baut und läuft, und mit einer Testanleitung für das iPhone.
- Der Nutzer ist kein iOS-Profi. Xcode-Schritte wie Signing, Capabilities und Geräte erklärst du Klick für Klick.
- UI-Texte auf Deutsch, Kommentare und Commit-Messages auf Deutsch.
- Keine externen Abhängigkeiten ohne Rückfrage, nur Apple-Frameworks.
- Nach jeder abgeschlossenen Teilaufgabe committen.
- Vor jeder Fertigmeldung `xcodebuild build` und `xcodebuild test` ausführen.

## Technik

- Swift 5.9+, SwiftUI, iOS 17+. Das Deployment-Target richtet sich nach der iOS-Version des iPhones.
- MVVM mit Services: `LocationService`, `WorkoutEngine`, `VoiceCoach`, `HealthKitService`, `PersistenceService`
- SwiftData, Swift Charts, MapKit, CoreLocation, CoreMotion (`CMAltimeter`, `CMPedometer`), AVFoundation, HealthKit
- Die Rechenlogik (Distanz, Splits, Pace, Auto-Pause) ist UI-frei und mit Unit-Tests abgedeckt.

## Build-Befehle

```bash
# Wird in Phase 1 ergänzt, sobald das Xcode-Projekt existiert, z. B.:
# xcodebuild -scheme Laufbuch -destination 'platform=iOS Simulator,name=iPhone 16' build test
```

## Phasen

- [ ] Phase 1 – Grundgerüst: Projekt, Tabs, SwiftData-Modelle, Berechtigungen
- [ ] Phase 2 – Aufzeichnung: GPS, Filter, Distanz, Pace, Splits, Auto-Pause, Hintergrund, Tests
- [ ] Phase 3 – Voice Coach: km-Ansagen, Ducking, Zusammenfassung
- [ ] Phase 4 – Auswertung: Karte nach Pace, Splits, Diagramme
- [ ] Phase 5 – Verlauf und Statistik: Bestleistungen, Wochenziel, 10-%-Hinweis
- [ ] Phase 6 – Export: Apple Health, GPX, CSV
- [ ] Phase 7 – Feinschliff: Live Activity, Icon, Dark Mode, Akku, Absturz-Wiederherstellung

## Stand

Das Projekt ist angelegt, es gibt noch keinen Code. Nächster Schritt: Plan für Phase 1.
