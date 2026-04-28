# Handoff — Nächste Session

> **Erstellt:** 2026-04-28 nach ~3h Planungs-Session.
> **Zweck:** Kontext für den nächsten Chat, damit ohne Einlesen weitergearbeitet werden kann.

---

## TL;DR — Was steht als Nächstes an?

1. **Committen + Pushen** — alle Änderungen dieser Session sind noch nicht in Git. Dann auf `asphyxx24/jarbrew` pushen.
2. **Phase 0 abschließen** — Hostname-Reboot, BIOS-WoL aktivieren, Fast Startup aus, `powercfg /change standby-timeout-ac 60`.
3. **Phase 1 starten** — Tailscale auf PC + Pixel, Claude Code install prüfen, OpenSSH, tmux, GitHub-MCP + Filesystem-MCP.
4. **`/voice` läuft bereits** (heute getestet, funktioniert gut) — kann also direkt in Phase 2 übergehen, sobald Phase 1 steht.
5. **Alle OPEN_QUESTIONS abgearbeitet** — `/voice` heute getestet ✓, funktioniert. Keine blockierenden offenen Fragen für Phase 0/1.

---

## Was diese Session passiert ist

### Repo geklont
- `papageiAI/jarbrew` geklont nach `C:\MeineDaten\jarbrew\` (Heim-PC ist angekommen).
- Kanonischer Owner ist `asphyxx24/jarbrew` — `papageiAI` ist Zweitaccount.

### Architekturdiskussion
- Frage "Was ist der Kern von Jarvis?" geklärt: **Claude (Modell)** ist Jarvis, **Claude Code** ist der Harness — Universal-Agent-Laufzeit, kein Coding-Tool.
- Kein Custom-Orchestrator, kein Dispatcher-LLM vor Phase 7. Eine Identität, ein Kontextfaden.
- Hybrid-Router als Phase-10+-Re-Evaluation offen gelassen (Tabelle 12 #11).

### PRP-Refactor (v0.3)
Die `JARVIS_PRP.md` wurde komplett überarbeitet (v0.2 → v0.3). Neu/geändert:
- **Owner auf `asphyxx24`** überall.
- **Section 2 NEU:** Nutzerprofil & Kontext ("muss mich beim Denken bewegen" als zentraler Workflow-Treiber).
- **Section 3 neu:** 5 Workflows A–E (A=Coding, B=BA, C=Research, D=Mobile/Mail, E=Running), Top-Prio A/B/D.
- **Section 4:** Architektur um Action-Queue, Second-Brain-Vault, PWA-Layer erweitert.
- **Section 8:** Sicherheitsnetz auf 5 Ebenen erweitert, **Ebene 2 = Action-Queue + Approval-Policy-Matrix** (16 Aktionsklassen).
- **Section 9:** Phasen 0–9 (war 0–5): Phase 3 = Action-Queue + Custom Slash-Commands, Phase 5 = Kontext-Layer (Vault/Gmail/Cal), Phase 6 = PWA, Phase 7 = Wake-Word, Phase 8 = Proaktivität, Phase 9 = Agent-Desktop.
- **Section 14 NEU:** Risiken & Stolpersteine (12 Punkte).
- **Tabelle 12** um 8 neue Einträge erweitert (#12–#19).

### Praxis-Findings aus `/voice`-Test
- `/voice` **funktioniert** — Transkription gut, Anglizismen kommen durch.
- **Kein `auto`-Sprachmodus** bei `/voice` (gilt nur für Groq-Pipeline). Feste Sprache (de/en via `/config`).
- **`/voice` nur in Claude Code CLI** — nicht in Mobile App, nicht über SSH/tmux. Mobile-Voice braucht eigene STT-Pipeline (Phase 6).
- → In PRP Section 5, Tabelle 12 #8 und Section 14 Risiken #11/#12 nachgepflegt.

### OPEN_QUESTIONS — alle Punkte abgearbeitet

| # | Frage | Status |
|---|---|---|
| 1 | Tailscale-Account | ✅ vorhanden |
| 2 | PC-Hostname | ✅ `jarvis` (Reboot ausstehend) |
| 3 | Betriebsmodus | ✅ WoL + 60-Min-Sleep (S3) |
| 4 | BA/Overleaf-Integration | ✅ Overleaf-Git-Sync ja, Pfad in Phase 1 |
| 5 | Bluetooth-Headset | ✅ Phase 2 mit Recon 70 AR (Kabel), BT-Eskalation danach |
| 6 | PTT-Trigger | ✅ Unified Remote als Start |
| 7 | Edge-TTS-Stimme | → Phase 2 (Probehören) |
| 8 | Adapter-Interface | → Phase 2 (entsteht entlang Code) |
| 9 | Wake-Word-Engine | → Phase 7 (Default: openWakeWord) |

---

## Aktueller Git-Status

```
Nicht committed (Working Tree dirty):
  modified:   HANDOFF.md       (Archiv-Header ergänzt)
  modified:   JARVIS_PRP.md    (v0.3 Refactor + /voice-Findings)
  modified:   OPEN_QUESTIONS.md (alle Fragen abgearbeitet, /voice ✓)

Nicht in Repo (untracked):
  neu:        NEXT_SESSION.md  (diese Datei)

Commits die gepusht werden müssen:
  16ec2d5  Reframe PRP scope: universal assistant, not coding-focused (bereits auf papageiAI remote)
  + alle Working-Tree-Änderungen → noch in einem Commit zusammenfassen
```

**Empfohlener nächster Commit:**
```
git add .
git commit -m "PRP v0.3: universal harness refactor, phases 0-9, action queue, vault, PWA, voice findings"
```

**Dann pushen** (auf `asphyxx24/jarbrew` — Remote muss ggf. auf asphyxx24 umgestellt werden):
```bash
git remote set-url origin https://github.com/asphyxx24/jarbrew.git
git push
```

---

## Wichtige Entscheidungen — nie rückwärts ohne Anlass

| Entscheidung | Tabelle 12 # |
|---|---|
| Claude Code als Universal-Harness | #11 |
| WoL + 60-Min-Sleep | #2 |
| STT = Groq Whisper Large v3 Turbo (Cloud) | #8 |
| TTS = Edge-TTS (Cloud, gratis) | #8a |
| Mobile-Frontend = PWA (kein Telegram) | #12 |
| Action-Queue + Approval (Phase 3) | #13, #16 |
| Second-Brain-Vault = Markdown + GitHub-Sync | #14 |
| Voice vor Kontext-Layer (Phase 2 vor Phase 5) | #15 |
| Smart-Home, Fine-Tuning, Multi-User: out of scope | #17–19 |
| `/voice` kein `auto`, nur Claude-Code-CLI | Section 5, Risiken #11/#12 |

---

## Prompt zum Starten der nächsten Session

```
Lies JARVIS_PRP.md (v0.3), CLAUDE.md und NEXT_SESSION.md.

Kurzer Stand: Planungsphase abgeschlossen, alle Entscheidungen getroffen (OPEN_QUESTIONS.md). 
Git-Änderungen müssen noch committed + auf asphyxx24/jarbrew gepusht werden.
Danach: Phase 0 (Hostname-Reboot, WoL/BIOS, Power-Settings) → Phase 1 (Tailscale, Claude Code, tmux, MCPs).
/voice wurde heute getestet — funktioniert. Kann in Phase 2 direkt eingesetzt werden.

Was als Erstes ansteht: commit + push, dann Phase 0.
```
