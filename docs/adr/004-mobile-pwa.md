# ADR-004: Mobile-Frontend — PWA (kein Telegram-MVP)

**Status:** Entschieden (2026-04-28)

**Kontext:**
Mobile-Frontend für Jarvis: Telegram-Bot als MVP → PWA, oder direkt PWA? Ein alternativer PRP-Entwurf schlug Telegram als Schnellstart-MVP vor.

**Entscheidung:**
Direkt PWA als Mobile-Frontend. Native App (Capacitor/Tauri) nur als Notlösung falls PWA-Limits beißen (insbesondere iOS Web-Push).

**Konsequenzen:**
- PWA-Features: Voice-PTT-Button, Action-Queue-Liste mit Approve/Reject/Edit, Web-Push.
- Backend: minimale API auf dem PC (Tailscale), die STT, Action-Queue und Push orchestriert.
- **PWA braucht eigene STT-Pipeline** — `/voice` ist Claude-Code-CLI-only, nicht in PWA nutzbar. Groq-Streaming-Client in Phase 6 gebaut.
- Framework-Entscheidung (Svelte+Vite/React/Solid) in Phase 6.
- Telegram verworfen: unnötiger Zwischenschritt, schlechtere UX als PWA.
