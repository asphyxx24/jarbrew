# Phase 0 — Vorbereitung

## Definition of Done
Alle Setup-Voraussetzungen erfüllt, PC bereit für Phase-1-Installation.

## Checkliste

- [ ] Hostname `jarvis` gesetzt — Reboot durchgeführt, `hostname` bestätigt
- [ ] BIOS/UEFI: "Wake on LAN" / "Power on by PCI-E" aktiviert
- [ ] Netzwerkadapter: "Allow this device to wake the computer" + "Only allow a magic packet" gesetzt
- [ ] Fast Startup deaktiviert: `Energieoptionen → Netzschalterverhalten → Schnellstart deaktivieren`
- [ ] Sleep-Timer gesetzt: `powercfg /change standby-timeout-ac 60`
- [ ] PowerShell Execution Policy: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`
- [ ] WoL-App auf Pixel eingerichtet (MAC-Adresse des PCs eingetragen, Test-Wake durchgeführt)

## Stolpersteine

- **BIOS-WoL:** Einstellung heißt je nach Hersteller "Wake on LAN", "PCI Power Up" oder "Power On By PCIE/PCI". Manchmal unter "Advanced" → "APM Configuration".
- **Fast Startup:** Muss aus, sonst reagiert der PC nach Shutdown nicht auf Magic Packets (S5-Zustand statt S3/S4).
- **Windows Defender SmartScreen** kann bei Installer-Skripten warnen — "Trotzdem ausführen" klicken.
