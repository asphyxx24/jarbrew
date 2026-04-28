# Offene Punkte vor Heim-Start

> Stand: 2026-04-28. Sortiert nach Dringlichkeit. Sobald ein Punkt entschieden ist, abhaken und Begründung in `JARVIS_PRP.md` Tabelle 11 oder `STT_TTS_OPTIONS.md` nachpflegen.

---

## Vor Phase 1 sinnvoll zu klären

- [ ] **Tailscale-Account** — bereits vorhanden, oder neu anlegen?
  - Wenn neu: per Google- oder GitHub-Login (kein extra User/Passwort).
  - Free-Tier reicht für persönlichen Gebrauch (PRP-Tabelle-11 #5 ✓).
- [ ] **PC-Hostname festlegen** — wird Teil des MagicDNS-Namens (z. B. `workshop.<tail-id>.ts.net`). Sollte stabil sein, nicht später wechseln.
- [ ] **Betriebsmodus** — 24/7 mit Sleep aus (PRP-Default) oder Wake-on-LAN?
  - Empfehlung: zum Start 24/7, nach 1 Monat Stromrechnung re-evaluieren (PRP Tabelle 11 #2).
- [ ] **Bachelorarbeit-Integration** — Overleaf-Git-Sync aktivieren ja/nein?
  - Wenn ja: Pfad zum lokalen Clone festlegen (z. B. `C:\BA\bachelor`).
  - Filesystem-MCP-Whitelist daran anpassen.

---

## Hardware-Entscheidungen (vor Phase 2)

- [ ] **Bluetooth-Headset** — schon eins vorhanden, das gut genug ist? Oder neu kaufen?
  - PRP-Empfehlung: Shokz OpenComm (Open-Ear, sehr gutes Mic für Voice).
  - Alternativen: Sony WH-1000XM5, AirPods Pro (2. Gen+).
  - Test-Kriterium: Mic muss klar genug für Whisper sein, auch beim Rumlaufen.
- [ ] **PTT-Trigger erste Iteration** — Unified Remote als Start (gratis, Handy-Custom-Remote für Space-Hold). Upgrade auf Stream Deck Pedal (~90 €) nur bei Komfort-Bedarf.
- [ ] **Edge-TTS-Stimme festlegen** — Probehören und entscheiden:
  - `de-DE-KillianNeural` (männlich)
  - `de-DE-KatjaNeural` (weiblich)
  - `de-DE-ConradNeural` (männlich)
  - oder andere aus dem Edge-TTS-Pool.
  - Kann mit einer Zeile Python in Phase 2 erledigt werden.

---

## Architektur-Entscheidungen (vor Implementierung)

- [ ] **Adapter-Interface skizzieren** — Funktionssignaturen + Config-Format für STT/TTS-Backends.
  - Kann als kleine Markdown-Spec entstehen, oder direkt im Code in Phase 2 wachsen.
  - Mindestanforderung: ein Konfig-Switch (Env-Var oder Config-File), der Backend zur Laufzeit auswählt.
- [ ] **Wake-Word-Engine** (Phase 5) — openWakeWord (PRP-Default) oder Picovoice Porcupine.
  - Kann warten bis Phase 5.

---

## Operative Themen

- [ ] **Claude Code `/voice` zuhause testen** — vor jeder Pipeline-Bastelei.
  - Wenn `/voice` zufriedenstellend läuft, ist die Groq-Whisper-Pipeline für Phase 1 nicht nötig.
  - Groq + eigene Pipeline werden erst ab Phase 5 (Wake-Word) wirklich relevant.
- [ ] **HANDOFF.md** — entscheiden, ob aktualisieren oder archivieren.
  - Stand der Datei ist vor der STT/TTS-Klärung.
  - PRP-Sicht: nach Phase 1 ohnehin archivierbar.

---

## Reihenfolge-Empfehlung für Heim-Start

1. Punkte 1–3 (Tailscale, Hostname, Betriebsmodus, BA-Integration) in einem Rutsch klären — 10 Minuten am PC.
2. Phase-1-Setup nach `JARVIS_PRP.md` Section 8 (Phase 1) ausführen.
3. **Allererstes Praxistest:** `/voice` in Claude Code testen — wenn das gut läuft, spart man sich viel Pipeline-Bastelei.
4. Erst danach Hardware-Themen (Headset, PTT-Trigger) und Edge-TTS-Stimm-Auswahl angehen.
5. Adapter-Interface entsteht entlang Phase 2.
