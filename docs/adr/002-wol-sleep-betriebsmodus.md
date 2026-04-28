# ADR-002: Betriebsmodus — WoL + 60-Min-Sleep (S3)

**Status:** Entschieden (2026-04-28)

**Kontext:**
PC-Betriebsmodus für Jarvis: 24/7 (Sleep aus) oder Wake-on-LAN? PRP-Default war 24/7, aber ~50–80 W idle bedeuten ~10–15 €/Monat.

**Entscheidung:**
WoL + Sleep nach 60 Min Inaktivität (S3 — Suspend to RAM). tmux-Session bleibt im RAM erhalten, Aufwachen <5 s. Wartezeit bei WoL-Wecken akzeptiert (~5–30 s boot/wake).

**Konsequenzen:**
- Phase-0-Setup nötig: BIOS-WoL aktivieren, Netzwerkadapter "nur Magic Packet" erlauben, Fast Startup aus, `powercfg /change standby-timeout-ac 60`.
- WoL-App auf Pixel einrichten (Magic Packet).
- Stromersparnis gegenüber 24/7 erheblich.
