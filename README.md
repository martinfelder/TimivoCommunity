# Timivo - Time Tracker

Deine Arbeitszeit einfach verfolgen. Eine native iOS-App zur Erfassung, Auswertung und Verwaltung von Arbeitszeiten – mit Kundenunterstützung, Live Activity, gestaffelten Pausenregeln und automatischem Backup.

<p align="center">
  <img src="screenshots/01_timer.png" width="230" alt="Timer">
  <img src="screenshots/02_entries.png" width="230" alt="Letzte Einträge">
  <img src="screenshots/04_report_chart.png" width="230" alt="Auswertung">
</p>

---

## Features

### Zeiterfassung
- **Timer starten/stoppen** – sofort oder mit manuellem Startzeitpunkt
- **Pausenarten** – Mittag, Kaffee, Bezahlte Pause, Sonstige
- **Bezahlte Pausen** werden nicht von der Arbeitszeit abgezogen
- **Gestaffelte Pausenregeln** – automatischer Pausenabzug abhängig von der tatsächlichen Arbeitszeit (z.B. ab 5.5 Std. → 15 min, ab 7 Std. → 30 min)
- **Echtzeit-Anzeige** von Überzeit/Unterzeit und Fortschritt
- **Soll erreicht um** – Berechnung, wann die Soll-Arbeitszeit erreicht wird

### Anzeigemodi
- **Netto Arbeitszeit** – Laufzeit seit Start
- **Gesamt Arbeitszeit** – Laufzeit minus Soll-Mittagspause
- Umschaltbar in den Einstellungen

### Live Activity & Dynamic Island
- **Sperrbildschirm** – Live-Timer mit Laufzeit, Arbeitszeit, Pausen und Soll-Anzeige
- **Dynamic Island** – Kompakte Anzeige mit Live-Timer
- Timer tickt automatisch ohne App-Updates

<p align="center">
  <img src="screenshots/06_settings_customers.png" width="230" alt="Kunden & Einstellungen">
  <img src="screenshots/03_settings.png" width="230" alt="Einstellungen">
  <img src="screenshots/07_backup.png" width="230" alt="Backup">
</p>

### Kundenprofile
- **Kundenauswahl** beim Starten des Timers
- **Individuelle Einstellungen pro Kunde:**
  - Soll-Arbeitszeit (täglich & wöchentlich)
  - Gestaffelte Pausenregeln
  - Beschäftigungsgrad (1–100%)
- Kunden hinzufügen, bearbeiten und löschen

### Beschäftigungsgrad (Pensum)
- **Global** und **pro Kunde** konfigurierbar (1–100%)
- Reduziert die Soll-Arbeitszeit proportional
- Wird in Statistik und Auswertung berücksichtigt

### Letzte Einträge
- Übersicht der letzten Arbeitstage mit Kunde, Datum, KW, Zeiten, Pausen und Über-/Unterzeit
- Detailansicht mit allen Informationen pro Eintrag
- Bearbeitungsmodus – Einträge bearbeiten oder löschen
- Mehrfachauswahl zum Löschen

<p align="center">
  <img src="screenshots/05_report_details.png" width="230" alt="Auswertung Details & Export">
</p>

### Auswertungen
- **Zeiträume:** Tag, Woche, Monat, Jahr
- **Kundenfilter** für alle Auswertungen
- **Zusammenfassung:** Arbeitszeit, Pausen, Soll-Mittag, Anzahl Tage
- **Diagramme:** Summen-Balkendiagramm (gestapelt: Arbeit, Pausen, Soll-Mittag) und Aufschlüsselung nach Tagen, Wochen oder Monaten
- **Kunden-Aufteilung:** Anteil pro Kunde mit Wochen-Soll/Ist-Vergleich

### Export
- **CSV-Export** mit wählbarem Zeitraum und Kundenfilter
- **Teilen** direkt über den System-Share-Sheet

### Backup & Restore
- **Automatisches Backup** – täglich, wöchentlich oder an bestimmten Wochentagen
- Konfigurierbare Uhrzeit und maximale Backup-Anzahl
- **Manuelles Backup** jederzeit möglich
- **Wiederherstellen** – Zusammenführen oder Ersetzen
- Vollständige Sicherung inkl. aller Einstellungen und Kundenprofile

### Speicherort
- Lokale Speicherung (UserDefaults)
- Externer Speicherort wählbar (z.B. iCloud, Dateien-App)

---

## Technologie

| | |
|---|---|
| **UI** | SwiftUI |
| **Charts** | Swift Charts |
| **Live Activity** | ActivityKit |
| **Widget** | WidgetKit |
| **Plattform** | iOS 17+ |
| **Sprache** | Swift 5.9+ |

---

## Autor

Martin Felder

---

## Lizenz

Alle Rechte vorbehalten. &copy; 2026 Martin Felder
