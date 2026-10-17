# Ortak

iOS-App (iPhone, iOS 18.6+) zum Mitschreiben von **Okey**-Runden. Ortak verwaltet Spieler und Teams, rechnet die Punkte nach den Hausregeln aus und wertet die Ergebnisse statistisch aus.

Die App ist kostenlos. Mit dem einmaligen In-App-Kauf **Ortak Pro** fallen die Limits weg.

## Funktionen

- **Spieler und Teams** anlegen (ein Team ist ein Paar aus 2 Spielern, optional mit eigenem Namen).
- **Spiel** mit 4 Spielern oder 2 Teams, 11 Runden. Wer am Ende die meisten Punkte hat, verliert.
- **Runde erfassen:** Faktor (×5, ×10, ×20), wer geöffnet hat, wie die Runde endete (Normal, Okey, Cift), Punkte pro Spieler und Cift-Markierung.
- **Runden nachträglich ändern:** Auf eine Runde in der Liste tippen, ein Sheet öffnet sich.
- **Statistik** (Pro): Spiele, Siege, Niederlagen, Punkte und Bilanz gegen jeden Gegner, für Spieler und Teams.
- **Farbthemen** mit Hell- und Dunkel-Variante.
- **Sprachen:** Deutsch (Hauptsprache) und Türkisch.

## Spielregeln, wie sie umgesetzt sind

Die Logik steckt in `Features/Games/Domain` (`Game`, `ScoreCalculator`) und ist unabhängig von der Oberfläche.

| Regel | Umsetzung |
| --- | --- |
| Cift | Die Punkte des Spielers werden verdoppelt, danach mit dem Faktor multipliziert. |
| Teamwertung | Summe der Spieler des Teams, jeder Spieler mit seiner eigenen Cift-Markierung. |
| Faktor | ×5, ×10, bei Sonderrunden ×20. |
| Bonus | Alle außer dem, der geöffnet hat, bekommen +50 (×5), +100 (×10) oder +200 (×20). Im Team-Spiel betrifft das nur das andere Team. Hat niemand geöffnet, gibt es keinen Bonus. |
| Okey oder Cift beendet | Der Bonus wird mit 10 multipliziert (500, 1000, 2000) und ersetzt den normalen Bonus. Dafür muss angegeben sein, wer beendet hat. |
| Pflicht-Cift | Jeder Spieler muss mindestens 2× Cift gehen, mehr ist erlaubt, auch in Runde 11. Wer vor Runde 9 noch keins hatte, muss in Runde 9 und 10 Cift gehen. Wer erst eins hatte, muss in Runde 10 Cift gehen. Die App markiert diese Spieler automatisch, sperrt den Button und zeigt einen Hinweis. |
| Nachträglich ändern | Eine Änderung wird abgelehnt, wenn dadurch eine spätere, schon gespielte Runde die Pflicht-Cift-Regel verletzt. |

## Free und Pro

| | Free | Pro |
| --- | --- | --- |
| Spieler | höchstens 4 | unbegrenzt |
| Teams | höchstens 2 | unbegrenzt |
| Statistik | gesperrt (Spiele werden trotzdem gespeichert) | ja |
| Eigene Spielmodi | – | geplant, in den Einstellungen als „Demnächst“ markiert |

Pro ist ein **einmaliger Kauf** (nicht verbrauchbar) über StoreKit 2. Produkt-ID: `de.duman.ortak.pro` (`Features/Pro/Data/StoreKitPurchaseService.swift`). Die Limits stehen in `FreeLimits`.

## Architektur

- MVVM mit `@Observable`-ViewModels, SwiftUI durchgehend, Swift Concurrency.
- Coordinator (`AppCoordinator`) hält die Navigationspfade der Tabs „Spiele“ und „Statistik“.
- Dependency Injection über den `AppContainer`. Repositories und Services sind Protokolle.
- Persistenz mit SwiftData. Die `@Model`-Klassen (`PlayerDTO`, `TeamDTO`, `GameDTO`) sind getrennt von den Domain-Entitäten, das Mapping steckt in eigenen Structs. Ein Spiel liegt als JSON (`GameRecord`) in `GameDTO.payload`.
- Das gewählte Farbthema liegt in `UserDefaults`.

```
Ortak/
  App/                 Einstieg, Container, Coordinator, Root- und Roster-View
  Shared/              Hilfen (Farben, Accessibility-IDs)
  Localizable.xcstrings  Texte Deutsch/Türkisch
  Features/
    Players/  Teams/  Games/  Statistics/  Settings/  Pro/
      Data/ Domain/ Presentation/
```

## Einrichtung

1. `Ortak.xcodeproj` in Xcode öffnen (Deployment Target iOS 18.6, nur iPhone).
2. **Pro lokal testen:** Edit Scheme → Run → Options → StoreKit Configuration → `Ortak.storekit` wählen. Ohne diese Auswahl zeigt die Paywall „Produkt nicht verfügbar“.
3. **Pro im Store:**
   - In App Store Connect ein In-App-Produkt vom Typ **Nicht verbrauchbar** mit der Produkt-ID `de.duman.ortak.pro` anlegen.
   - Die Vereinbarung „Bezahlte Apps“ aktivieren.
   - Die Capability „In-App Purchase“ ist nicht nötig. „StoreKit External Purchases“ nicht hinzufügen.

## Lokalisierung

Deutsch ist die Entwicklungssprache, die Texte im Code sind die deutschen Schlüssel. Türkische Übersetzungen stehen in `Ortak/Localizable.xcstrings`. Neue Texte erscheinen dort nach dem Build und müssen übersetzt werden. Texte, die nicht direkt in einer SwiftUI-View stehen, werden mit `String(localized:)` erzeugt.

## Tests

Unit-Tests mit Swift Testing in `OrtakTests` (Punkteberechnung, Spielregeln, Cift-Pflicht, Runden ändern, Statistik, Themes). UI-Tests mit XCTest in `OrtakUITests`. Die Accessibility-IDs stehen in `Shared/AccessibilityID.swift`.

## Offen

- Eigene Spielmodi (Pro): Regeln wie Rundenzahl, Faktoren und Strafen selbst festlegen.
- Das TestSupport-SDK für die Accessibility-IDs ist im Projekt nicht eingebunden.
- Türkische Texte von einer Muttersprachlerin oder einem Muttersprachler gegenlesen lassen.
