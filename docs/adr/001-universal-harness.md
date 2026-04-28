# ADR-001: Claude Code als Universal-Harness

**Status:** Entschieden (2026-04-28)

**Kontext:**
Frage: Was ist der Kern von Jarvis — ein Custom-Orchestrator (kleines Dispatcher-LLM + schwere Modelle on-demand) oder Claude Code als Hauptkomponente? Hintergrund: ein alternativer Claude-Chat hatte einen Haiku-Dispatcher-Ansatz vorgeschlagen.

**Entscheidung:**
Claude Code ist der Universal-Harness. Claude (das Modell) ist Jarvis. Claude Code ist die persistente Agent-Laufzeit mit MCP-Plugin-System, Permission-Flow, Compaction und Hooks. Kein zweites Orchestrator-LLM vor Phase 7.

**Konsequenzen:**
- Mikro-Tasks ("Licht aus") laufen bewusst durchs Hauptmodell (~1–2 s, ~0,5 ct/Aktion) — akzeptierter Preis für eine Identität, einen Kontextfaden.
- Hybrid-Router (Haiku-Dispatcher + Claude-Code-Backend) ist eine Phase-10+-Re-Evaluation, falls Latenz/Kosten in Praxis nerven.
- Andockpunkt wäre Sicherheitsnetz Ebene 3 (Verifier-Middleware) in `docs/JARVIS_PRP.md` Section 8.
