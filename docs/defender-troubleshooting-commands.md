# Microsoft Defender Troubleshooting Commands

Diese Sammlung enthält PowerShell- und Windows-Befehle zur Analyse, Reparatur und Aktualisierung von Microsoft Defender Antivirus auf Windows 11 Clients.

Alle Befehle sind bewusst als Einzeiler formuliert, damit sie direkt kopiert und in einer administrativen PowerShell ausgeführt werden können.

> **Hinweis:** Viele Befehle benötigen eine PowerShell mit Administratorrechten. Registry-Änderungen sollten zuerst auf einem Testgerät geprüft werden.

---

## 1. Defender-Status prüfen

```powershell
Get-MpComputerStatus | Select-Object AMRunningMode,AMServiceEnabled,AntivirusEnabled,RealTimeProtectionEnabled,IoavProtectionEnabled,BehaviorMonitorEnabled,IsTamperProtected,NISEnabled,AMProductVersion,AMEngineVersion,AntivirusSignatureVersion,AntivirusSignatureLastUpdated
```

Zeigt den wichtigsten Gesamtstatus von Microsoft Defender Antivirus an. Besonders wichtig ist `AMRunningMode`: `Normal` bedeutet aktiv, `Passive Mode` weist meistens auf ein anderes AV-Produkt oder eine Policy hin.

---

## 2. Kompakter Malware-Protection-Ja/Nein-Check

```powershell
$s=Get-MpComputerStatus; [PSCustomObject]@{MalwareProtectionActive=($s.AMRunningMode -eq 'Normal' -and $s.AMServiceEnabled -eq $true -and $s.AntivirusEnabled -eq $true -and $s.RealTimeProtectionEnabled -eq $true); AMRunningMode=$s.AMRunningMode; AMServiceEnabled=$s.AMServiceEnabled; AntivirusEnabled=$s.AntivirusEnabled; RealTimeProtectionEnabled=$s.RealTimeProtectionEnabled; BehaviorMonitorEnabled=$s.BehaviorMonitorEnabled; IoavProtectionEnabled=$s.IoavProtectionEnabled; NISEnabled=$s.NISEnabled}
```

Gibt ein kompaktes Objekt zurück, das direkt zeigt, ob die Malware Protection aktiv ist. Praktisch für schnelle Vorher-/Nachher-Prüfungen.

---

## 3. Defender-Services und Security-Center-Services prüfen

```powershell
Get-Service -Name WinDefend,WdNisSvc,Sense,SecurityHealthService,wscsvc -ErrorAction SilentlyContinue | Select-Object Name,DisplayName,Status,StartType
```

Prüft die wichtigsten Dienste rund um Defender Antivirus, Defender for Endpoint Sensor, Windows Security und Security Center.

---

## 4. Defender-Services und Defender-Treiber prüfen

```powershell
Get-Service WinDefend,WdBoot,WdFilter,WdNisSvc,WdNisDrv,SecurityHealthService,wscsvc -ErrorAction SilentlyContinue | Format-Table -Auto DisplayName,Name,StartType,Status
```

Zeigt zusätzlich Defender-Filter- und NIS-Komponenten. Wichtig, wenn `WinDefend` nicht startet oder direkt wieder stoppt.

---

## 5. WinDefend manuell starten und Fehlermeldung ausgeben

```powershell
try { Start-Service WinDefend -ErrorAction Stop; 'WinDefend gestartet' } catch { "FEHLER beim Starten von WinDefend: $($_.Exception.Message)" }
```

Startet den Defender Antivirus Service manuell und gibt die konkrete Fehlermeldung zurück. Diese Fehlermeldung ist oft entscheidender als die Anzeige in Windows Security.

---

## 6. Registrierte Antivirus-Produkte anzeigen

```powershell
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct | Select-Object displayName,instanceGuid,productState,pathToSignedProductExe | Format-List
```

Zeigt, welche Antivirus-Produkte im Windows Security Center registriert sind. Verwaiste Einträge können Defender im passiven Modus halten.

---

## 7. Nur Trellix/McAfee im Security Center anzeigen

```powershell
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct | Where-Object { $_.displayName -match 'Trellix|McAfee' } | Select-Object displayName,instanceGuid,productState,pathToSignedProductExe | Format-List
```

Filtert gezielt nach Trellix- oder McAfee-Registrierungen. Nützlich nach einer AV-Migration zu Microsoft Defender.

---

## 8. Prüfen, ob der Trellix/McAfee-Dateipfad noch existiert

```powershell
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct | Where-Object { $_.displayName -match 'Trellix|McAfee' } | ForEach-Object { [PSCustomObject]@{DisplayName=$_.displayName; Path=$_.pathToSignedProductExe; Exists=(Test-Path $_.pathToSignedProductExe)} }
```

Prüft, ob der vom Security Center gemeldete Trellix-/McAfee-Pfad noch auf dem Dateisystem existiert. Wenn `Exists` auf `False` steht, ist der Eintrag wahrscheinlich verwaist.

---

## 9. Trellix/McAfee-Services suchen

```powershell
Get-Service | Where-Object { $_.Name -match 'mfe|mcafee|trellix|masvc|macmnsvc|mfemms|frminst' -or $_.DisplayName -match 'mfe|mcafee|trellix|masvc|macmnsvc|mfemms|frminst' } | Select-Object Name,DisplayName,Status,StartType
```

Sucht nach verbliebenen Trellix-/McAfee-Diensten. Wenn hier noch Einträge vorhanden sind, sollte nicht nur der Security-Center-Eintrag gelöscht werden.

---

## 10. Trellix/McAfee-Treiber suchen

```powershell
Get-CimInstance Win32_SystemDriver | Where-Object { $_.Name -match 'mfe|mcafee|trellix' -or $_.DisplayName -match 'mfe|mcafee|trellix' } | Select-Object Name,DisplayName,State,StartMode,PathName
```

Sucht nach verbliebenen Trellix-/McAfee-Treibern. AV-Treiberreste können Defender weiterhin blockieren oder Startprobleme verursachen.

---

## 11. Trellix/McAfee-Uninstall-Keys suchen

```powershell
Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*','HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*' -ErrorAction SilentlyContinue | Where-Object { $_.DisplayName -match 'Trellix|McAfee' } | Select-Object DisplayName,DisplayVersion,Publisher,UninstallString,QuietUninstallString | Format-List
```

Prüft, ob Windows noch Trellix-/McAfee-Produkte als installierte Software kennt. Falls ja, sollte die saubere Deinstallation oder das Trellix Removal Tool geprüft werden.

---

## 12. Trellix/McAfee-Ordnerreste suchen

```powershell
'C:\Program Files\McAfee','C:\Program Files\Trellix','C:\Program Files\Common Files\McAfee','C:\Program Files\Common Files\Trellix','C:\ProgramData\McAfee','C:\ProgramData\Trellix' | ForEach-Object { if (Test-Path $_) { "FOUND: $_" } }
```

Prüft typische Dateisystempfade auf Trellix-/McAfee-Reste. Nur weil die Software nicht mehr unter Apps sichtbar ist, bedeutet das nicht, dass alle Komponenten entfernt wurden.

---

## 13. Security-Center-Eintrag für Trellix/McAfee sichern

```powershell
New-Item -Path C:\Temp -ItemType Directory -Force | Out-Null; Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct | Select-Object displayName,instanceGuid,productState,pathToSignedProductExe | Format-List | Out-File C:\Temp\SecurityCenter2_AV_before_cleanup.txt
```

Exportiert die aktuelle Security-Center-AV-Registrierung vor einer Bereinigung. Sinnvoll, bevor ein verwaister AV-Eintrag entfernt wird.

---

## 14. Verwaisten Trellix/McAfee-Security-Center-Eintrag entfernen

```powershell
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct | Where-Object { $_.displayName -match 'Trellix|McAfee' } | Remove-CimInstance
```

Entfernt einen verwaisten Trellix-/McAfee-Eintrag aus `root\SecurityCenter2`. Nur verwenden, wenn keine Services, Treiber, Installationsreste und Dateipfade mehr vorhanden sind.

---

## 15. Security Center Services neu starten

```powershell
Restart-Service wscsvc -Force -ErrorAction SilentlyContinue; Restart-Service SecurityHealthService -Force -ErrorAction SilentlyContinue
```

Startet Windows Security Center und Windows Security Health Service neu. Nach dem Entfernen verwaister AV-Einträge ist ein Reboot trotzdem oft zuverlässiger.

---

## 16. Gerät neu starten

```powershell
Restart-Computer
```

Startet den Client neu. Bei AV-Migrationen und Treiber-/Security-Center-Bereinigungen ist ein Neustart meistens notwendig.

---

## 17. Nach Reboot Defender und AV-Provider prüfen

```powershell
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct | Select-Object displayName,productState,pathToSignedProductExe; Get-MpComputerStatus | Select-Object AMRunningMode,AMServiceEnabled,AntivirusEnabled,RealTimeProtectionEnabled,IsTamperProtected
```

Prüft nach einem Neustart, ob Trellix/McAfee weg ist und Defender wieder aktiv läuft.

---

## 18. Defender Policy Registry prüfen

```powershell
'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender','HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection','HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\Policy Manager','HKLM:\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection','HKLM:\SOFTWARE\Microsoft\Windows Defender' | ForEach-Object { Write-Host "`n=== $_ ==="; Get-ItemProperty $_ -ErrorAction SilentlyContinue | Select-Object DisableAntiSpyware,DisableAntiVirus,DisableRealtimeMonitoring,ForceDefenderPassiveMode,ServiceStartStates,IsServiceRunning }
```

Zeigt zentrale Defender-Policy-Werte. Besonders kritisch sind `DisableAntiSpyware`, `DisableAntiVirus`, `DisableRealtimeMonitoring` und `ForceDefenderPassiveMode`.

---

## 19. ForceDefenderPassiveMode prüfen

```powershell
Get-ItemProperty 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection' -ErrorAction SilentlyContinue | Select-Object ForceDefenderPassiveMode
```

Prüft, ob Defender per Policy absichtlich in den passiven Modus gezwungen wird.

---

## 20. ForceDefenderPassiveMode auf 0 setzen

```powershell
New-Item -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection' -Force | Out-Null; Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection' -Name ForceDefenderPassiveMode -Type DWord -Value 0
```

Setzt den passiven Modus explizit auf deaktiviert. Wenn GPO, Intune, ConfigMgr oder MDE Security Settings Management den Wert erneut setzen, kommt er nach Policy Refresh zurück.

---

## 21. DisableAntiSpyware löschen

```powershell
Remove-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender' -Name DisableAntiSpyware -Force -ErrorAction SilentlyContinue
```

Löscht den Policy-Wert `DisableAntiSpyware`. Dieser Wert kann Defender deaktivieren beziehungsweise am Start hindern, wenn er durch eine Management-Ebene gesetzt wurde.

---

## 22. DisableAntiSpyware und DisableAntiVirus prüfen

```powershell
Get-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender' -ErrorAction SilentlyContinue | Select-Object DisableAntiSpyware,DisableAntiVirus
```

Prüft, ob die deaktivierenden Policy-Werte noch vorhanden sind.

---

## 23. DisableAntiSpyware vorher sichern und dann löschen

```powershell
New-Item -Path C:\Temp -ItemType Directory -Force | Out-Null; reg export 'HKLM\SOFTWARE\Policies\Microsoft\Windows Defender' C:\Temp\DefenderPolicyBackup.reg /y; Remove-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender' -Name DisableAntiSpyware -Force -ErrorAction SilentlyContinue
```

Exportiert zuerst den Defender-Policy-Registry-Zweig und löscht danach `DisableAntiSpyware`. Geeignet, wenn du die Änderung nachvollziehbar dokumentieren willst.

---

## 24. Echtzeitschutz aktivieren, WinDefend starten und Status prüfen

```powershell
Set-MpPreference -DisableRealtimeMonitoring $false; Start-Service WinDefend -ErrorAction SilentlyContinue; Get-MpComputerStatus | Select-Object AMRunningMode,AMServiceEnabled,AntivirusEnabled,RealTimeProtectionEnabled
```

Versucht, Realtime Protection zu aktivieren, den Defender-Dienst zu starten und den Ergebnisstatus direkt auszugeben.

---

## 25. Policy Refresh ausführen und Werte erneut prüfen

```powershell
gpupdate /force; Get-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender' -ErrorAction SilentlyContinue | Select-Object DisableAntiSpyware,DisableAntiVirus
```

Prüft, ob die entfernten Policy-Werte durch Gruppenrichtlinien wieder gesetzt werden. Wenn sie zurückkommen, ist die lokale Änderung nicht die eigentliche Lösung.

---

## 26. Defender Events der letzten 7 Tage prüfen

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Windows Defender/Operational';StartTime=(Get-Date).AddDays(-7)} -ErrorAction SilentlyContinue | Where-Object { $_.LevelDisplayName -in 'Error','Warning' -or $_.Id -in 5007,5010,5013,5011,1116,1117,1118,1119 } | Select-Object TimeCreated,Id,LevelDisplayName,Message | Format-List
```

Liest relevante Defender-Warnungen und Fehler aus dem Operational Log. Besonders wichtig sind Konfigurationsänderungen, blockierte Änderungen und fehlgeschlagene Schutzaktionen.

---

## 27. Defender Events der letzten 3 Tage bei Startproblemen prüfen

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Windows Defender/Operational';StartTime=(Get-Date).AddDays(-3)} -ErrorAction SilentlyContinue | Where-Object { $_.LevelDisplayName -in 'Error','Warning' -or $_.Id -in 5001,5004,5007,5010,5011,5013,2001,2003,2004 } | Select-Object TimeCreated,Id,LevelDisplayName,Message | Format-List
```

Fokussiert auf Defender-Start- und Konfigurationsprobleme der letzten Tage.

---

## 28. Service-Control-Manager-Events zu Defender prüfen

```powershell
Get-WinEvent -FilterHashtable @{LogName='System';ProviderName='Service Control Manager';StartTime=(Get-Date).AddDays(-7)} -ErrorAction SilentlyContinue | Where-Object { $_.Message -match 'WinDefend|Windows Defender|Microsoft Defender|Sense' } | Select-Object TimeCreated,Id,LevelDisplayName,Message | Format-List
```

Sucht im System-Eventlog nach Dienstfehlern zu Defender und Defender for Endpoint.

---

## 29. Service-Control-Manager-Events der letzten 4 Stunden prüfen

```powershell
Get-WinEvent -FilterHashtable @{LogName='System';ProviderName='Service Control Manager';StartTime=(Get-Date).AddHours(-4)} -ErrorAction SilentlyContinue | Where-Object { $_.Message -match 'WinDefend|WdFilter|WdNisSvc|WdNisDrv|Windows Defender|Microsoft Defender' } | Select-Object TimeCreated,Id,LevelDisplayName,Message | Format-List
```

Schneller Fokus auf aktuelle Service-Startprobleme nach einem Reboot oder manuellen Startversuch.

---

## 30. GPResult für Defender-Analyse exportieren

```powershell
New-Item -Path C:\Temp -ItemType Directory -Force | Out-Null; gpresult /h C:\Temp\gpresult-defender.html
```

Erstellt einen HTML-Report der angewendeten Gruppenrichtlinien. Wichtig, wenn deaktivierende Defender-Werte nach `gpupdate` wiederkommen.

---

## 31. MDM-Diagnose exportieren

```powershell
New-Item -Path C:\Temp -ItemType Directory -Force | Out-Null; mdmdiagnosticstool.exe -out C:\Temp\MDMDiagReport.zip
```

Exportiert MDM-/Intune-Diagnosedaten. Relevant, wenn Intune Security Baselines oder Endpoint Security Policies Defender-Werte setzen.

---

## 32. Normales Defender-Signature-Update

```powershell
Update-MpSignature -Verbose
```

Startet ein normales Security-Intelligence-/Signaturupdate über die konfigurierte Updatequelle.

---

## 33. Defender-Signature-Update über Microsoft Update

```powershell
Update-MpSignature -UpdateSource MicrosoftUpdateServer -Verbose
```

Versucht das Signaturupdate gezielt über Microsoft Update.

---

## 34. Defender-Signature-Update über MMPC

```powershell
Update-MpSignature -UpdateSource MMPC -Verbose
```

Versucht das Signaturupdate direkt über Microsoft Malware Protection Center.

---

## 35. Defender-Signature-Update mit MpCmdRun

```powershell
$MpCmd=(Get-ChildItem 'C:\ProgramData\Microsoft\Windows Defender\Platform\MpCmdRun.exe' -Recurse -ErrorAction SilentlyContinue | Sort-Object FullName -Descending | Select-Object -First 1).FullName; if (-not $MpCmd) { $MpCmd='C:\Program Files\Windows Defender\MpCmdRun.exe' }; & $MpCmd -SignatureUpdate
```

Aktualisiert Signaturen über `MpCmdRun.exe`. Hilfreich, wenn PowerShell-Cmdlets nicht sauber funktionieren.

---

## 36. Definitionscache löschen und Signaturen neu laden

```powershell
$MpCmd=(Get-ChildItem 'C:\ProgramData\Microsoft\Windows Defender\Platform\MpCmdRun.exe' -Recurse -ErrorAction SilentlyContinue | Sort-Object FullName -Descending | Select-Object -First 1).FullName; if (-not $MpCmd) { $MpCmd='C:\Program Files\Windows Defender\MpCmdRun.exe' }; & $MpCmd -RemoveDefinitions -All; & $MpCmd -SignatureUpdate
```

Entfernt vorhandene Defender-Definitionen und lädt sie neu. Sinnvoll bei beschädigtem oder hängendem Signaturstand.

---

## 37. Defender Platform resetten, Defender aktivieren und Signaturen updaten

```powershell
$MpCmd=(Get-ChildItem 'C:\ProgramData\Microsoft\Windows Defender\Platform\MpCmdRun.exe' -Recurse -ErrorAction SilentlyContinue | Sort-Object FullName -Descending | Select-Object -First 1).FullName; if (-not $MpCmd) { $MpCmd='C:\Program Files\Windows Defender\MpCmdRun.exe' }; & $MpCmd -ResetPlatform; & $MpCmd -WdEnable; & $MpCmd -SignatureUpdate
```

Setzt die Defender-Plattform zurück, aktiviert Defender und führt ein Signaturupdate aus. Besonders relevant, wenn `WinDefend` nach AV-Migration nicht startet.

---

## 38. Vollständiger Reparaturversuch mit Reset, Aktivierung, Definitionslöschung und Update

```powershell
$MpCmd=(Get-ChildItem 'C:\ProgramData\Microsoft\Windows Defender\Platform\MpCmdRun.exe' -Recurse -ErrorAction SilentlyContinue | Sort-Object FullName -Descending | Select-Object -First 1).FullName; if (-not $MpCmd) { $MpCmd='C:\Program Files\Windows Defender\MpCmdRun.exe' }; & $MpCmd -ResetPlatform; & $MpCmd -WdEnable; & $MpCmd -RemoveDefinitions -All; & $MpCmd -SignatureUpdate; Get-MpComputerStatus | Select-Object AMRunningMode,AMServiceEnabled,AntivirusEnabled,RealTimeProtectionEnabled,AMProductVersion,AMEngineVersion,AntivirusSignatureVersion
```

Kombiniert mehrere Reparaturschritte in einem Lauf und zeigt anschließend den Defender-Status. Geeignet als pragmatischer Reparaturversuch nach Trellix-/McAfee-Cleanup.

---

## 39. Offline Security Intelligence Update herunterladen und installieren

```powershell
New-Item -Path C:\Temp -ItemType Directory -Force | Out-Null; Invoke-WebRequest -Uri 'https://go.microsoft.com/fwlink/?LinkID=121721&arch=x64' -OutFile C:\Temp\mpam-fe.exe; Start-Process C:\Temp\mpam-fe.exe -Wait
```

Lädt das aktuelle x64 Security-Intelligence-Paket direkt von Microsoft herunter und führt es aus. Sinnvoll, wenn der normale Updateweg nicht funktioniert.

---

## 40. Defender-Versionen und Signaturstand prüfen

```powershell
Get-MpComputerStatus | Select-Object AMProductVersion,AMEngineVersion,AntivirusSignatureVersion,AntivirusSignatureLastUpdated,AntispywareSignatureVersion,AntispywareSignatureLastUpdated
```

Zeigt Defender-Plattform-, Engine- und Signaturversionen sowie den letzten Aktualisierungszeitpunkt.

---

## 41. MpCmdRun-Log prüfen

```powershell
Get-Content 'C:\ProgramData\Microsoft\Windows Defender\Support\MpCmdRun.log' -Tail 100 -ErrorAction SilentlyContinue
```

Zeigt die letzten Einträge des Defender-Command-Line-Logs. Relevant bei Update-, Plattform- oder Aktivierungsproblemen.

---

## 42. DISM-Komponentenreparatur starten

```powershell
DISM.exe /Online /Cleanup-Image /RestoreHealth
```

Repariert den Windows-Komponentenspeicher. Sinnvoll, wenn Defender-Komponenten oder Windows-Systemdateien beschädigt sein könnten.

---

## 43. System File Checker ausführen

```powershell
sfc.exe /scannow
```

Prüft und repariert geschützte Windows-Systemdateien. Nach DISM ausführen, wenn Defender weiterhin nicht startet.

---

## 44. Kompletter Diagnose-Export

```powershell
$Out='C:\Temp\AV-Cleanup-Check'; New-Item -Path $Out -ItemType Directory -Force | Out-Null; Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct | Select-Object displayName,instanceGuid,productState,pathToSignedProductExe | Out-File "$Out\01_SecurityCenter_AV.txt"; Get-MpComputerStatus | Select-Object AMRunningMode,AMServiceEnabled,AntivirusEnabled,RealTimeProtectionEnabled,IsTamperProtected | Out-File "$Out\02_Defender_Status.txt"; Get-Service | Where-Object { $_.Name -match 'mfe|mcafee|trellix|masvc|macmnsvc|mfemms|frminst' -or $_.DisplayName -match 'mfe|mcafee|trellix|masvc|macmnsvc|mfemms|frminst' } | Select-Object Name,DisplayName,Status,StartType | Out-File "$Out\03_Trellix_Services.txt"; Get-CimInstance Win32_SystemDriver | Where-Object { $_.Name -match 'mfe|mcafee|trellix' -or $_.DisplayName -match 'mfe|mcafee|trellix' } | Select-Object Name,DisplayName,State,StartMode,PathName | Out-File "$Out\04_Trellix_Drivers.txt"; Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*','HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*' -ErrorAction SilentlyContinue | Where-Object { $_.DisplayName -match 'Trellix|McAfee' } | Select-Object DisplayName,DisplayVersion,Publisher,UninstallString,QuietUninstallString | Out-File "$Out\05_Trellix_UninstallKeys.txt"; Get-ItemProperty 'HKLM:\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection' -ErrorAction SilentlyContinue | Select-Object ForceDefenderPassiveMode | Out-File "$Out\06_ForceDefenderPassiveMode.txt"; Write-Host "Export erstellt: $Out"
```

Erstellt einen kompakten Diagnoseexport für AV-/Defender-Migrationen. Enthält Security-Center-AV-Provider, Defender-Status, Trellix-Reste und Passive-Mode-Policy.

---

## Empfohlene Reihenfolge bei Trellix-zu-Defender-Problemen

1. Defender-Status mit `Get-MpComputerStatus` prüfen.
2. Registrierte AV-Produkte in `root/SecurityCenter2` prüfen.
3. Trellix-/McAfee-Dienste, Treiber, Uninstall-Keys und Ordnerreste suchen.
4. Nur wenn wirklich keine Trellix-/McAfee-Reste mehr vorhanden sind: verwaisten Security-Center-Eintrag löschen.
5. Security Center neu starten oder Client rebooten.
6. `DisableAntiSpyware`, `DisableAntiVirus` und `ForceDefenderPassiveMode` prüfen.
7. Defender-Plattform resetten und Signaturen aktualisieren.
8. Bei weiterhin fehlerhaftem Start: Eventlogs, DISM und SFC prüfen.

---

## Kritischer Hinweis

Wenn Registry-Werte wie `DisableAntiSpyware`, `DisableAntiVirus` oder `ForceDefenderPassiveMode` nach `gpupdate` oder Reboot wieder auftauchen, ist das lokale Löschen nicht die eigentliche Lösung. Dann schreibt eine Management-Ebene den Wert erneut, zum Beispiel Gruppenrichtlinie, Intune, ConfigMgr, Security Baseline oder Microsoft Defender for Endpoint Security Settings Management.
