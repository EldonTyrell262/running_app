# Entwicklungs-Mac einrichten: Mac mini Late 2012

Diese Anleitung bringt einen offiziell nicht mehr unterstützten Mac mini (Late 2012, Macmini6,x) mit dem **OpenCore Legacy Patcher (OCLP)** auf **macOS Sequoia**, damit dort **Xcode 26.3** läuft.

## Hardware

| Teil | Stand | Hinweis |
|---|---|---|
| Modell | Mac mini Late 2012 (Intel Ivy Bridge) | |
| Speicher | SSD 500 GB | gut |
| RAM | 8 GB | reicht, 16 GB (2 × 8 GB DDR3L-1600 SO-DIMM) wäre angenehmer |

## Warum Sequoia und Xcode 26.3?

- Apple liefert für diesen Mac offiziell höchstens **Catalina (10.15)**, und darauf läuft nur Xcode 12. Das ist zu alt für aktuelle iPhones.
- **Xcode 26.3 ist die letzte Version, die auf macOS Sequoia 15.6+ läuft.** Ab Xcode 26.4 ist macOS Tahoe 26.2 nötig, Xcode 27 verlangt Apple Silicon.
- Xcode 26.3 bringt das **iOS-26.2-SDK** mit (Deployment-Targets iOS 15–26.2). APIs, die erst ab iOS 26.3 kommen, stehen nicht zur Verfügung.
- Tahoe-Unterstützung gibt es im OCLP erst ab 3.0 (anfangs noch als Release Candidate), deshalb ist Sequoia die stabilere Wahl.

## Ablauf

1. **Catalina installieren:** Über die Internet-Wiederherstellung (beim Start **⌘ + ⌥ + R** halten, auf einer Windows-Tastatur **Windows + Alt + R**). Vorher die SSD im Festplattendienstprogramm als **APFS / GUID** löschen.
   - Falls sich das Laufwerk nicht aushängen lässt, im Terminal der Wiederherstellung:
     ```bash
     diskutil list                                   # interne SSD finden, z. B. disk0
     diskutil unmountDisk force disk0
     diskutil eraseDisk APFS "Macintosh HD" GPT disk0
     ```
   - Mit ⇧ + ⌘ + ⌥ + R wird stattdessen das Original-macOS (Mountain Lion) installiert. Das ist ein Umweg, weil OCLP mindestens 10.10 braucht.
2. **OCLP installieren** (in Catalina, Terminal):
   ```bash
   curl -L -o ~/Downloads/OpenCore-Patcher.pkg https://github.com/dortania/OpenCore-Legacy-Patcher/releases/latest/download/OpenCore-Patcher.pkg
   sudo installer -pkg ~/Downloads/OpenCore-Patcher.pkg -target /
   open /Applications/OpenCore-Patcher.app
   ```
3. **USB-Installer erstellen** (Stick ≥ 32 GB): In OCLP auf **Create macOS Installer → Download macOS Installer → macOS Sequoia 15.7.x** gehen, dann auf den Stick schreiben. Danach **OpenCore auf den Stick installieren**.
4. **Vom Stick booten:** Beim Start **Alt** halten, **EFI Boot** wählen, dann **Install macOS Sequoia**. Bei jedem Neustart wieder über EFI Boot und **macOS Installer** starten, bis die Einrichtung erscheint.
5. **Nach der Installation:**
   - OCLP → **Build and Install OpenCore** auf die **interne SSD**, dann bootet der Mac ohne Stick.
   - OCLP → **Post-Install Root Patch → Start Root Patching** → Neustart. Danach laufen Grafikbeschleunigung (Intel HD 4000) und WLAN.
   - Bis zum Root-Patch fehlt das WLAN. Notfalls das iPhone per USB-Kabel als Hotspot nutzen.
6. **Xcode 26.3** nicht aus dem App Store laden, sondern als `.xip` von <https://developer.apple.com/download/all/>, entpacken und nach `/Programme` verschieben.

## Damit nichts kaputtgeht

- **Automatische macOS-Updates ausschalten:** Systemeinstellungen → Allgemein → Softwareupdate → ⓘ → „macOS-Updates installieren“ **aus**.
- **Kein Upgrade auf Tahoe** über die Softwareaktualisierung.
- Sequoia-Updates erst installieren, wenn die OCLP-Release-Notes sie unterstützen, und danach immer die **Root-Patches erneut einspielen**. Ohne Patches friert der Mac nach dem Booten ein.

## Rückfallebene

Wenn das iPhone eine neuere iOS-Version hat, als Xcode 26.3 offiziell unterstützt, und die Installation per Kabel scheitert, bleiben die Cloud-Builds über GitHub Actions (`.github/workflows/ios-build.yml`). Damit kommt die App über TestFlight aufs iPhone, wofür das Apple Developer Program nötig ist.
