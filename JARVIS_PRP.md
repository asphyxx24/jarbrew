# JARVIS — Personal Home AI Assistant (PRP)

> **PRP = Project Requirements Prompt.** Dieses Dokument ist der Einstiegspunkt für jede Claude-Code-Session, die an diesem Projekt arbeitet. Es enthält Zielbild, Architektur, Phasen und Entscheidungen und wird mitgepflegt.
>
> **Version:** 0.3 · **Stand:** 2026-04-28 · **Owner:** asphyxx24 · **Repo:** `asphyxx24/jarbrew`
> **Vorgänger-Dokument:** `.context/attachments/pasted_text_2026-04-22_13-40-59.txt` (archiviert)

---

## Inhalt

1. [Zielbild & Nicht-Ziele](#1-zielbild--nicht-ziele)
2. [Nutzerprofil & Kontext](#2-nutzerprofil--kontext)
3. [Kern-Workflows](#3-kern-workflows)
4. [Architektur-Überblick](#4-architektur-überblick)
5. [Eingabe-Modi A & B (PTT ↔ Wake-Word)](#5-eingabe-modi-a--b-ptt--wake-word)
6. [Hardware](#6-hardware)
7. [Software-Stack (Windows-nativ)](#7-software-stack-windows-nativ)
8. [Sicherheitsnetz & Action-Queue](#8-sicherheitsnetz--action-queue)
9. [Phasenplan 0–9 (MVP)](#9-phasenplan-09-mvp)
10. [MCP-Roadmap](#10-mcp-roadmap)
11. [Zukunftsroadmap (Phasen 10+)](#11-zukunftsroadmap-phasen-10)
12. [Entscheidungstabelle](#12-entscheidungstabelle)
13. [Validierungskriterien pro Phase](#13-validierungskriterien-pro-phase)
14. [Risiken & Stolpersteine](#14-risiken--stolpersteine)
15. [Glossar & Referenzen](#15-glossar--referenzen)
16. [Handoff für künftige Claude-Code-Sessions](#16-handoff-für-künftige-claude-code-sessions)

---

## 1. Zielbild & Nicht-Ziele

### Zielbild

Ein **persönlicher Universal-Assistent** — ein Jarvis im Wortsinn, kein Coding-Bot mit Voice-Aufsatz. Eine Identität, eine fortlaufende Session, voller Anwendungs-Mix. Konkret:

- per **Sprache angesprochen** werden, während ich durch die Wohnung laufe (Haupt-Use-Case)
- über **eine Konversation** Aufgaben verschiedener Tiefe übernehmen — von Mikro ("Licht aus", "Timer 10 Min", "wie spät") über Mittel (E-Mails triagieren, Browser-Recherche, Termine) bis Deep Work (Bachelorarbeit, Coding, mehrstufige Recherchen)
- Zugriff auf meine **Infrastruktur** haben: PC, Dateisystem, GitHub, Browser, Smart Home, eigene Accounts (E-Mail, Kalender)
- **remote** erreichbar sein (unterwegs, Bahn); dort reicht Text-/Diktat-Eingabe, primärer Kanal mobil ist eine **PWA**
- **sichtbar** arbeiten: persistente Sessions, von überall attachbar, optional ein eigener streambarer Desktop für Computer-Use-Tasks
- **mitwachsen**: neue Fähigkeiten kommen als MCP oder parallele Container hinzu, ohne Architektur-Bruch

### Kern-Klarstellung — wer ist "Jarvis"?

Ich rede mit **Claude (dem Modell)**. **Claude Code ist der Harness** — eine persistente Agent-Laufzeit mit MCP-Plugin-System, Permission-Flow, Compaction und Hooks. Das ist kein Coding-Tool, sondern ein Universal-Agent, in dem Coding *einer* der Anwendungsfälle ist. Smart Home, Browser, Recherche, E-Mail laufen über MCPs in derselben Session und teilen sich Kontext. Computer Use (Phase 9) ist ein paralleler Container, den dieselbe Session orchestriert.

Konsequenz: Mikro-Tasks ("Licht aus") laufen bewusst durch dasselbe Modell wie Deep-Work-Tasks. Latenz ~1–2 s und Token-Kosten ~0,5 Cent pro Mikro-Aktion sind der akzeptierte Preis für **eine Identität, einen Kontextfaden**. Ein vorgelagerter Mikro-Router (Hybrid-Architektur) ist eine spätere Optimierung — siehe Section 8 / Phase 10+ —, kein Phase-1-Thema.

### Nicht-Ziele

- Kein Voice-Setup von unterwegs (Text/Diktat reicht)
- Kein Always-On-Listening als Default — Privacy-first, Wake-Word nur auf Wunsch
- Kein Hardware-Blindkauf vor Praxistest
- Kein zweites Orchestrator-LLM vor Phase 7 — eine Identität zuerst, Routing erst, wenn Praxisdaten zeigen, dass es nötig ist
- **Keine Finanz-/Payment-Autonomie** — Geld ausgeben bleibt manuell
- **Kein Ersatz** für IDE, Terminal oder Browser — Jarvis ist Zusatz, nicht Substitut
- **Keine Multi-User-Fähigkeit** — Single-User-System
- **Keine eigene Model-Fine-Tuning-Pipeline** — RAG reicht, Fine-Tuning nur wenn nachweislich nötig

---

## 2. Nutzerprofil & Kontext

- **Nutzer:** Anton Rusik, Bachelorstudent TU Dresden (Computernetzwerke).
- **Tech-Level:** Fortgeschritten. Bachelorarbeit über Cloud-AI-APIs (STT/LLM/TTS) → tiefes Vorwissen zu Latenz-/Protokoll-Themen, hat Erfahrung mit Deepgram, OpenAI, Anthropic SDK, Requesty/SSE.
- **Besondere Präferenz — der Workflow-Treiber:** *"Ich muss mich beim Denken bewegen."* Voice-First ist kein Gimmick, sondern zentral: Ideen kommen beim Rumlaufen, nicht am Schreibtisch. Jarvis muss den Gedanken einfangen können, während die Hände nicht an der Tastatur sind.
- **Primär-Geräte:** Windows-PC (stationär, Hauptarbeitsgerät), Google Pixel 7 Pro (mobil, immer dabei), Laptop (gelegentlich).
- **Privacy-Haltung:** Pragmatisch. Cloud-Anteile erlaubt (STT, LLM), keine paranoide Local-Only-Linie. Wake-Word und Mikro-Always-On sind aber bewusst opt-in.
- **Sekundär-Nutzer (geplant):** MacBook (Chef). Adapter-Architektur muss plattformübergreifend funktionieren.

**Was das für die Architektur heißt:** Jeder Workflow muss einen Voice-Pfad haben (auch wenn parallel ein Text-Pfad existiert). Mobile-First für unterwegs, aber Desktop-PC bleibt das Schwergewicht. Approval-Schwelle bewusst niedrig, weil Vertrauen über Praxis aufgebaut wird, nicht durch Vorab-Konfiguration.

---

## 3. Kern-Workflows

> **Diskussionsanker.** Jeder Workflow ist ein Use-Case-Cluster, kein Skript. Die Top-Prio-Workflows (A/B/D) treiben Phase-1–6-Entscheidungen.

### Workflow A — Coding (Top-Priorität)

**Szenario:** Ich laufe in der Wohnung, überlege ein neues Feature für ein Projekt (Mess-Pipeline, Jarvis selbst, Job-Code).

1. Eingabe via Voice (PTT): *"Jarvis, in der Mess-Pipeline brauchen wir einen zweiten Modus — einen warm, einen kalt, parallel."*
2. Transkription via Cloud-STT, Übergabe an Claude-Code-Session.
3. Claude öffnet das Projekt-Repo via Filesystem-MCP, liest existierenden Code (z. B. `measurements/config.py`), skizziert Implementierungsplan.
4. Plan kommt per TTS oder Text zurück, Diff-Vorschlag im Terminal.
5. Ich segne ab (*"mach das"* / Button), Claude schreibt Code.
6. **Commit braucht explizite Bestätigung** (CLAUDE.md Regel 10), Push erst recht.
7. Session bleibt offen, Kontext erhalten.

**Kanal:** Voice am PC primär, Tastatur optional, Mobile als Read-Only-Status.

### Workflow B — Bachelorarbeit-Support (Top-Priorität)

**Szenario:** Ich denke über ein Thesis-Kapitel nach, brauche Messdaten-Recap oder will einen Abschnitt überarbeiten.

1. Eingabe: *"Jarvis, wie sind die p99-Werte vom STT-Provider im März? Schreib zwei Sätze für Kapitel 4."*
2. Filesystem-MCP aufs `bachelorThesis`-Repo, Aggregation der Messdaten oder Aufruf existierender Analysis-Notebooks.
3. Werte zurück, LaTeX-tauglicher Absatz formuliert.
4. Falls Overleaf-Git-Sync aktiv (siehe Tabelle 12): Absatz als Commit-Vorschlag direkt ins Thesis-Repo. Sonst: Zwischenablage.
5. Mein Approval, Commit läuft.

**Kanal:** Voice oder Text, PC-zentriert.

### Workflow C — Wissensarbeit / Research (Zweite Priorität)

**Szenario:** Ich habe einen Artikel gelesen oder eine Idee und will sie dauerhaft ablegen.

1. Eingabe: *"Jarvis, notier: Medium-Artikel sagt, lokale Voice-Verarbeitung ist zu langsam. Verlinke das mit Jarvis-Architektur-Entscheidung."*
2. Markdown-Notiz wird in den Second-Brain-Vault geschrieben (`references/...md`).
3. Jarvis setzt `[[Wiki-Links]]` zu verwandten Notizen automatisch.
4. Vault committet sich periodisch zu privatem GitHub-Repo (Auto-Sync).

**Kanal:** Voice oder Text, jede Lage.

### Workflow D — Alltag (Mobile, Top-Priorität)

**Szenario:** Ich bin unterwegs mit dem Pixel, habe einen Gedanken oder will eine kurze Aktion.

1. Eingabe via PWA-Voice-PTT (oder Text): *"Jarvis, schreib eine Mail an Betreuer X, dass ich Donnerstag um 14:00 zum Methodik-Gespräch komme."*
2. Jarvis entwirft Mail (Ton-Kalibrierung "formell, Betreuer").
3. **Draft landet in der Action-Queue** — PWA zeigt Preview mit Approve / Reject / Edit.
4. Bei Approve: Senden via Gmail-MCP. Bestätigung als Push.

**Kanal:** Mobile primär. Voice oder Text. Backup: Tastatur-Eingabe.

### Workflow E — Running Companion (Zweite Priorität)

**Szenario:** Ich bin draußen oder in der Wohnung unterwegs, will mit Jarvis "mitdenken".

1. PTT auf PWA-Bildschirm halten (oder Stream-Deck-Pedal später).
2. Jarvis hört zu, antwortet per TTS.
3. Display-Interaktion nicht nötig, Kontext bleibt über die ganze Session erhalten.
4. Notizen, die fallen, landen automatisch im Vault-Inbox-Folder.

**Kanal:** Voice-only, hands-free.

---

## 4. Architektur-Überblick

```
┌────────────────────────────────────────────────────────────────────────┐
│  Windows-Host (nativ, kein WSL für Claude Code)                         │
│                                                                          │
│  ┌──────────────── Schicht 1: Gehirn ─────────────────┐                 │
│  │  Claude Code CLI (nativ)                            │                 │
│  │  - läuft in tmux unter Git Bash                     │                 │
│  │  - persistente Session `jarvis`                     │                 │
│  │  - lädt CLAUDE.md, Custom Slash-Commands            │                 │
│  └──────────────────────┬──────────────────────────────┘                 │
│                         │                                                │
│  ┌──────────── MCP-Server (lokal) ─────────────────────┐                │
│  │  Phase 1: Filesystem · GitHub                       │                 │
│  │  Phase 5: Gmail · Calendar · Vault-Search           │                 │
│  │  Phase 9: Computer-Use · Playwright                 │                 │
│  └─────────────────────────────────────────────────────┘                 │
│                         │                                                │
│  ┌──────── Action-Queue + Approval-Store ──────────────┐                │
│  │  Drafts (Mail, Commit, Kalender-Event)              │                │
│  │  JSON oder SQLite, Policy-Matrix in CLAUDE.md       │                │
│  └─────────────────────────────────────────────────────┘                │
│                         │                                                │
│  ┌──────────── Schicht 3: Eingabe ─────────────────────┐                │
│  │  - `/voice` (PTT, Default)                          │                │
│  │  - openWakeWord + faster-whisper (Modus B)          │                │
│  │  - Unified Remote / Stream Deck Pedal               │                │
│  └─────────────────────────────────────────────────────┘                │
│                                                                          │
│  ┌──────── Schicht 4: Agent-Desktop (ab Phase 9) ──────┐                │
│  │  Docker → anthropic-quickstarts/computer-use-demo   │                │
│  │  Ubuntu 22.04 · xfce · VNC:5900 · noVNC:6080        │                │
│  └─────────────────────────────────────────────────────┘                │
│                                                                          │
│  ┌──────── Second-Brain-Vault (ab Phase 5) ────────────┐                │
│  │  Markdown + Wiki-Links · Auto-Sync zu privatem      │                │
│  │  GitHub-Repo (Backup + Versionierung)               │                │
│  └─────────────────────────────────────────────────────┘                │
└────────────────────────┬─────────────────────────────────────────────────┘
                         │
                    Tailscale (MagicDNS + SSH)
                         │
            ┌────────────┼────────────────┐
            │            │                │
       ┌────▼────┐  ┌────▼────────┐  ┌────▼────┐
       │ Pixel   │  │ Laptop      │  │ TV /    │
       │ PWA     │  │ SSH+tmux    │  │ Browser │
       │ + Voice │  │             │  │ (Agent- │
       │ + Push  │  │             │  │ Desktop)│
       └─────────┘  └─────────────┘  └─────────┘
```

**Zentrale Bausteine:**

1. **Claude Code CLI** als Agent-Kern — Pro-Account, MCP-fähig, in tmux persistent.
2. **MCP-Server** als Fähigkeits-Erweiterung (Filesystem, GitHub, Gmail, Calendar, Vault-Search, Computer-Use, Playwright).
3. **Action-Queue + Approval-Store** als Sicherheits-Schicht — jeder Draft landet hier, Mensch approved per PWA oder Tastatur.
4. **Voice-Layer** als Input/Output-Kanal (Cloud-STT primär, Edge-TTS primary, lokale Fallbacks via Adapter-Pattern).
5. **Second-Brain-Vault** als Langzeit-Kontext (Markdown + Wiki-Links + Git-Sync).
6. **Tailscale** als Netzwerk-Fabric — alles ist von überall erreichbar, nur authentifiziert.
7. **PWA** als Mobile-Frontend — Voice-PTT, Action-Queue-Liste, Approval-UX, Push-Notifications. Native App nur als Notlösung, falls PWA-Limits beißen.

**Datenfluss-Beispiel "In Küche sagen 'commit'":**

1. PTT-Button am Handy gedrückt (Unified Remote oder PWA) → Mic an.
2. Headset-Mic → Cloud-STT (Groq Whisper) → Text.
3. Text in tmux-Session gepiped, Claude Code interpretiert.
4. Claude fasst Aktion zusammen ("ich würde jetzt im Repo X `git add . && git commit -m 'Y'` ausführen — OK?").
5. Bestätigung per Doppel-Klick PTT oder Approval in PWA.
6. Claude führt aus, Output in tmux + Push aufs Handy ("committed").

---

## 5. Eingabe-Modi A & B (PTT ↔ Wake-Word)

Zwei Modi, explizit umschaltbar, **gegenseitig ausschließend**.

### Modus A — Push-to-Talk (Default, Privacy-first)

- Claude Code `/voice` (eingebaut seit März 2026, Mindestversion v2.1.69).
- Leertaste halten → Mic an. Loslassen → Mic aus.
- Trigger-Optionen (aufsteigend nach Aufwand):
  1. **Unified Remote** (Custom-Remote für "Space-Hold") — kostenlos, Handy als Fernbedienung.
  2. **Stream Deck Pedal** (~90 €) — Fußschalter, komfortabelster Weg beim Coden.
  3. **Flic 2 Button** (~35 €) — dezent, überall klebbar.
  4. **PWA-PTT-Button** — direkter Mobile-Pfad, ab Phase 6 verfügbar.
- Kein Always-On, kein Hintergrundprozess, kein Mithören.
- **Einschränkung 1:** `/voice` braucht **Claude.ai-Login**, nicht API-Key/Bedrock/Vertex.
- **Einschränkung 2:** `/voice` läuft **ausschließlich in Claude Code CLI** — nicht in der Claude Mobile App, nicht über SSH/tmux-Pipes vom Handy. Mobile-Voice (Workflow D, E auf Pixel) braucht eine eigene STT-Pipeline (Groq-Streaming-Client, ab Phase 6/7).
- **Einschränkung 3:** Keine `auto`-Spracherkennung — Sprache ist fest wählbar (de/en via `/config`). Einzelne Anglizismen kommen im DE-Modus durch, längere englische Phrasen werden eingedeutscht.

### Modus B — Wake-Word "Hey Claude" (ab Phase 7)

- Lokal, komplett offline:
  - **openWakeWord** als Windows-Service (via NSSM) — ~150 MB RAM, <5 % CPU idle.
  - Custom-"Hey Claude"-Modell, trainierbar in ~1 h mit synthetischen Samples.
  - **faster-whisper** transkribiert den Satz nach Wake-Word-Trigger.
- Transkript wird via `tmux send-keys -t jarvis "<Text>" Enter` in die Claude-Code-Session gepiped.
- Alternative: **Picovoice Porcupine** — robuster bei False-Positives, kostenlos für Personal Use.

### Umschalt-Skript `jarvis-mode.ps1`

```powershell
# Pseudocode — Implementierung in Phase 7
param([ValidateSet('ptt','wake','status')][string]$Mode)

switch ($Mode) {
  'ptt' {
    Stop-Service openwakeword -ErrorAction SilentlyContinue
    Set-TrayIcon -Color Green -Tooltip 'Jarvis: PTT aktiv'
  }
  'wake' {
    Start-Service openwakeword
    Set-TrayIcon -Color Orange -Tooltip 'Jarvis: Wake-Word aktiv'
  }
  'status' { Get-Service openwakeword }
}
```

### Sichtbarkeit

- Tray-Icon-Farbe zeigt aktiven Modus **immer** an.
- Beim Booten: immer PTT (grün). Wake-Word muss explizit gestartet werden.
- **Privacy-Leitlinie:** Wenn unsicher, welcher Modus läuft → `jarvis-mode status` liefert Wahrheit.

---

## 6. Hardware

### Bestehend

- **Windows-PC (Hauptnutzer)** — Hostname: `jarvis`. WoL + Sleep nach 60 Min Inaktivität (S3).
  - CPU: Ryzen 7 5800X (8C/16T, Zen 3)
  - RAM: 32 GB
  - GPU: AMD Radeon RX 6700 XT (12 GB VRAM) — **kein CUDA**, Vulkan-Backend nutzbar
  - OS: Windows 11
  - Storage: NVMe SSD
  - Implikation: Lokale ML-Workloads müssen CPU- oder Vulkan-/DirectML-Pfade nutzen. CUDA-spezifische Optimierungen entfallen.
- **Google Pixel 7 Pro** — Mobile-Hauptgerät (Android 14+).
- **Sekundär-Nutzer (geplant)** — MacBook (Chef). Adapter-Architektur muss plattformübergreifend funktionieren; Apple-Silicon-Backends (Metal in whisper.cpp) sind dort sogar performant.

### Zu beschaffen / entscheiden

| Komponente | Zweck | Optionen | Empfehlung |
|---|---|---|---|
| Bluetooth-Headset mit gutem Mic | Voice-Eingabe beim Rumlaufen | Sony WH-1000XM5, AirPods Pro (2. Gen+), Shokz OpenComm | **Shokz OpenComm** — Open-Ear, Umgebung hörbar, Mic sehr gut für Voice. |
| PTT-Trigger | Hands-free Leertaste | Unified Remote (gratis), Stream Deck Pedal (~90 €), Flic 2 (~35 €) | **Unified Remote zuerst** — null Kosten, testet das Konzept. Bei Komfort-Bedarf auf Pedal aufrüsten. |
| Portable Display (optional) | Monitoring unterwegs im Haus | Gebrauchtes iPad, USB-C-Monitor 14–16" | **Gebrauchtes iPad** — vielseitiger. Später, wenn Need erwiesen. |
| Flic Button (optional) | Schnellstart-SSH von unterwegs | Flic 2 + Hub LR (~80 €) | Erst nach Phase 4, wenn Bedarf spürbar. |

### Bewusst später

- Stream Deck Pedal, Shokz, dediziertes Sub-Agent-Gerät (Mini-PC) → erst nach Phase 7, wenn Voice + Mobile + Wake-Word stabil sind.

---

## 7. Software-Stack (Windows-nativ)

> Alle Kommandos laufen in **PowerShell als Admin** oder **Git Bash**. Kein WSL für Claude Code.

### Auf dem Windows-Host

```powershell
# 1. Git for Windows (bringt Git Bash mit)
winget install --id Git.Git -e

# 2. Node.js LTS
winget install --id OpenJS.NodeJS.LTS -e

# 3. Claude Code nativ
irm https://claude.ai/install.ps1 | iex
claude --version   # muss ≥ 2.1.69 sein

# 4. Tailscale
winget install --id Tailscale.Tailscale -e
# dann: einmal UI öffnen, Account verknüpfen, MagicDNS in Admin-Console aktivieren

# 5. OpenSSH-Server (für mosh)
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Set-Service sshd -StartupType Automatic
Start-Service sshd

# 6. Docker Desktop (für Agent-Desktop ab Phase 9)
winget install --id Docker.DockerDesktop -e

# 7. GitHub CLI
winget install --id GitHub.cli -e
gh auth login

# 8. WoL-Setup
powercfg /change standby-timeout-ac 60
# + BIOS-WoL aktivieren · Fast Startup aus · Netzwerkadapter "nur Magic Packet"
```

### In Git Bash

```bash
# tmux via MSYS2-pacman oder scoop
pacman -S tmux   # falls pacman verfügbar; sonst: scoop install tmux

# Autostart-Shortcut in shell-rc:
echo 'alias jarvis="tmux new-session -A -s jarvis \"claude\""' >> ~/.bashrc
```

### Auf dem Pixel

- **Tailscale** — gleicher Account wie PC.
- **Termux** (von F-Droid, nicht Play Store) — `pkg install openssh mosh tmux`.
- **Termux:Widget** — Homescreen-Shortcuts zur Jarvis-Session.
- **Unified Remote** — als PTT-Trigger.
- **PWA (ab Phase 6)** — als Homescreen-App installiert.
- **Claude Mobile App** — für eigenständige Voice-Mode-Gespräche unterwegs (separater Kontext, nicht die tmux-Session).

### MCP-Startkonfiguration (Phase 1)

- **GitHub MCP** — Bachelorarbeit + Projekte.
- **Filesystem MCP** — zielgerichtet auf Whitelist (`C:\MeineDaten\jarbrew`, BA-Pfad, ggf. `C:\MeineDaten\bachelorThesis`).

---

## 8. Sicherheitsnetz & Action-Queue

Eingaben aus Voice sind fehleranfällig (Transkription, Mehrdeutigkeit). Vier Ebenen, inkrementell aktivierbar.

### Ebene 1 — `CLAUDE.md`-Regeln (sofort, kostenlos)

Siehe [`CLAUDE.md`](./CLAUDE.md) im Repo-Root. Kernregeln:

- Bei abgehackter/mehrdeutiger/wahrscheinlich-transkribierter Eingabe: **präzise Rückfrage** vor Handlung.
- Vor destruktiven Aktionen (löschen, `git push`, Deployments, Paketinstalls): **1-Satz-Zusammenfassung + warten auf "ja"**.
- Bei unklarem Kontext: fragen statt raten.

### Ebene 2 — Action-Queue + Approval-Store (Phase 3)

Strukturierte Persistenz für Drafts, die ein Mensch bestätigen muss, bevor sie ausgeführt werden.

**Komponenten:**
- **Storage:** JSON-Datei (`.jarvis/actions.json` im Vault-Repo). Migration zu SQLite erst bei spürbarer Concurrency.
- **Lebenszyklus:** Draft erstellt → in Queue → Notification (Push/PWA) → Approve/Reject/Edit → bei Approve ausgeführt → Resultat zurück → archiviert. Timeout (z. B. 24 h ohne Approval) → verfällt.
- **UI:** Phase 3 zunächst CLI-Befehle (`jarvis approve <id>`), Phase 6 als PWA-Liste mit Inline-Buttons.

**Action-Policy-Matrix (initial, living):**

| Aktion | Autonom erlaubt? | Approval nötig? | Nie erlaubt? |
|---|---|---|---|
| Datei lesen (lokal) | ✅ | | |
| Datei schreiben im Vault | ✅ | | |
| Datei schreiben in Code-Repo | ✅ | | |
| `git commit` | ❌ | ✅ | |
| `git push` | ❌ | ✅ | |
| PR öffnen/mergen | ❌ | ✅ | |
| Kalender-Event lesen | ✅ | | |
| Kalender-Event anlegen/ändern | ❌ | ✅ | |
| Mail lesen | ✅ | | |
| Mail senden | ❌ | ✅ | |
| Browser-GET | ✅ | | |
| Browser-POST / Formular abschicken | ❌ | ✅ | |
| Paket installieren / Systemänderung | ❌ | ✅ | |
| Smart-Home-Steuerung (lesen) | ✅ | | |
| Smart-Home-Steuerung (schreiben) | ❌ | ✅ | (autonom freischaltbar pro Gerät) |
| Finanzen / Geld ausgeben | ❌ | | ✅ (Phase 1–∞) |

Default-Bias bei Unsicherheit: Approval anfordern. Matrix wird in Phase 3 implementiert und in der Praxis verfeinert (Logging der Approvals/Rejections, nach 2 Wochen Review).

### Ebene 3 — Verifier-Middleware (optional, nach 1–2 Wochen Praxis)

- Python-Skript (~100 Zeilen).
- Transkript → Claude Haiku Prompt: `READY: <bereinigter Text>` oder `CLARIFY: <Rückfrage>`.
- `READY` → via `tmux send-keys` in Claude-Code-Session.
- `CLARIFY` → per TTS vorlesen, Antwort holen, Loop.
- Kosten: < 1 Cent pro Anfrage, Latenz ~200 ms.
- Entscheidung: bauen, wenn Ebene 1/2 in der Praxis durchgelassen hat, was sie nicht hätte durchlassen sollen. Auch der natürliche Andockpunkt für einen späteren Hybrid-Mikro-Router.

### Ebene 4 — `voice-mode` MCP mit 2-Wege-Dialog (Phase 7)

- Claude antwortet gesprochen, Rückfragen werden natürlich.
- Lokaler Stack via Adapter-Pattern, primary bleibt Cloud (Groq + Edge-TTS).

### Ebene 5 — Container-Isolation (Agent-Desktop, Phase 9+)

- Computer-Use läuft im Docker-Container, isoliert vom Host-Filesystem.
- Nur explizit gemountete Volumes sind schreibbar.
- Reboot = Reset (optional via `readonly-rootfs + tmpfs`).

---

## 9. Phasenplan 0–9 (MVP)

### Phase 0 — Vorbereitung (~15 Min)

- [ ] Windows-Version prüfen (Windows 11).
- [ ] WoL-Voraussetzungen: BIOS-WoL aktiv, Fast Startup aus, Netzwerkadapter "nur Magic Packet" erlaubt, `powercfg /change standby-timeout-ac 60`.
- [ ] PowerShell Execution Policy = RemoteSigned.
- [ ] Hostname auf `jarvis` gesetzt (Reboot zum Wirksamwerden).

### Phase 1 — Grundsetup (~60 Min)

- [ ] Tailscale auf PC + Pixel, MagicDNS aktivieren, Verbindung testen (`ping jarvis.<tail-id>.ts.net`).
- [ ] Claude Code installieren, `claude --version` → ≥ 2.1.69. Login mit Pro-Account (kein API-Key).
- [ ] OpenSSH-Server aktivieren, Tailscale-SSH ausprobieren.
- [ ] tmux installieren, `tmux new-session -A -s jarvis` in Git Bash, `claude` starten, `Ctrl+b d` zum Detachen testen.
- [ ] `gh auth login`, GitHub-MCP + Filesystem-MCP konfigurieren (Filesystem mit Whitelist, kein Home/Root).
- [ ] **Definition of Done:** Workflow A (Coding) läuft per Text vom Pixel via Termux.

### Phase 2 — Voice zuhause, PTT-Modus (~45 Min)

- [ ] Bluetooth-Headset mit PC koppeln, als Standard-Input setzen, HFP-Codec auf mSBC (oder höher) zwingen.
- [ ] In Claude Code `/voice` testen (Leertaste halten, sprechen).
- [ ] Falls `/voice`-Qualität OK: fertig. Falls nicht: Groq-Whisper-Pipeline aus `STT_TTS_OPTIONS.md` als eigenen Client bauen.
- [ ] Edge-TTS-Stimme wählen (Killian / Katja / Conrad probehören), Test-Output am PC.
- [ ] Unified Remote PC-Server + Pixel-Client installieren, Custom-Remote "PTT Space Hold" bauen.
- [ ] Realtest: durch Wohnung laufen, Mic-Reichweite prüfen.
- [ ] **Definition of Done:** Sprachbefehl aus Küche klappt stabil, Workflow A läuft per Voice am PC.

### Phase 3 — Sicherheitsnetz Ebene 1+2 + Custom Slash-Commands (~90 Min)

- [ ] `CLAUDE.md` reviewen, Action-Policy-Matrix (Section 8 Ebene 2) als zweiten Block ergänzen.
- [ ] Action-Queue als JSON-Datei anlegen (`.jarvis/actions.json` im Vault, sobald Phase 5 da; vorerst lokaler Pfad).
- [ ] CLI-Befehle für Queue: `jarvis list`, `jarvis approve <id>`, `jarvis reject <id>`, `jarvis edit <id>`.
- [ ] Custom Slash-Commands in `.claude/commands/` anlegen — vorgeschlagen: `/note` (Notiz im Vault), `/morning` (Tagesüberblick), `/approve` (Queue-Item bestätigen), `/draft-mail` (Mail-Entwurf), `/journal` (Tageseintrag).
- [ ] Claude-Code-Permission-Prompts strikt einstellen (nicht alles autoapproven).
- [ ] Test: vager Befehl → Rückfrage. Destruktiver Befehl → 1-Satz-Zusammenfassung + Warten. Mail-Draft → landet in Queue.

### Phase 4 — Remote-Zugang (~30 Min)

- [ ] Termux auf Pixel (F-Droid), `pkg install openssh mosh tmux`.
- [ ] SSH-Profil mit Tailscale-Hostname `jarvis.<tail-id>.ts.net`, Key-Auth einrichten.
- [ ] Startup-Command `mosh jarvis.<tail-id>.ts.net -- tmux new-session -A -s jarvis`.
- [ ] Termux:Widget-Shortcut auf Homescreen.
- [ ] Test aus mobilem Netz (nicht Heim-WLAN): attachen, Aktion absetzen, Netzwechsel überstehen.

### Phase 5 — Kontext-Layer (Vault + Mail + Kalender) (~2 h)

- [ ] `~/second-brain/`-Repo (oder `C:\MeineDaten\second-brain\`) anlegen, zu privatem GitHub pushen, Auto-Sync via Cron oder Git-Hook.
- [ ] Vault-Struktur: `daily/`, `references/`, `projects/`, `inbox/` — Details in `second-brain/README.md`.
- [ ] Gmail-MCP einrichten (OAuth via Google), Test: Mail lesen, Draft erstellen → in Action-Queue.
- [ ] Google-Calendar-MCP einrichten, Test: Events lesen, Anlegen via Approval.
- [ ] Filesystem-MCP-Whitelist um Vault-Pfad erweitern.
- [ ] **Definition of Done:** Workflow B + C funktionieren mit Vault-Persistenz; Workflow D (Mail) end-to-end mit Approval.

### Phase 6 — Mobile-Frontend (PWA) (~3–4 h verteilt)

- [ ] PWA-Gerüst aufsetzen — Framework-Wahl in dieser Phase (Svelte+Vite vorgeschlagen, weil schlank).
- [ ] PWA-Features: Voice-PTT-Button, Action-Queue-Liste mit Approve/Reject/Edit-Buttons, Push-Notifications via Web-Push API.
- [ ] **PWA braucht eigene STT-Pipeline** — `/voice` ist Claude-Code-CLI-only, funktioniert nicht in der PWA. Lösung: Groq-Whisper-Streaming-Client (Python oder Node) auf dem PC, PWA sendet Audio via WebSocket, bekommt Transkript zurück. Dieser Client ist dann auch der Grundstein für Phase 7 (Wake-Word).
- [ ] Backend: minimal — PWA spricht via Tailscale mit einer kleinen API auf dem PC, die STT, Action-Queue und Push orchestriert.
- [ ] Installation auf Pixel als Homescreen-App.
- [ ] **Definition of Done:** Workflow D (Mobile) fühlt sich wie eine echte App an. Native App nur, falls PWA-Limits beißen.

### Phase 7 — Zwei-Wege-Sprache + Wake-Word (~90 Min)

- [ ] voice-mode MCP installieren:
  ```powershell
  irm https://astral.sh/uv/install.ps1 | iex
  claude mcp add --scope user voicemode -- uvx --refresh --with webrtcvad --with "setuptools<71" voice-mode
  ```
- [ ] STT/TTS-Adapter prüfen — Primary bleibt Groq + Edge-TTS, lokale Fallbacks (faster-whisper, Piper) optional.
- [ ] openWakeWord installieren, Custom-"Hey Claude"-Modell trainieren oder Picovoice Porcupine nutzen.
- [ ] `jarvis-mode.ps1` schreiben (PTT/Wake-Toggle), in `C:\Tools\` ablegen, PATH setzen.
- [ ] Tray-Icon-Indikator (BurntToast oder AutoHotkey).
- [ ] **Definition of Done:** "Hey Claude, was ist die Uhrzeit" → antwortet gesprochen.

### Phase 8 — Proaktivität (~2 h)

- [ ] Scheduler-Job: **Morning-Briefing** 08:00 — Kalender des Tages + Wetter + offene Action-Queue + 1 Status-Zeile pro aktivem Projekt.
- [ ] Scheduler-Job: **Evening-Recap** 22:00 — Was wurde heute committed? Was steht morgen an?
- [ ] Event-getriggert: Neue Mail von definierten Absendern (Betreuer, Job) → sofort Push.
- [ ] Event-getriggert: 72 h keine Thesis-Commits → freundlicher Reminder.
- [ ] Implementation: Windows Task Scheduler oder Python APScheduler — Entscheidung in dieser Phase.
- [ ] **Definition of Done:** mindestens 3 proaktive Jobs laufen zuverlässig und nerven nicht.

### Phase 9 — Agent-Desktop / Computer-Use (~2–3 h)

- [ ] Docker-Container basierend auf `anthropic-quickstarts/computer-use-demo`, Volumes auf Projekt-Ordner.
- [ ] Streaming: noVNC auf Port 6080, erreichbar via Tailscale auf Handy/TV/iPad-Browser.
- [ ] Claude-Code-Session orchestriert den Container (Computer-Use-MCP).
- [ ] Filesystem-Sandbox: nur explizit gemountete Volumes schreibbar.
- [ ] **Definition of Done:** Browser auf `http://jarvis.<tail-id>.ts.net:6080` zeigt Live-Desktop, Claude öffnet per Prompt eine Website, ich sehe live zu.

---

## 10. MCP-Roadmap

**Start-Set (Phase 1):**

- GitHub MCP
- Filesystem MCP — mit Whitelist (Repo-Pfad, BA-Pfad, später Vault)

**Phase 5 (Kontext-Layer):**

- Gmail MCP
- Google-Calendar MCP
- Vault-Search MCP (initial: einfache Filesystem-Suche; RAG-Upgrade in Roadmap)

**Phase 9 (Agent-Desktop):**

- Computer-Use MCP
- Playwright MCP — für Web-Automation, sobald Browser-Tasks häufig werden

**Erweiterungs-Trigger:**

| Erweiterung | Wann installieren |
|---|---|
| Obsidian MCP | Falls Obsidian als Vault-Reader gewünscht |
| Jira / Confluence MCP | Wenn Job-Kontext mit Jira startet |
| Home Assistant MCP | Mit Smart-Home-Phase (Roadmap) |
| RAG-MCP (LanceDB / Chroma) | Wenn Vault-Volumen Filesystem-Suche überfordert |

---

## 11. Zukunftsroadmap (Phasen 10+)

> "Living Roadmap" — modular, jede Phase für sich aktivierbar.

### Phase 10 — Multi-Agent via Conductor · ~30 Min

- **Bedarf:** Parallel arbeitende Agents (Bachelorarbeit / Job / Recherche).
- **Skizze:** Conductor-Workspaces pro Kontext, Claude Code startet pro Workspace eine dedizierte tmux-Session.
- **Entry:** Spürbarer Bedarf nach Parallelität.

### Phase 11 — Smart-Home (Home Assistant MCP)

- **Bedarf:** "Jarvis, dim das Licht im Büro."
- **Skizze:** Home-Assistant-Instanz (Container oder HAOS), HA-MCP-Server, Zigbee-/Matter-Adapter nach Bedarf.
- **Entry:** Smart-Home-Hardware vorhanden.

### Phase 12 — RAG über Bachelorarbeit + Notizen

- **Skizze:** LanceDB oder Chroma lokal, Embeddings via lokales Modell (`sentence-transformers`), Vault-Search-MCP als RAG-Adapter.
- **Entry:** Vault-Volumen jenseits von "alles in den Kontext kippen".

### Phase 13 — Lernende Guardrails

- Anton approved/rejectet oft → Jarvis lernt Muster und fragt bei offensichtlichen Fällen nicht mehr.
- Kombination aus Logging + periodischem Re-Review der Action-Policy-Matrix.

### Phase 14 — Telefonie / SMS-Filter (spekulativ)

- VoIP-Setup, eingehende Anrufe filtern, Transkription + Summary.

### Phase 15 — Dedizierter Mini-PC als Jarvis-Host

- **Skizze:** Raspberry Pi 5 oder NUC, 24/7, Hauptrechner bleibt reine Workstation.
- **Entry:** Hauptrechner zu beschäftigt / zu viel Ressourcenverbrauch durch Jarvis.

---

## 12. Entscheidungstabelle

> Default-Vorschläge mit ✓/✗-Spalte für späteres Abhaken.

| # | Frage | Default | ✓/✗ | Notiz |
|---|---|---|---|---|
| 1 | Heimrechner-OS | Windows nativ | ✓ | User bestätigt |
| 2 | Betriebsmodus | WoL + Sleep nach 60 Min Inaktivität (S3) | ✓ | Aktive Sessions nicht abwürgen, nachts Strom sparen. Phase-0-Setup: BIOS-WoL, "nur Magic Packet"-Wake, Fast Startup aus, `powercfg /change standby-timeout-ac 60` |
| 3 | Bluetooth-Headset | Shokz OpenComm | — | Alternativ Sony WH-1000XM5. Erst nach Phase 2 kaufen, wenn Voice mit vorhandener Hardware getestet |
| 4 | PTT-Trigger erste Iteration | Unified Remote | — | Upgrade auf Stream Deck Pedal nur bei Komfort-Bedarf |
| 5 | Bachelorarbeit-Tool | LaTeX (Overleaf) + Git-Sync | ✓ | Filesystem-MCP aufs lokale Overleaf-Clone. Pfad bei Implementierung Phase 1 entschieden |
| 6 | Verifier-Ebene 3 bauen | Erst nach 1–2 Wochen Praxis | — | Datenbasis vor Implementation |
| 7 | Portable Display | Später — gebrauchtes iPad | — | Nach Phase 4 re-evaluieren |
| 8 | STT-Anbieter (Primary) | Groq Whisper Large v3 Turbo | ✓ | ~$1.80/Monat bei 45 h. `language="auto"` Pflicht — gilt für Groq-Pipeline (Phase 7+), **nicht** für `/voice` (hat kein `auto`, nur feste Sprachen). Details: `STT_TTS_OPTIONS.md` |
| 8a | TTS-Anbieter (Primary) | Microsoft Edge-TTS | ✓ | gratis, deutsche Neural-Stimmen. Stimme zu wählen (Killian/Katja/Conrad) in Phase 2 |
| 8b | STT-Lokal-Fallback | faster-whisper `medium` (CPU) oder whisper.cpp + Vulkan | ✓ | CPU-Variante einfach, Vulkan performant aber Setup-Aufwand. AMD-GPU = kein CUDA |
| 8c | TTS-Lokal-Fallback | Piper | ✓ | CPU, ~5–10× Realtime, läuft Win + Mac |
| 8d | STT/TTS-Architektur | Adapter-Pattern, Backends austauschbar | ✓ | Mac-Nutzer-Kompatibilität; Cloud-Backup-Slots: Deepgram, Azure Neural, Google Neural2 |
| 8e | Verworfene STT/TTS-Anbieter | OpenAI (STT+TTS), AWS Polly/Transcribe, ElevenLabs, EC2-Self-Hosting | ✓ | Begründung dokumentiert in `STT_TTS_OPTIONS.md` |
| 9 | Wake-Word-Engine | openWakeWord | — | Picovoice Porcupine als Backup bei False-Positives |
| 10 | Agent-Desktop-Backend | Docker-Container | — | Keine WSL2-Abhängigkeit für Claude Code, Docker-Desktop ok |
| 11 | Architektur-Kern | Claude Code als Universal-Harness, kein Custom-Orchestrator | ✓ | Eine Identität, ein Kontextfaden über Mikro-/Mid-/Deep-Tasks. Mikro-Tasks bewusst durchs Hauptmodell (~1–2 s, ~0,5 ct). Hybrid-Router (Haiku-Dispatcher) ist Phase-10+-Re-Eval. Begründung: Section 1 "Kern-Klarstellung" |
| 12 | Mobile-Frontend | PWA (Voice-PTT, Action-Queue, Push) | ✓ | Native App nur als Notlösung, falls PWA-Limits beißen. Framework (Svelte/React/Solid) Phase-6-Entscheidung. **Telegram-MVP verworfen** — direktes PWA-Ziel |
| 13 | Action-Queue-Storage | JSON-Datei (`.jarvis/actions.json` im Vault) | ✓ | Migration zu SQLite erst bei spürbarer Concurrency |
| 14 | Second-Brain-Vault | Markdown + Wiki-Links + Auto-Sync zu privatem GitHub | ✓ | Obsidian-kompatibel ohne Migration. Strukturordner: `daily/`, `references/`, `projects/`, `inbox/` |
| 15 | Phasen-Reihenfolge: Voice vor Kontext-Layer | Voice in Phase 2, Kontext-Layer in Phase 5 | ✓ | Bewusste Abweichung von Kalashnikov-Medium-Artikel ("Kontext zuerst"). Begründung: Voice ist Workflow-Treiber für den User ("muss mich beim Denken bewegen"). Ohne funktionierenden Voice-Pfad ist der Rest Demo-Material |
| 16 | Mail/Kalender-Approval | Senden/Anlegen brauchen Approval, Lesen nicht | ✓ | Action-Policy-Matrix in Section 8. Lebende Liste, wird in Praxis verfeinert |
| 17 | Smart-Home im Phase-1–9-Scope | nein | ✓ | Phase 11 Roadmap, nicht Kern |
| 18 | Fine-Tuning eigener Modelle | nein | ✓ | RAG reicht. Fine-Tuning nur bei nachweisbarem Bedarf |
| 19 | Multi-User | nein | ✓ | Single-User-System für Anton |

---

## 13. Validierungskriterien pro Phase

- **Phase 1:** Vom Pixel via Tailscale in `jarvis`-tmux einloggen, Claude antwortet auf "hi", Detach/Attach klappt.
- **Phase 2:** Durch Wohnung laufen, Befehl absetzen, Claude transkribiert korrekt und reagiert.
- **Phase 3:** Vager Befehl → Rückfrage. Destruktiver Befehl → 1-Satz-Zusammenfassung + warten. Mail-Draft → in Action-Queue → CLI-Approve → Versand.
- **Phase 4:** Aus Café/LTE attachen und Aktion laufen lassen; Netzwechsel im Zug übersteht mosh.
- **Phase 5:** Notiz per Voice landet im Vault, ist auf GitHub nach Auto-Sync sichtbar. Kalender-Event wird per Approval angelegt.
- **Phase 6:** PWA auf Pixel installiert, Voice-PTT funktioniert, Action-Queue zeigt offene Drafts mit Approve/Reject-Buttons, Push kommt an.
- **Phase 7:** "Hey Claude, was ist die Uhrzeit" → gesprochene Antwort. Umschalter PTT↔Wake-Word funktioniert, Tray-Icon ändert Farbe.
- **Phase 8:** Morning-Briefing-Push kommt 08:00 zuverlässig. Mindestens ein Event-Trigger (Mail von Betreuer) feuert in Praxis.
- **Phase 9:** Browser auf `http://jarvis.<tail-id>.ts.net:6080` zeigt Live-Desktop, Claude öffnet per Prompt eine Website, ich sehe live zu.

---

## 14. Risiken & Stolpersteine

1. **`/voice` in tmux unzuverlässig** → Fallback: Voice-Pipeline direkt in PowerShell, tmux nur für Text-Session. Eigene Groq-Whisper-Pipeline parallel testen.
2. **Tailscale-SSH-Konflikt mit OpenSSH-Server** → Plan: beide aktivieren, Tailscale regelt Access. Falls mosh nötig, OpenSSH ohnehin Pflicht.
3. **Bluetooth-Mic unter Windows (HFP-Codec)** → Workaround: manuell mSBC oder höher erzwingen, ggf. Kabel-Headset.
4. **STT-Cloud-Kosten laufen aus dem Ruder** → Monitoring-Script in Phase 2, Budget-Alarm bei >10 €/Monat.
5. **Approval-Müdigkeit** → User approved zu schnell → Policy zu großzügig → Unfall. **Gegenmaßnahme:** Phase-3-Logging der Approvals/Rejections, nach 2 Wochen Review der Policy. Default-Bias: bei Zweifel mehr Approval, nicht weniger.
6. **Action-Queue-Backlog** → Drafts häufen sich, weil Approval umständlich → Frustration → User schaltet alles auf autonom. **Gegenmaßnahme:** Approval-UX in PWA muss sub-2-Sekunden sein, sonst nicht produktiv.
7. **Vault-Drift / RAG-Qualitätsverlust** → Vault wird groß → Filesystem-Suche reicht nicht mehr → Re-Indexing-Job (Phase 12).
8. **Claude Code Pro-Limits** → bei Überschreitung API-Fallback-Key bereitlegen. Auch: `/voice` an Pro-Login gebunden.
9. **PWA-Push auf iOS** — falls Chef-Mac-Use-Case mit iPhone aufkommt: iOS unterstützt Web-Push erst eingeschränkt. Fallback: native App-Wrapper (Capacitor/Tauri) als Notlösung.
10. **Edge-TTS API-Abkündigung** — inoffiziell, Microsoft kann jederzeit dichtmachen. Adapter-Pattern + Piper-Fallback puffern das.
11. **`/voice`-Code-Switching-Limitation** — `/voice` hat kein `auto`, Sprache ist fest. Im DE-Modus kommen kurze Anglizismen durch, längere englische Phrasen werden eingedeutscht. Workaround Phase 2: Sprache auf `de` lassen, bei stark englischem Kontext temporär auf `en` wechseln (`/config`). Dauerlösung: eigene Groq-Pipeline ab Phase 7 mit `language="auto"`.
12. **`/voice` nur auf Claude Code CLI** — Mobile-Sessions (Pixel/Termux/SSH) können `/voice` nicht nutzen. Bahn-Remote ist daher immer Text/Diktat. PWA braucht eigene STT (Phase 6). Kein workaround, by design.

---

## 15. Glossar & Referenzen

**Begriffe:**

- **PTT** — Push-to-Talk, Mic nur aktiv wenn Knopf gehalten.
- **MCP** — Model Context Protocol, standardisierter Weg, Tools an Claude anzubinden.
- **tmux** — Terminal-Multiplexer, erhält Sessions über Disconnects.
- **mosh** — Mobile Shell, übersteht Netzwechsel, was SSH nicht tut.
- **MagicDNS** — Tailscale-Feature, vergibt DNS-Namen pro Gerät.
- **PWA** — Progressive Web App, installierbar als Homescreen-App, Push-fähig (Android voll, iOS eingeschränkt).
- **WoL** — Wake-on-LAN, PC per Magic Packet aus Sleep wecken.

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
- MCP-Doku: https://modelcontextprotocol.io
- Tailscale: https://tailscale.com/kb
- Inspirations-Artikel (Kalashnikov, Medium) — Kontext-First-Sicht, von der diese PRP bewusst abweicht (siehe Tabelle 12 #15): https://kalashnikovisme.medium.com/how-to-start-building-your-own-jarvis-almost-everyone-does-it-wrong-daac4a9ee1fe

---

## 16. Handoff für künftige Claude-Code-Sessions

**Wenn du diese PRP frisch liest und arbeiten sollst, mach in dieser Reihenfolge:**

1. **Lies `CLAUDE.md` im Repo-Root.** Voice-Sicherheitsregeln + Action-Policy-Matrix-Verweis.
2. **Finde die aktuelle Phase.** Schau in Section 9, welche Checkbox als Nächstes offen ist, oder frag den User.
3. **Prüfe offene Entscheidungen in Tabelle 12.** Falls noch ✗ oder "—" bei einer Phase-relevanten Zeile: frag den User, bevor du installierst.
4. **Halte die PRP aktuell.** Wenn du einen Schritt abschließt, hake die Checkbox ab und committe (nur wenn der User das wünscht).
5. **Niemals ohne Rückfrage installieren, pushen, löschen.** CLAUDE.md-Regel Ebene 1 gilt. Mail/Kalender/Commits brauchen Action-Queue-Approval (Section 8 Ebene 2).

**Was bereits fest steht:**

- OS = Windows nativ, Hostname = `jarvis`, Betriebsmodus = WoL + 60-Min-Sleep.
- Bachelorarbeit = LaTeX/Overleaf, Overleaf-Git-Sync ja, lokaler Pfad in Phase 1 entschieden.
- STT = Groq Whisper Large v3 Turbo (Cloud), TTS = Microsoft Edge-TTS (Cloud, gratis). Lokale Fallbacks via Adapter (faster-whisper / whisper.cpp + Piper).
- PTT ist Default, Wake-Word opt-in ab Phase 7.
- Mobile-Frontend = PWA, kein Telegram-MVP.
- Architektur-Kern = Claude Code als Universal-Harness.
- Action-Queue + Approval-Policy = Phase 3.
- Vault = Markdown + GitHub-Auto-Sync, ab Phase 5.

**Was als Nächstes ansteht:** Phase 1 (Grundsetup) — siehe Section 9, sobald die offenen Punkte aus `OPEN_QUESTIONS.md` abgearbeitet sind.
