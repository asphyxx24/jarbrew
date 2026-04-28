# CHANGELOG

**Aktuelle Phase:** Phase 0 — Vorbereitung (in Arbeit)

Wann wurde was fertig? Kurze Einträge, neueste zuerst.

---

## [Unveröffentlicht — Phase 0 in Arbeit]

### Geplant
- Phase 0: Hostname-Reboot, WoL/BIOS-Setup, Power-Settings

---

## 2026-04-28 — Planungsphase abgeschlossen

### Erledigt
- PRP v0.3 fertiggestellt (Universal-Harness-Reframing, Phasen 0–9, Action-Queue, Vault, PWA)
- Alle Architektur-Entscheidungen getroffen (ADRs 001–005)
- `/voice` getestet — funktioniert, Einschränkungen dokumentiert
- Repo-Struktur aufgesetzt (docs/, phases/, .archive/, src/, docs/adr/)
- Remote auf `asphyxx24/jarbrew` umgestellt

### Entscheidungen dieser Phase
- Claude Code als Universal-Harness (ADR-001)
- WoL + 60-Min-Sleep S3 (ADR-002)
- Groq Whisper + Edge-TTS + Adapter-Pattern (ADR-003)
- PWA als Mobile-Frontend, kein Telegram (ADR-004)
- `/voice` CLI-only, kein auto-Sprachmodus (ADR-005)
