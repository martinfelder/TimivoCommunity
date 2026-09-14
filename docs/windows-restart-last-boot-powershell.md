# Windows: Letzten Neustart mit PowerShell prüfen

Mit den folgenden PowerShell-Befehlen lässt sich schnell prüfen, wann ein Windows-Rechner zuletzt gestartet wurde und wie lange er bereits läuft.

## Letzten Startzeitpunkt anzeigen

```powershell
(Get-CimInstance Win32_OperatingSystem).LastBootUpTime
```

Zeigt den Zeitpunkt des letzten Systemstarts als `DateTime`-Wert an.

## Letzten Startzeitpunkt formatiert anzeigen

```powershell
(Get-CimInstance Win32_OperatingSystem).LastBootUpTime.ToString("dd.MM.yyyy HH:mm:ss")
```

Gibt den letzten Startzeitpunkt kompakt im Format `TT.MM.JJJJ HH:mm:ss` aus.

## Laufzeit seit dem letzten Neustart anzeigen

```powershell
New-TimeSpan -Start (Get-CimInstance Win32_OperatingSystem).LastBootUpTime -End (Get-Date)
```

Zeigt die vergangene Zeit seit dem letzten Systemstart als `TimeSpan` an.

Alternativ als kompakte Textausgabe:

```powershell
(New-TimeSpan -Start (Get-CimInstance Win32_OperatingSystem).LastBootUpTime -End (Get-Date)).ToString()
```

## Remote-Rechner prüfen

```powershell
(Get-CimInstance Win32_OperatingSystem -ComputerName "PCNAME").LastBootUpTime
```

Fragt den letzten Systemstart eines Remote-Rechners ab. `PCNAME` muss durch den tatsächlichen Computernamen ersetzt werden.

Beispiel:

```powershell
(Get-CimInstance Win32_OperatingSystem -ComputerName "NB001234").LastBootUpTime
```

> Hinweis: Für die Remote-Abfrage müssen die erforderlichen Berechtigungen und die CIM/WinRM-Kommunikation zum Zielsystem verfügbar sein.

## Kompakte Übersicht mit Computername, Startzeit und Laufzeit

```powershell
$os=Get-CimInstance Win32_OperatingSystem; [PSCustomObject]@{ComputerName=$env:COMPUTERNAME; LastBootUpTime=$os.LastBootUpTime; Uptime=(New-TimeSpan -Start $os.LastBootUpTime -End (Get-Date))}
```

Praktisch für eine schnelle Diagnose, da Computername, letzter Startzeitpunkt und aktuelle Laufzeit gemeinsam ausgegeben werden.
