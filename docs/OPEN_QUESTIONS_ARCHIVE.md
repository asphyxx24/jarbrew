# Offene Punkte vor Heim-Start

> Stand: 2026-04-28 (zuletzt aktualisiert nach PRP-Refactor v0.3 — Universal-Harness-Reframing, Action-Queue, Vault, PWA, neue Phasenstruktur). Sortiert nach Dringlichkeit. Sobald ein Punkt entschieden ist, abhaken und Begründung in `JARVIS_PRP.md` Tabelle 12 oder `STT_TTS_OPTIONS.md` nachpflegen.

---

## Bereits geklärt in dieser Session

- ✅ **Architektur-Kern: Claude Code als Universal-Harness.** Kein zweites Orchestrator-LLM vor Phase 7. Mikro-Tasks ("Licht aus") laufen bewusst durchs Hauptmodell. Begründung: PRP Section 1 "Kern-Klarstellung" + Tabelle 12 #11.
- ✅ **HANDOFF.md archiviert** (Header-Hinweis ergänzt).
- ✅ **Mobile-Frontend = PWA**, kein Telegram-MVP. Native App nur als Notlösung. Tabelle 12 #12.
- ✅ **Action-Queue + Approval-Matrix als Sicherheits-Ebene 2** in Phase 3. Tabelle 12 #13, #16.
- ✅ **Second-Brain-Vault** (Markdown + Wiki-Links + GitHub-Auto-Sync) in Phase 5. Tabelle 12 #14.
- ✅ **Phasenplan auf 0–9 erweitert** (Repo-Voice-First-Logik + geteilte Kontext-Layer-/Mobile-/Proaktivität-Phasen). Begründung der Reihenfolge: Tabelle 12 #15.
- ✅ **Smart Home, Fine-Tuning, Multi-User**: explizit aus dem Phase-1–9-Scope. Tabelle 12 #17–#19.
- ✅ **Repo-Owner: `asphyxx24`** — `papageiAI` ist Zweitaccount. Push erst, wenn alles fertig.

---

## Vor Phase 1 sinnvoll zu klären

- [x] **Tailscale-Account** — bereits vorhanden ✓
- [x] **PC-Hostname festgelegt: `jarvis`** ✓ — Reboot zum Wirksamwerden noch ausstehend.
- [x] **Betriebsmodus: WoL mit 60-Min-Sleep-Timer (S3)** ✓
  - Phase-0-Setup-Aufgaben: BIOS-WoL · Netzwerkadapter „nur Magic Packet" · Fast Startup aus · `powercfg /change standby-timeout-ac 60` · WoL-App auf Pixel.
- [x] **Bachelorarbeit-Integration: Overleaf-Git-Sync ja** ✓
  - Lokaler Clone-Pfad bei Implementierung in Phase 1 entschieden.
  - Filesystem-MCP-Whitelist dann anpassen.

---

## Hardware-Entscheidungen (vor Phase 2)

- [x] **Headset Phase-2-Start: Recon 70 AR per Kabel** ✓
  - Erstmal prüfen, ob das Mic für Whisper reicht. Kein Vorab-Kauf.
  - **Phase 2b (Reichweite-Frage):** falls Voice am Schreibtisch klappt, dann entscheiden zwischen (a) BT-TX/RX-Adapter mit explizitem Mic-Pfad-Support ~30–40 €, (b) echtes BT-Headset wie Shokz OpenComm ~150 €, (c) langes Kabel + USB-Verlängerung. Risiko bei (a): doppelte BT-Strecke + HFP-Codec macht Mic schlechter — viele billige Adapter unterstützen Mic gar nicht.
- [x] **PTT-Trigger erste Iteration: Unified Remote** ✓ (gratis, Pixel-Custom-Remote für Space-Hold). Eskalation auf Stream Deck Pedal / Flic 2 / PWA-Button nur bei Komfort-Bedarf in Phase 2b oder Phase 6.
- [→] **Edge-TTS-Stimme: in Phase 2 mit Probehören entschieden** (Killian/Katja/Conrad/weitere). 5-Minuten-Test mit einer Zeile Python.

---

## Architektur-Entscheidungen — alle in jeweiliger Phase entschieden

Keiner dieser Punkte blockiert Phase 0/1. Werden bei Implementierung der jeweiligen Phase entschieden.

- [→] **Adapter-Interface (STT/TTS)** — wächst in Phase 2 entlang Code, keine Vorab-Spec.
- [→] **Vault-Pfad** (Phase 5) — Default: `C:\MeineDaten\second-brain\`. Final dann.
- [→] **Action-Queue-Storage-Pfad** (Phase 3) — Default: lokal in `C:\MeineDaten\jarbrew\.jarvis\actions.json`, ab Phase 5 in den Vault verschieben.
- [→] **PWA-Framework** (Phase 6) — Default: Svelte+Vite. Final in Phase 6.
- [→] **Scheduler-Tool** (Phase 8) — Default: Windows Task Scheduler. Final in Phase 8.
- [→] **RAG-Vektor-Store** (Phase 12 Roadmap) — Default: LanceDB. Erst entscheiden, wenn Vault-Volumen Filesystem-Suche überfordert.
- [→] **Wake-Word-Engine** (Phase 7) — Default: openWakeWord. Picovoice Porcupine als Backup-Option.

---

## Operative Themen

- [x] **Claude Code `/voice` zuhause getestet** ✓ (2026-04-28)
  - Transkription funktioniert gut, Anglizismen kommen durch.
  - Findings: kein `auto`-Sprachmodus, nur feste Sprachen (de/en via `/config`). Nur in Claude Code CLI nutzbar, nicht mobil.
  - Groq-Whisper-Pipeline für Phase 2 nicht nötig — `/voice` reicht. Eigener Client erst ab Phase 7 (Wake-Word + PWA-STT).

---

## Phase-10+-Re-Evaluation (nicht jetzt entscheiden)

- [ ] **Hybrid-Router (Haiku-Dispatcher + Claude-Code-Backend)** — re-evaluieren, falls nach 2–4 Wochen Praxis Latenz oder Token-Kosten bei Mikro-Tasks spürbar nerven.
  - Andockpunkt: Verifier-Middleware (Sicherheitsnetz Ebene 3, Section 8 PRP).
  - Praxisdaten zuerst, dann entscheiden — Tabelle 12 #11.
- [ ] **Native-App-Wrapper** (Capacitor/Tauri) — nur falls PWA-Limits beißen, vor allem auf iOS bei Web-Push.

---

## Reihenfolge-Empfehlung für Heim-Start

1. **Commit + Push** — alle Änderungen dieser Session sichern (auf `asphyxx24/jarbrew`).
2. **Phase 0** abschließen: Hostname-Reboot, WoL/BIOS/Power-Setup. ~15 Min.
3. **Phase 1** nach PRP Section 9: Tailscale, Claude Code, OpenSSH, tmux, GitHub-MCP, Filesystem-MCP. ~60 Min.
4. In Phase 2: Edge-TTS-Stimme probehören, Recon 70 AR per Kabel testen, Unified Remote einrichten. `/voice` läuft bereits ✓.
5. Adapter-Interface entsteht entlang Phase 2.
