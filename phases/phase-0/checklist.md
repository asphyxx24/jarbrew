# Phase 0 — Vorbereitung

## Definition of Done
Alle Setup-Voraussetzungen erfüllt, PC bereit für Phase-1-Installation.

## Checkliste

- [x] Hostname `jarvis` gesetzt — Reboot durchgeführt, `hostname` bestätigt
- [x] BIOS/UEFI: "Wake on LAN" / "Power on by PCI-E" aktiviert — beim Reboot durch User aktiviert
- [x] Netzwerkadapter: "Allow this device to wake the computer" + "Only allow a magic packet" gesetzt — Realtek PCIe GbE: `*WakeOnMagicPacket=1`, `*WakeOnPattern=0`, `S5WakeOnLan=1`
- [x] Fast Startup deaktiviert: `Energieoptionen → Netzschalterverhalten → Schnellstart deaktivieren` — bereits gesetzt (HiberbootEnabled=0)
- [x] Sleep-Timer gesetzt: `powercfg /change standby-timeout-ac 60` — verifiziert: 0x00000e10 (3600 s) auf Plan „Ultimative Leistung"
- [x] PowerShell Execution Policy: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` — bereits gesetzt
- [ ] WoL-App auf Pixel eingerichtet (MAC-Adresse des PCs eingetragen, Test-Wake durchgeführt) — Host in „Wake On Lan"-App eingetragen ✓, **End-to-End-Test offen** (User macht morgen: PC schlafen lassen → Pixel-App tippen → PC wacht auf)

## Verifizierungs-Snapshot 2026-04-29

- MAC: `04-D9-F5-60-D2-C4` (Realtek PCIe GbE, 1 Gbps)
- IPv4: `192.168.178.21/24` · Broadcast: `192.168.178.255`
- WoL-App auf Pixel: „Wake On Lan" (Mike Webb)

## Stolpersteine

- **BIOS-WoL:** Einstellung heißt je nach Hersteller "Wake on LAN", "PCI Power Up" oder "Power On By PCIE/PCI". Manchmal unter "Advanced" → "APM Configuration".
- **Fast Startup:** Muss aus, sonst reagiert der PC nach Shutdown nicht auf Magic Packets (S5-Zustand statt S3/S4).
- **Windows Defender SmartScreen** kann bei Installer-Skripten warnen — "Trotzdem ausführen" klicken.
