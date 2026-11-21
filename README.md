# Laufbuch 🏃

Eine native iPhone-App (SwiftUI) zum Aufzeichnen täglicher Läufe, als persönliche Alternative zu adidas Running und Strava. Das Ziel ist die Vorbereitung auf den Berlin-Marathon 2027.

## Funktionen (geplant)

- GPS-Aufzeichnung im Hintergrund mit GPS-Filter und Auto-Pause an Ampeln
- Distanz, Zeit, aktuelle Pace, Ø-Pace und Pace pro Kilometer (Splits)
- Deutsche Sprachansage nach jedem Kilometer, Musik wird dabei nur leiser
- Gesprochene und angezeigte Zusammenfassung am Ende des Laufs
- Karte nach Pace eingefärbt, Diagramme, Höhenmeter per Barometer, Kadenz
- Verlauf, Wochen- und Monatsstatistik, Bestleistungen (1 km bis Marathon)
- Export an Apple Health, als GPX (z. B. für Strava) und als CSV

Den vollständigen Anforderungskatalog findest du in [`docs/PROMPT.md`](docs/PROMPT.md), den Arbeitsstand in [`CLAUDE.md`](CLAUDE.md).

## Voraussetzungen

- Ein Mac mit aktuellem Xcode. Welche Version nötig ist, hängt von der iOS-Version auf dem iPhone ab.
- Ein iPhone 11 oder neuer
- Ein Apple-Konto. Mit dem kostenlosen Konto muss die App alle 7 Tage neu aufgespielt werden. Das Apple Developer Program (99 €/Jahr) hebt diese Grenze auf und ermöglicht TestFlight.

## Loslegen mit Claude Code

```bash
git clone https://github.com/EldonTyrell262/running_app.git
cd running_app
claude
```

Dann in Claude Code:

> Lies `CLAUDE.md` und `docs/PROMPT.md` und starte mit dem Plan für Phase 1.

## Builds in der Cloud (GitHub Actions)

Der Workflow [`.github/workflows/ios-build.yml`](.github/workflows/ios-build.yml) baut und testet die App bei jedem Push auf einem macOS-Rechner von GitHub, sobald ein Xcode-Projekt existiert. Das ist praktisch, wenn der eigene Mac für das aktuelle Xcode zu alt ist.

## Lizenz

Privates Projekt für den persönlichen Gebrauch.
