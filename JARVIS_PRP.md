# JARVIS — Personal Home AI Assistant (PRP)

> **PRP = Project Requirements Prompt.** Dieses Dokument ist der Einstiegspunkt für jede Claude-Code-Session, die an diesem Projekt arbeitet. Es enthält Zielbild, Architektur, Phasen und Entscheidungen und wird mitgepflegt.
>
> **Version:** 0.2 · **Stand:** 2026-04-22 · **Owner:** papageiAI
> **Vorgänger-Dokument:** `.context/attachments/pasted_text_2026-04-22_13-40-59.txt`

---

## Inhalt

1. [Zielbild & Nicht-Ziele](#1-zielbild--nicht-ziele)
2. [Architektur (4 Schichten)](#2-architektur-4-schichten)
3. [Typische Workflows](#3-typische-workflows)
4. [Eingabe-Modi A & B (PTT ↔ Wake-Word)](#4-eingabe-modi-a--b-ptt--wake-word)
5. [Hardware](#5-hardware)
6. [Software-Stack (Windows-nativ)](#6-software-stack-windows-nativ)
7. [Sicherheitsnetz](#7-sicherheitsnetz)
8. [Phasenplan 0–5 (MVP)](#8-phasenplan-05-mvp)
9. [MCP-Roadmap](#9-mcp-roadmap)
10. [Zukunftsroadmap (Phasen 6–12)](#10-zukunftsroadmap-phasen-612)
11. [Entscheidungstabelle](#11-entscheidungstabelle)
12. [Validierungskriterien pro Phase](#12-validierungskriterien-pro-phase)
13. [Glossar & Referenzen](#13-glossar--referenzen)
14. [Handoff für künftige Claude-Code-Sessions](#14-handoff-für-künftige-claude-code-sessions)

---

## 1. Zielbild & Nicht-Ziele

### Zielbild

Ein **persönlicher Universal-Assistent** — ein Jarvis im Wortsinn, kein Coding-Bot mit Voice-Aufsatz. Eine Identität, eine fortlaufende Session, voller Anwendungs-Mix. Konkret:

- per **Sprache angesprochen** werden, während ich durch die Wohnung laufe (Haupt-Use-Case)
- über **eine Konversation** Aufgaben verschiedener Tiefe übernehmen — von Mikro ("Licht aus", "Timer 10 Min", "wie spät") über Mittel (E-Mails triagieren, Browser-Recherche, Termine) bis Deep Work (Bachelorarbeit, Coding, mehrstufige Recherchen)
- Zugriff auf meine **Infrastruktur** haben: PC, Dateisystem, GitHub, Browser, Smart Home, eigene Accounts (E-Mail, Kalender)
- **remote** erreichbar sein (unterwegs, Bahn); dort reicht Text-/Diktat-Eingabe
- **sichtbar** arbeiten: persistente Sessions, von überall attachbar, optional ein eigener streambarer Desktop für Computer-Use-Tasks
- **mitwachsen**: neue Fähigkeiten kommen als MCP oder parallele Container hinzu, ohne Architektur-Bruch

### Kern-Klarstellung — wer ist "Jarvis"?

Ich rede mit **Claude (dem Modell)**. **Claude Code ist der Harness** — eine persistente Agent-Laufzeit mit MCP-Plugin-System, Permission-Flow, Compaction und Hooks. Das ist kein Coding-Tool, sondern ein Universal-Agent, in dem Coding *einer* der Anwendungsfälle ist. Smart Home, Browser, Recherche, E-Mail laufen über MCPs in derselben Session und teilen sich Kontext. Computer Use (Phase 6) ist ein paralleler Container, den dieselbe Session orchestriert.

Konsequenz: Mikro-Tasks ("Licht aus") laufen bewusst durch dasselbe Modell wie Deep-Work-Tasks. Latenz ~1–2 s und Token-Kosten ~0,5 Cent pro Mikro-Aktion sind der akzeptierte Preis für **eine Identität, einen Kontextfaden**. Ein vorgelagerter Mikro-Router (Hybrid-Architektur) ist eine spätere Optimierung — siehe Section 7 / Phase 6+ —, kein Phase-1-Thema.

### Nicht-Ziele

- Kein Voice-Setup von unterwegs (Text reicht)
- Kein Always-On-Listening als Default — Privacy-first, Wake-Word nur auf Wunsch
- Kein Hardware-Blindkauf vor Praxistest
- Kein zweites Orchestrator-LLM vor Phase 5 — eine Identität zuerst, Routing erst, wenn Praxisdaten zeigen, dass es nötig ist

---

## 2. Architektur (4 Schichten)

```
┌────────────────────────────────────────────────────────────────────┐
│  Windows-Host (nativ, kein WSL für Claude Code)                     │
│                                                                      │
│  ┌──────────── Schicht 1: Gehirn ───────────────┐                   │
│  │  Claude Code CLI (nativ)                      │                   │
│  │  - läuft in tmux unter Git Bash               │                   │
│  │  - persistente Session `jarvis`               │                   │
│  │  - MCPs: GitHub, Filesystem, Playwright, …    │                   │
│  └────────────────┬──────────────────────────────┘                   │
│                   │                                                   │
│  ┌──────────── Schicht 3: Eingabe ──────────────┐                   │
│  │  - `/voice` (PTT, Default)                    │                   │
│  │  - openWakeWord + faster-whisper (Modus B)    │                   │
│  │  - Unified Remote / Stream Deck Pedal         │                   │
│  └───────────────────────────────────────────────┘                   │
│                                                                      │
│  ┌──────────── Schicht 4: Agent-Desktop (ab Phase 6) ───────────┐   │
│  │  Docker Desktop → anthropic-quickstarts/computer-use-demo    │   │
│  │  Ubuntu 22.04 · xfce · VNC:5900 · noVNC:6080                 │   │
│  │  streambar via Browser über Tailscale                         │   │
│  └──────────────────────────────────────────────────────────────┘   │
└───────────────────┬──────────────────────────────────────────────────┘
                    │
  ┌─────────────── Schicht 2: Zugriff ──────────────┐
  │  Tailscale (MagicDNS + Tailscale SSH)            │
  └─────┬────────────────────────────────────┬───────┘
        │                                    │
   Android (Termux + mosh + tmux)    iPad / Browser
   → persistente Session              → Live-View Agent-Desktop
```

**Datenfluss-Beispiel "In Küche sagen 'commit'":**

1. PTT-Button am Handy gedrückt (Unified Remote) → Leertaste am PC wird gehalten.
2. Headset-Mic nimmt Sprache auf → Claude Code `/voice` transkribiert live.
3. Claude Code (in tmux) fasst Aktion zusammen, fragt bei Destruktivem zurück.
4. Claude führt `git commit` aus, Output landet in derselben tmux-Session.
5. Ich gehe ins Büro zurück, schaue aufs Fenster — alles sichtbar. Oder: remote via Tailscale attachen.

---

## 3. Typische Workflows

> **Diskussionsanker!** Jeder Schritt ist nummeriert. Sag "Workflow X Schritt Y soll anders sein", dann passen wir architektonisch an.

### Workflow 1 — Durch die Wohnung laufen (Primär-Use-Case)

1. Morgens PC hochfahren → Autostart öffnet Windows-Terminal-Tab mit Git Bash → `tmux new-session -A -s jarvis` → `claude`.
2. Bluetooth-Headset koppelt automatisch (einmal gepaart, danach Auto-Reconnect).
3. Modus-Anzeige im Tray ist grün → **PTT aktiv**.
4. In Küche gehen, Unified-Remote-Button am Handy halten → PTT triggert Leertaste am PC.
5. "Hey, committe den letzten Stand der Bachelorarbeit mit Message 'Kapitel 3 fertig'."
6. Claude Code zeigt Transkript, fasst Aktion zusammen: *"Ich würde jetzt im Repo `bachelor` `git add .` und commit mit 'Kapitel 3 fertig' ausführen — OK?"*
7. Button zweimal kurz drücken (= "ja"). Claude führt aus. Bestätigung per TTS (ab Phase 5) oder per ntfy-Push aufs Handy.
8. Weitermachen. Session bleibt offen, Kontext erhalten.

### Workflow 2 — Am Schreibtisch mit Wake-Word (Focus-Modus)

1. `jarvis-mode wake` im Terminal → openWakeWord-Service startet, Tray-Icon wird orange.
2. "Hey Claude, öffne den Playwright-MCP und lade mir Status meiner letzten Jira-Tickets."
3. Claude startet Tool-Call, fragt bei Unklarem zurück (auch per Voice möglich, wenn voice-mode MCP aktiv).
4. Wenn ich rausgehe: `jarvis-mode ptt` → Tray wieder grün, Mic schläft.

### Workflow 3 — Unterwegs in der Bahn (Remote)

1. Handy verbindet automatisch zu Tailscale.
2. Termux-Widget auf Homescreen antippen → `mosh pc.ts.net -- tmux new-session -A -s jarvis`.
3. Dieselbe Session wie zuhause — Claude Code läuft weiter, voller Kontext.
4. Per Tastatur oder Gboard-Diktat Prompts eingeben.
5. Netzwechsel (LTE↔U-Bahn) übersteht mosh transparent. Bei S-Bahn-Tunnel: kurz Pause, Session klebt.
6. Abends zuhause wieder attachen → nahtlos weiter.

### Workflow 4 — Claude recherchiert live im Agent-Desktop (ab Phase 6)

1. TV einschalten, Browser öffnen: `http://pc.ts.net:6080/vnc.html`.
2. Prompt an Claude: "Recherchiere aktuelle Richtlinien zu [Thema] und sammle die 5 besten Quellen in Obsidian."
3. Ich sehe live: Claude öffnet Firefox, googelt, scrollt, wählt aus.
4. Bei unklarer Entscheidung fragt er per Voice zurück: "Ich habe drei Papers, alle zusammenfassen oder nur das neueste?"
5. Am Ende liegen die Quellen im Obsidian-Vault (via Filesystem-MCP + gemountetem Volume zum Container).

---

## 4. Eingabe-Modi A & B (PTT ↔ Wake-Word)

Der User will die Wahl haben. Zwei Modi, explizit umschaltbar, **gegenseitig ausschließend**.

### Modus A — Push-to-Talk (Default, Privacy-first)

- Claude Code `/voice` (eingebaut seit März 2026, Mindestversion v2.1.69).
- Leertaste halten → Mic an. Loslassen → Mic aus.
- Trigger-Optionen (aufsteigend nach Aufwand):
  1. **Unified Remote** (Custom-Remote für "Space-Hold") — kostenlos, Handy als Fernbedienung.
  2. **Stream Deck Pedal** (~90 €) — Fußschalter, komfortabelster Weg beim Coden.
  3. **Flic 2 Button** (~35 €) — dezent, überall klebbar.
- Kein Always-On, kein Hintergrundprozess, kein Mithören.
- **Einschränkung:** `/voice` braucht **Claude.ai-Login**, nicht API-Key/Bedrock/Vertex. Siehe [Claude Code Voice Docs](https://code.claude.com/docs/en/voice-dictation).

### Modus B — Wake-Word "Hey Claude"

- Lokal, komplett offline:
  - **openWakeWord** als Windows-Service (via NSSM) — ~150 MB RAM, <5 % CPU idle.
  - Custom-"Hey Claude"-Modell, trainierbar in ~1h mit synthetischen Samples.
  - **faster-whisper** transkribiert den Satz nach Wake-Word-Trigger.
- Transkript wird via `tmux send-keys -t jarvis "<Text>" Enter` in die Claude-Code-Session gepiped.
- Alternative: **Picovoice Porcupine** — robuster bei False-Positives, kostenlos für Personal Use.

### Umschalt-Skript `jarvis-mode.ps1`

```powershell
# Pseudocode — Implementierung in Phase 3
param([ValidateSet('ptt','wake','status')][string]$Mode)

switch ($Mode) {
  'ptt' {
    Stop-Service openwakeword -ErrorAction SilentlyContinue
    Set-TrayIcon -Color Green -Tooltip 'Jarvis: PTT aktiv'
    Invoke-TmuxSend 'jarvis' '# Modus: PTT aktiv (nur auf Knopfdruck)'
  }
  'wake' {
    Start-Service openwakeword
    Set-TrayIcon -Color Orange -Tooltip 'Jarvis: Wake-Word aktiv'
    Invoke-TmuxSend 'jarvis' '# Modus: Wake-Word aktiv — sag "Hey Claude"'
  }
  'status' { Get-Service openwakeword }
}
```

### Wichtig: **Sichtbarkeit**

- Tray-Icon-Farbe zeigt aktiven Modus **immer** an.
- Beim Booten: immer PTT (grün). Wake-Word muss explizit gestartet werden.
- **Privacy-Leitlinie:** Wenn unsicher, welcher Modus läuft → `jarvis-mode status` liefert Wahrheit.

---

## 5. Hardware

### Bestehend

- **Windows-PC (Hauptnutzer)** — 24/7 geplant.
  - CPU: Ryzen 7 5800X (8C/16T, Zen 3)
  - RAM: 32 GB
  - GPU: AMD Radeon RX 6700XT (12 GB VRAM) — **kein CUDA**, Vulkan-Backend nutzbar
  - OS: Windows 11
  - Storage: NVMe SSD
  - Implikation: Lokale ML-Workloads müssen CPU- oder Vulkan-/DirectML-Pfade nutzen. CUDA-spezifische Optimierungen entfallen.
- **Android-Handy** — bereits vorhanden.
- **Sekundär-Nutzer (geplant)** — MacBook (Chef). Adapter-Architektur muss plattformübergreifend funktionieren; Apple-Silicon-Backends (Metal in whisper.cpp) sind dort sogar performant.

### Zu beschaffen / entscheiden

| Komponente | Zweck | Optionen | Empfehlung |
|---|---|---|---|
| Bluetooth-Headset mit gutem Mic | Voice-Eingabe beim Rumlaufen | Sony WH-1000XM5, AirPods Pro (2. Gen+), Shokz OpenComm | **Shokz OpenComm** — Open-Ear, Umgebung hörbar, Mic sehr gut für Voice. |
| PTT-Trigger | Hands-free Leertaste | Unified Remote (gratis, Handy), Stream Deck Pedal (~90 €), Flic 2 (~35 €) | **Unified Remote zuerst** — null Kosten, testet das Konzept. Bei Komfort-Bedarf auf Pedal aufrüsten. |
| Portable Display (optional) | Monitoring unterwegs im Haus | Gebrauchtes iPad, USB-C-Monitor 14–16" | **Gebrauchtes iPad** — vielseitiger (Browser + SSH + Claude App). Später, wenn Need erwiesen. |
| Flic Button (optional) | Schnellstart-SSH von unterwegs | Flic 2 + Hub LR (~80 €) | Erst nach Phase 4, wenn Bedarf spürbar. |

---

## 6. Software-Stack (Windows-nativ)

> Alle Kommandos laufen in **PowerShell als Admin** oder **Git Bash**. Kein WSL für Claude Code.

### Auf dem Windows-Host

```powershell
# 1. Git for Windows (bringt Git Bash mit)
winget install --id Git.Git -e

# 2. Node.js LTS (falls nicht schon da — Claude Code bringt eigenen Installer mit, Node ist trotzdem praktisch)
winget install --id OpenJS.NodeJS.LTS -e

# 3. Claude Code nativ
irm https://claude.ai/install.ps1 | iex
claude --version   # muss ≥ 2.1.69 sein

# 4. Tailscale
winget install --id Tailscale.Tailscale -e
# dann: einmal UI öffnen, Account verknüpfen, MagicDNS in Admin-Console aktivieren

# 5. OpenSSH-Server (für Remote-Zugriff)
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Set-Service sshd -StartupType Automatic
Start-Service sshd
# Firewall-Regel sollte automatisch angelegt werden; sonst New-NetFirewallRule …

# 6. Docker Desktop (für Agent-Desktop ab Phase 6)
winget install --id Docker.DockerDesktop -e

# 7. GitHub CLI
winget install --id GitHub.cli -e
gh auth login
```

### In Git Bash

```bash
# tmux via MSYS2-pacman oder direkt aus Git-SDK; einfachste Variante:
pacman -S tmux   # falls pacman verfügbar ist (Git-SDK-Installationen)

# Autostart-Shortcut in shell-rc:
echo 'alias jarvis="tmux new-session -A -s jarvis \"claude\""' >> ~/.bashrc
```

### Auf dem Android-Handy

- **Tailscale** — gleicher Account wie PC.
- **Termux** (von F-Droid, *nicht* Play Store!) — kostenlos, Open Source.
  ```bash
  pkg install openssh mosh tmux
  ```
- **Termux:Widget** — Homescreen-Shortcuts zur Jarvis-Session.
- **Unified Remote** (als PTT-Trigger, falls Modus A primär genutzt).
- **Claude Mobile App** — für Voice-Mode-Gespräche unterwegs (eigener Kontext, nicht die tmux-Session).

### MCP-Startkonfiguration (Phase 1)

- **GitHub MCP** — Bachelorarbeit + Projekte.
- **Filesystem MCP** — zielgerichtet auf `C:\Projekte`, `C:\BA` (nicht Root, nicht Home).
- **Playwright MCP** — für Overleaf-UI und sonstige Browser-Tasks.

---

## 7. Sicherheitsnetz

Eingaben aus Voice sind fehleranfällig (Transkription, Mehrdeutigkeit). Drei Ebenen, inkrementell aktivierbar:

### Ebene 1 — `CLAUDE.md`-Regeln (sofort, kostenlos)

Siehe [`CLAUDE.md`](./CLAUDE.md) im Repo-Root. Kernregeln:

- Bei abgehackter/mehrdeutiger/wahrscheinlich-transkribierter Eingabe: **präzise Rückfrage** vor Handlung.
- Vor destruktiven Aktionen (löschen, `git push`, Deployments, Paketinstalls): **1-Satz-Zusammenfassung + warten auf "ja"**.
- Bei unklarem Kontext: fragen statt raten.

### Ebene 2 — Verifier-Middleware (optional, nach 1–2 Wochen Praxis)

- Python-Skript (~100 Zeilen).
- Transkript → Claude Haiku Prompt: `READY: <bereinigter Text>` oder `CLARIFY: <Rückfrage>`.
- `READY` → via `tmux send-keys` in Claude-Code-Session.
- `CLARIFY` → per TTS vorlesen, Antwort holen, Loop.
- Kosten: < 1 Cent pro Anfrage, Latenz ~200 ms.
- Entscheidung: bauen, wenn Ebene 1 in der Praxis durchgelassen hat, was sie nicht hätte durchlassen sollen.

### Ebene 3 — `voice-mode` MCP mit 2-Wege-Dialog (Phase 5)

- Claude antwortet gesprochen, Rückfragen werden natürlich.
- Ersetzt Ebene 2 nicht, ergänzt sie.
- Lokaler Stack empfohlen: **Whisper STT + Kokoro TTS** (null laufende Kosten, Privacy).

### Ebene 4 — Container-Isolation (Agent-Desktop, Phase 6+)

- Computer-Use läuft im Docker-Container, isoliert vom Host-Filesystem.
- Nur explizit gemountete Volumes sind schreibbar.
- Reboot = Reset (optional via `readonly-rootfs + tmpfs`).

---

## 8. Phasenplan 0–5 (MVP)

### Phase 0 — Vorbereitung (~15 Min)

- [ ] Windows-Version prüfen (Windows 11 empfohlen; 10 funktioniert, hat aber weniger Audio-Polish).
- [ ] Entscheidung: Rechner 24/7 vs. Wake-on-LAN? → **Zum Start 24/7, Sleep aus.**
- [ ] Git for Windows installiert? Powershell Execution Policy = RemoteSigned?

### Phase 1 — Grundsetup (~60 Min)

- [ ] Tailscale auf PC + Handy, MagicDNS aktivieren, Verbindung testen (`ping pc.ts.net`).
- [ ] Claude Code installieren, `claude --version` → ≥ 2.1.69.
- [ ] OpenSSH-Server aktivieren, Tailscale-SSH ausprobieren (ACL-Regel "tag:personal kann tag:personal").
- [ ] `tmux new-session -A -s jarvis` in Git Bash, `claude` starten, `Ctrl+b d` zum Detachen testen.
- [ ] `gh auth login`, GitHub- + Filesystem-MCP konfigurieren.

### Phase 2 — Voice zuhause, PTT-Modus (~45 Min)

- [ ] Bluetooth-Headset mit PC koppeln, als Standard-Input setzen.
- [ ] In Claude Code `/voice` testen (Leertaste halten, sprechen).
- [ ] Unified Remote PC-Server + Android-Client installieren, Custom-Remote "PTT Space Hold" bauen.
- [ ] Realtest: durch Wohnung laufen, Mic-Reichweite prüfen. Falls zu schwach → Phase 2b: Audio-Verstärkung oder externes Mic am PC.
- [ ] **Entry für Phase 3:** Sprachbefehl aus Küche klappt stabil.

### Phase 3 — Sicherheitsnetz Ebene 1 (~30 Min)

- [ ] `CLAUDE.md` im Repo-Root bereits vorhanden — Inhalt reviewen, ggf. anpassen.
- [ ] Claude-Code-Permission-Prompts strikt einstellen (nicht alles autoapproven).
- [ ] Absichtlich mehrdeutigen Voice-Befehl absetzen → prüfen, ob Claude zurückfragt.
- [ ] Destruktiven Befehl ("lösche alle .md-Dateien") → prüfen, ob 1-Satz-Zusammenfassung + Warten kommt.

### Phase 4 — Remote-Zugang (~30 Min)

- [ ] Termux auf Android (F-Droid), `pkg install openssh mosh tmux`.
- [ ] SSH-Profil mit Tailscale-Hostname, Key-Auth einrichten.
- [ ] Startup-Command `mosh pc.ts.net -- tmux new-session -A -s jarvis`.
- [ ] Termux:Widget-Shortcut auf Homescreen.
- [ ] Test aus mobilem Netz (nicht Heim-WLAN): attachen, Aktion absetzen, Netzwechsel überstehen.

### Phase 5 — Zwei-Wege-Sprache + Wake-Word (~90 Min)

- [ ] voice-mode MCP installieren:
  ```powershell
  # Voraussetzung: uv installiert
  irm https://astral.sh/uv/install.ps1 | iex
  claude mcp add --scope user voicemode -- uvx --refresh --with webrtcvad --with "setuptools<71" voice-mode
  ```
- [ ] Kokoro-FastAPI Auto-Setup über voice-mode ausführen, LaunchAgent/Service prüfen (Port 8880).
- [ ] Whisper-STT lokal einrichten (voice-mode macht das im Normalfall selbst).
- [ ] openWakeWord installieren, Custom-"Hey Claude"-Modell trainieren oder Picovoice Porcupine nutzen.
- [ ] `jarvis-mode.ps1`-Skript schreiben (PTT/Wake-Toggle), in `C:\Tools\` ablegen, PATH setzen.
- [ ] Tray-Icon-Indikator (BurntToast oder AutoHotkey-Skript).
- [ ] Entry-Kriterium: "Hey Claude, was ist die Uhrzeit" → antwortet gesprochen, ohne API-Kosten.

---

## 9. MCP-Roadmap

**Start-Set (Phase 1):**

- GitHub MCP — `https://github.com/github/github-mcp-server` oder equivalent.
- Filesystem MCP — mit whitelist (`C:\Projekte`, `C:\BA`).
- Playwright MCP — für Overleaf + Web-Recherche.

**Erweiterungs-Trigger:**

| Erweiterung | Wann installieren |
|---|---|
| Obsidian MCP | Sobald Notizen-Integration gewünscht |
| Jira / Confluence MCP | Wenn Job-Kontext mit Jira startet |
| Computer-Use MCP | Mit Phase 6 (Agent-Desktop) |
| Home Assistant MCP | Mit Phase 8 (Smart Home) |
| Gmail / IMAP MCP | Mit Phase 9 (E-Mail-Triage) |

---

## 10. Zukunftsroadmap (Phasen 6–12)

> "Living Roadmap" — modular, jede Phase für sich aktivierbar.

### Phase 6 — Agent-Desktop (Claude's eigene Workstation) · ~2 h

- **Bedarf:** Claude soll Dinge mit Maus/Keyboard tun, ich will live zusehen können.
- **Skizze:** Docker-Container basierend auf `anthropic-quickstarts/computer-use-demo`, Volumes auf Projekt-Ordner.
- **Streaming:** noVNC auf Port 6080, erreichbar via Tailscale auf Handy/TV/iPad-Browser.
- **Entry-Kriterium:** Phase 5 stabil im Alltag.

### Phase 7 — Multi-Agent via Conductor · ~30 Min

- **Bedarf:** Parallel arbeitende Agents (Bachelorarbeit / Job / Recherche).
- **Skizze:** Conductor-Workspaces pro Kontext, Claude Code startet pro Workspace eine dedizierte tmux-Session.
- **Entry:** Spürbarer Bedarf nach Parallelität.

### Phase 8 — Smart-Home (Home Assistant MCP)

- **Bedarf:** "Jarvis, dim das Licht im Büro."
- **Skizze:** Home-Assistant-Instanz (Container oder HAOS), HA-MCP-Server, Zigbee-/Matter-Adapter nach Bedarf.
- **Entry:** Smart-Home-Hardware vorhanden.

### Phase 9 — E-Mail- / Kalender-Assistent

- **Bedarf:** Triage, Drafts, Terminvorschläge.
- **Skizze:** Gmail-/IMAP-MCP + Google-Calendar-MCP, Policy in `CLAUDE.md` für Mail-Kontext (nie ohne Rückfrage senden).
- **Entry:** Credentials-Komfort da.

### Phase 10 — Telefonie / SMS-Filter (spekulativ)

- VoIP-Setup, eingehende Anrufe filtern, Transkription + Summary.

### Phase 11 — RAG über Bachelorarbeit + Notizen

- **Skizze:** Chroma oder LanceDB lokal, Embeddings via lokales Modell, Claude-Custom-Tool.
- **Entry:** Dokumentenvolumen jenseits von "alles in den Kontext kippen".

### Phase 12 — Dedizierter Mini-PC als Jarvis-Host

- **Skizze:** Raspberry Pi 5 oder NUC, 24/7, Hauptrechner bleibt reine Workstation.
- **Entry:** Hauptrechner zu beschäftigt / zu viel Ressourcenverbrauch durch Jarvis.

---

## 11. Entscheidungstabelle

> Default-Vorschläge mit ✓/✗-Spalte für späteres Abhaken.

| # | Frage | Default | ✓/✗ | Notiz |
|---|---|---|---|---|
| 1 | Heimrechner-OS | Windows nativ | ✓ | User bestätigt |
| 2 | 24/7 oder WoL | 24/7, Sleep aus | — | Nach 1 Monat Stromrechnung prüfen |
| 3 | Bluetooth-Headset | Shokz OpenComm | — | Alternativ Sony WH-1000XM5 |
| 4 | PTT-Trigger erste Iteration | Unified Remote | — | Upgrade auf Stream Deck Pedal nur bei Komfort-Bedarf |
| 5 | Bachelorarbeit-Tool | LaTeX (Overleaf) | ✓ | User bestätigt. Empfehlung: Overleaf-Git-Sync + Filesystem MCP |
| 6 | Verifier-Ebene 2 bauen | Erst nach 1–2 Wochen Praxis | — | Datenbasis vor Implementation |
| 7 | Portable Display | Später — gebrauchtes iPad | — | Nach Phase 4 re-evaluieren |
| 8 | STT-Anbieter (Primary) | Groq Whisper Large v3 Turbo | ✓ | ~$1.80/Monat bei 45 h. `language="auto"` Pflicht. Eskalation auf large-v3 möglich. Details: `STT_TTS_OPTIONS.md` |
| 8a | TTS-Anbieter (Primary) | Microsoft Edge-TTS | ✓ | gratis, deutsche Neural-Stimmen. Stimme noch zu wählen (Killian/Katja/Conrad) |
| 8b | STT-Lokal-Fallback | faster-whisper `medium` (CPU) oder whisper.cpp + Vulkan | ✓ | CPU-Variante einfach, Vulkan performant aber Setup-Aufwand. AMD-GPU = kein CUDA |
| 8c | TTS-Lokal-Fallback | Piper | ✓ | CPU, ~5–10× Realtime, läuft Win + Mac |
| 8d | STT/TTS-Architektur | Adapter-Pattern, Backends austauschbar | ✓ | Wiederverwendbarkeit (Mac-Nutzer); Cloud-Backup-Slots: Deepgram, Azure Neural, Google Neural2 |
| 8e | Verworfene STT/TTS-Anbieter | OpenAI (STT+TTS), AWS Polly, AWS Transcribe, ElevenLabs, EC2-Self-Hosting | ✓ | Begründung dokumentiert in `STT_TTS_OPTIONS.md` |
| 9 | Wake-Word-Engine | openWakeWord | — | Picovoice Porcupine als Backup bei False-Positives |
| 10 | Agent-Desktop-Backend | Docker-Container | — | Keine WSL2-Abhängigkeit für Claude Code, Docker-Desktop ok |

---

## 12. Validierungskriterien pro Phase

- **Phase 1:** Vom Handy via Tailscale in `jarvis`-tmux einloggen, Claude antwortet auf "hi", Detach/Attach klappt.
- **Phase 2:** Durch Wohnung laufen, Befehl absetzen, Claude transkribiert korrekt und reagiert.
- **Phase 3:** Vager Befehl → Claude fragt nach statt zu raten. Destruktiver Befehl → 1-Satz-Zusammenfassung, wartet auf "ja".
- **Phase 4:** Aus Café/LTE attachen und Aktion laufen lassen; Netzwechsel im Zug übersteht mosh.
- **Phase 5:** "Hey Claude, was ist die Uhrzeit" → gesprochene Antwort; Umschalter PTT↔Wake-Word funktioniert, Tray-Icon ändert Farbe.
- **Phase 6:** Browser auf `http://pc.ts.net:6080` zeigt Live-Desktop, Claude öffnet per Prompt eine Website, ich sehe es live.

---

## 13. Glossar & Referenzen

**Begriffe:**

- **PTT** — Push-to-Talk, Mic nur aktiv wenn Knopf gehalten.
- **MCP** — Model Context Protocol, standardisierter Weg, Tools an Claude anzubinden.
- **tmux** — Terminal-Multiplexer, erhält Sessions über Disconnects.
- **mosh** — Mobile Shell, übersteht Netzwechsel, was SSH nicht tut.
- **MagicDNS** — Tailscale-Feature, vergibt DNS-Namen pro Gerät.

**Weblinks:**

- Claude Code Voice Dictation: https://code.claude.com/docs/en/voice-dictation
- Claude Computer Use API: https://docs.anthropic.com/en/docs/build-with-claude/computer-use
- Anthropic Computer-Use-Demo Container: https://github.com/anthropics/anthropic-quickstarts
- voice-mode MCP: https://voice-mode.readthedocs.io/ · https://github.com/mbailey/voicemode
- Tailscale SSH: https://tailscale.com/docs/features/tailscale-ssh
- openWakeWord: https://github.com/dscripka/openWakeWord
- faster-whisper: https://github.com/SYSTRAN/faster-whisper
- Picovoice Porcupine: https://picovoice.ai/platform/porcupine/
- Termux: https://termux.dev/ (F-Droid)
- Unified Remote: https://www.unifiedremote.com/

---

## 14. Handoff für künftige Claude-Code-Sessions

**Wenn du diese PRP frisch liest und arbeiten sollst, mach in dieser Reihenfolge:**

1. **Lies `CLAUDE.md` im Repo-Root.** Dort stehen die Voice-Sicherheitsregeln — für jede Voice-Interaktion relevant.
2. **Finde die aktuelle Phase.** Schau in Section 8, welche Checkbox als nächstes offen ist, oder frag den User.
3. **Prüfe offene Entscheidungen in Tabelle 11.** Falls noch ✗ oder "—" bei einer Phase-relevanten Zeile: frag den User, bevor du installierst.
4. **Halte die PRP aktuell.** Wenn du einen Schritt abschließt, hake die Checkbox ab und committe (nur wenn der User das wünscht).
5. **Niemals ohne Rückfrage installieren, pushen, löschen.** CLAUDE.md-Regel Ebene 1 gilt.

**Was bereits fest steht:** OS = Windows nativ. Bachelorarbeit = LaTeX/Overleaf. STT = Groq Whisper Large v3 Turbo (Cloud). TTS = Microsoft Edge-TTS (Cloud, gratis). Lokale Fallbacks via Adapter-Architektur (faster-whisper / whisper.cpp + Piper). PTT ist Default, Wake-Word opt-in. Details und verworfene Optionen in `STT_TTS_OPTIONS.md`.

**Was als nächstes ansteht:** Phase 1 (Grundsetup) — siehe Section 8.
