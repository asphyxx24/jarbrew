# ADR-005: /voice ist Claude Code CLI-only

**Status:** Entschieden aus Praxis-Test (2026-04-28)

**Kontext:**
Beim ersten `/voice`-Test in Claude Code wurden zwei Einschränkungen festgestellt, die im PRP und in der Planung nicht explizit waren.

**Befunde:**
1. `/voice` unterstützt **kein `auto`-Sprachmodus** — nur feste Sprachen (de/en via `/config`). Anglizismen kommen im DE-Modus durch, längere englische Phrasen werden eingedeutscht.
2. `/voice` läuft **ausschließlich in Claude Code CLI** — nicht in der Claude Mobile App, nicht über SSH/tmux-Pipes vom Handy, nicht in der PWA.

**Entscheidung / Konsequenzen:**
- Phase 1/2: Sprache auf `de` setzen, bei stark englischem Kontext temporär auf `en` wechseln (`/config`).
- Phase 6 (PWA): eigene STT-Pipeline (Groq-Streaming-Client) nötig.
- Phase 7 (Wake-Word): Groq-Pipeline mit `language="auto"` — dann ist Code-Switching gelöst.
- Bahn-Remote / Termux-SSH: Voice nicht möglich, Text/Diktat reicht (laut Zielbild Section 1).
- `/voice`-Qualität wurde als gut bewertet — Transkription funktioniert, Anglizismen kommen durch.
