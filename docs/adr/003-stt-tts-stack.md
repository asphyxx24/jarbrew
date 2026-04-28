# ADR-003: STT/TTS-Stack — Groq Whisper + Edge-TTS + Adapter-Pattern

**Status:** Entschieden (2026-04-28)

**Kontext:**
Welcher STT/TTS-Stack? Optionen: Deepgram Streaming, Groq Whisper, OpenAI, AWS, ElevenLabs, lokale Modelle. Details in `docs/STT_TTS_OPTIONS.md`.

**Entscheidung:**
- **STT Primary:** Groq Whisper Large v3 Turbo (~$1.80/Monat bei 45 h). `language="auto"` Pflicht für Groq-Pipeline.
- **TTS Primary:** Microsoft Edge-TTS (gratis, deutsche Neural-Stimmen). Stimme in Phase 2 per Probehören gewählt.
- **Lokal-Fallback STT:** faster-whisper `medium` (CPU) oder whisper.cpp + Vulkan.
- **Lokal-Fallback TTS:** Piper.
- **Architektur:** Adapter-Pattern — Backends hinter stabilem Interface, Config-Switch zur Laufzeit.

**Konsequenzen:**
- `/voice` in Claude Code hat kein `auto`, nur feste Sprachen (de/en via `/config`) — gilt für Phase 1/2.
- Groq-Pipeline mit `language="auto"` kommt in Phase 7 (Wake-Word + eigene STT-Pipeline).
- Cloud-Backup-Slots: Deepgram, Azure Neural, Google Neural2 (optional).
- **Verworfen:** OpenAI STT/TTS, AWS Polly/Transcribe, ElevenLabs, EC2 Self-Hosting.
