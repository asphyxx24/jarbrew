# jarbrew — Jarvis Personal AI Assistant

Persönlicher, sprachgesteuerter AI-Assistent auf Basis von Claude Code. Läuft persistent in einer tmux-Session auf einem Windows-PC, erreichbar über Tailscale von überall.

## Schnellstart für neue Sessions

```
Lies docs/JARVIS_PRP.md und CLAUDE.md.
Prüfe CHANGELOG.md für den aktuellen Stand.
Aktuelle Phase findest du in docs/JARVIS_PRP.md Section 9 oder in phases/.
```

## Struktur

```
CLAUDE.md              Verhaltensregeln für Claude Code (immer im Root)
CHANGELOG.md           Was ist wann fertig?
README.md              Diese Datei

docs/
  JARVIS_PRP.md        Hauptdokument: Zielbild, Architektur, Phasen, Entscheidungen
  STT_TTS_OPTIONS.md   STT/TTS-Stack-Analyse und Entscheidungsgrundlage
  adr/                 Architecture Decision Records (leichtgewichtige Entscheidungshistorie)
    001-universal-harness.md
    002-wol-sleep-betriebsmodus.md
    003-stt-tts-stack.md
    004-mobile-pwa.md
    005-voice-cli-only.md

phases/
  phase-0/checklist.md  Vorbereitung: Hostname, WoL, Power-Settings
  phase-1/checklist.md  Grundsetup: Tailscale, Claude Code, tmux, MCPs
  phase-2/              Voice PTT (coming)
  phase-3/              Action-Queue + Slash-Commands (coming)

src/
  voice/               Phase 2+: STT/TTS-Client (Groq-Streaming, Edge-TTS)
  action-queue/        Phase 3+: Approval-System
  slash-commands/      Phase 3+: Custom Claude Slash-Commands
  pwa/                 Phase 6+: Mobile PWA

.archive/              Veraltete Dokumente (zur Nachvollziehbarkeit erhalten)
```

## Wichtigste Entscheidungen (nie rückwärts ohne Anlass)

| Entscheidung | ADR |
|---|---|
| Claude Code als Universal-Harness, kein Dispatcher-LLM | ADR-001 |
| WoL + 60-Min-Sleep (S3) | ADR-002 |
| Groq Whisper + Edge-TTS + Adapter-Pattern | ADR-003 |
| PWA als Mobile-Frontend, kein Telegram | ADR-004 |
| `/voice` ist CLI-only, kein auto-Sprachmodus | ADR-005 |
