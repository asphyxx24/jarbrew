# Phase 1 — Grundsetup

## Definition of Done
Vom Pixel via Tailscale in `jarvis`-tmux einloggen, Claude antwortet auf "hi", Detach/Attach klappt. GitHub-MCP und Filesystem-MCP antworten auf eine Testfrage.

## Checkliste

### Tailscale
- [ ] Tailscale auf PC installiert: `winget install --id Tailscale.Tailscale -e`
- [ ] Tailscale auf Pixel installiert (gleicher Account)
- [ ] MagicDNS in Tailscale Admin-Console aktiviert
- [ ] Verbindung getestet: `ping jarvis.<tail-id>.ts.net` vom Pixel

### Claude Code
- [ ] Claude Code installiert: `irm https://claude.ai/install.ps1 | iex`
- [ ] Version geprüft: `claude --version` → ≥ 2.1.69
- [ ] Login mit Pro-Account (OAuth, nicht API-Key — `/voice` braucht Pro-Login)
- [ ] `claude config` → Auth zeigt "Claude.ai / OAuth"

### OpenSSH-Server
- [ ] OpenSSH-Server aktiviert:
  ```powershell
  Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
  Set-Service sshd -StartupType Automatic; Start-Service sshd
  ```
- [ ] Tailscale-SSH-ACL-Regel gesetzt (tag:personal kann tag:personal)
- [ ] SSH-Verbindung vom Pixel getestet: `ssh <user>@jarvis.<tail-id>.ts.net`

### tmux
- [ ] tmux installiert: `scoop install tmux` oder `pacman -S tmux`
- [ ] Session gestartet: `tmux new-session -A -s jarvis`
- [ ] `claude` darin gestartet, Detach-Test: `Ctrl+b d`, Re-Attach: `tmux attach -t jarvis`
- [ ] Alias in `~/.bashrc`: `alias jarvis='tmux new-session -A -s jarvis "claude"'`

### GitHub CLI + MCPs
- [ ] GitHub CLI: `winget install --id GitHub.cli -e && gh auth login`
- [ ] GitHub-MCP konfiguriert und getestet
- [ ] Filesystem-MCP konfiguriert mit Whitelist (jarbrew-Pfad, BA-Pfad)
- [ ] Testfrage: "Lies die README des jarbrew-Repos" → Claude antwortet korrekt

### Overleaf-Git-Sync
- [ ] Overleaf-Git-Sync für Bachelorarbeit aktiviert (Overleaf → Menu → Git)
- [ ] Lokaler Clone-Pfad festgelegt und geklont
- [ ] Filesystem-MCP-Whitelist um BA-Pfad erweitert

## Stolpersteine

- `/voice` braucht Claude.ai-Login, nicht API-Key. Wenn API-Key in Env: löscht oder umbenennt ihn.
- tmux unter Git Bash: ältere Git-for-Windows-Versionen haben kein tmux im Bundle — dann `scoop install tmux`.
- Voice in tmux: unsicher ob Audio-Passthrough klappt. Falls nicht: Claude Code direkt in PowerShell starten, ohne tmux, Voice testen, dann tmux-Variante debuggen.
- Mic-Permission: Windows Settings → Privacy → Microphone → Terminal-App erlauben.
