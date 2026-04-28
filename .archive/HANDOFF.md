# HANDOFF — Weitermachen am Heim-PC

> **STATUS: ARCHIVIERT (2026-04-28).**
> Diese Datei war die einmalige Übergabe-Doku für den Repo-Sprung von Conductor-Worktree auf den Heim-PC. Das Repo ist inzwischen am Heim-PC angekommen, der PRP wurde erweitert (Reframing auf Universal-Harness, STT/TTS festgelegt). Aktuelle Quellen: `JARVIS_PRP.md` (Hauptdokument), `CLAUDE.md` (Verhaltensregeln), `OPEN_QUESTIONS.md` (offene Punkte), `STT_TTS_OPTIONS.md` (Speech-Stack).
> Diese Datei bleibt zur Nachvollziehbarkeit der ursprünglichen Übergabe erhalten — der Inhalt unten ist nicht mehr autoritativ.
>
> ---
>
> **Zweck (historisch):** Dich zuhause ohne Vorwissen auf den aktuellen Projektstand bringen, damit du mit Claude Code direkt weiterarbeiten kannst. Nach Phase 1 darf dieses Dokument archiviert oder gelöscht werden.
>
> **Erstellt am:** 2026-04-22 · **Branch:** `jarvis-prp`

---

## 1. TL;DR (30 Sekunden)

Du baust **Jarvis** — einen persönlichen, sprachgesteuerten AI-Assistenten auf deinem Windows-PC. Die Projektplanung ist fertig und liegt in `JARVIS_PRP.md`. Als nächstes steht **Phase 1 (Grundsetup)** an: Tailscale + Claude Code + SSH + tmux + erste MCPs. Dauer grob 60 Minuten.

**Zuerst tun:**

1. Dieses Repo pullen (du bist drin).
2. `JARVIS_PRP.md` lesen — das ist dein Leitfaden.
3. `CLAUDE.md` lesen — die Verhaltensregeln für Claude Code.
4. Claude Code öffnen und mit dem Prompt starten: *"Lies JARVIS_PRP.md und CLAUDE.md. Wir sind in Phase 1. Leite mich Schritt für Schritt durch die Checkboxen in Section 8, Phase 1."*

---

## 2. Was ist Jarvis? (Kurzkontext)

Ziel: Ein persönlicher Assistent, den du **per Sprache** ansprechen kannst, während du durch die Wohnung läufst. Er läuft als Claude Code in einer persistenten `tmux`-Session auf deinem Windows-PC, ist über **Tailscale** von überall erreichbar (Handy, Tablet), hat Zugriff auf deine Dateien + GitHub + Browser, und kann später sogar einen **eigenen Desktop** bekommen, dem du beim Arbeiten live zuschauen kannst.

Dreh- und Angelpunkt: **Claude Code** (CLI) plus **MCP-Server** (GitHub, Filesystem, Playwright). Kein Voice-Setup von unterwegs nötig — remote reicht Text/Diktat.

Wenn du mehr Kontext willst, lies die zwei Kern-Dateien (siehe §4).

---

## 3. Aktueller Stand

**Was schon passiert ist:**

- ✅ PRP geschrieben → `JARVIS_PRP.md` (14 Sections, ausführlich).
- ✅ Voice-Sicherheitsregeln dokumentiert → `CLAUDE.md` (liest Claude Code automatisch).
- ✅ Vorgängerdokument archiviert → `.context/attachments/pasted_text_2026-04-22_13-40-59.txt`.
- ✅ Branch umbenannt → aktueller Branch heißt `jarvis-prp`.
- ✅ Grundentscheidungen getroffen (siehe §7).

**Was noch NICHT passiert ist:**

- ❌ Nichts wurde installiert.
- ❌ Nichts wurde committet oder gepusht.
- ❌ Kein Hardware-Kauf gemacht.
- ❌ Keine Konten/Accounts angelegt (außer Claude Pro, das hast du schon).

Du fängst also mit einer reinen Planung an und setzt jetzt Phase 1 um.

---

## 4. Dateien im Repo — was wofür?

| Datei | Zweck | Wann lesen? |
|---|---|---|
| `JARVIS_PRP.md` | **Das Hauptdokument.** Zielbild, Architektur, 4 Workflows, Phasenplan, Zukunftsroadmap, Entscheidungstabelle, Referenzen. | Jetzt, einmal komplett durchscrollen. Bei Fragen gezielt Section ansteuern. |
| `CLAUDE.md` | Verhaltensregeln für Claude Code (Voice-Sicherheit, Commits, MCP-Grenzen, Notfall-Stop). Claude Code lädt das automatisch bei Session-Start. | Einmal lesen, danach nur bei Anpassungen. |
| `HANDOFF.md` (diese Datei) | Dein Übergabe-Dokument für den ersten Session-Start zuhause. | Jetzt. Danach archivierbar. |
| `.context/attachments/pasted_text_2026-04-22_13-40-59.txt` | Original-Entwurf, aus dem die PRP entstanden ist. Archiv, nicht mehr aktiv nötig. | Nur bei Zweifeln an PRP-Inhalten, um die ursprüngliche Intention nachzulesen. |
| `.context/` | Arbeitsordner (gitignored in Conductor). Hier dürfen Zwischen-Notizen rein. | Nach Bedarf. |

**Tipp:** Der schnellste Einstieg ist Section 8 in `JARVIS_PRP.md` (Phasenplan) + Section 3 (Workflows).

---

## 5. Erste Schritte zuhause — in dieser Reihenfolge

### 5.1 Repo aktuell holen

Öffne PowerShell oder Git Bash im Projektverzeichnis und:

```bash
git fetch
git checkout jarvis-prp
git pull
```

Du solltest jetzt `JARVIS_PRP.md`, `CLAUDE.md` und `HANDOFF.md` im Repo-Root sehen.

### 5.2 Die drei Dokumente lesen (in Reihenfolge)

1. **Dieses Dokument** fertig durchlesen (du bist hier).
2. **`JARVIS_PRP.md`** einmal komplett — 15 Minuten. Fokus auf Section 3 (Workflows), 8 (Phasenplan), 11 (Entscheidungstabelle).
3. **`CLAUDE.md`** — 3 Minuten. Die Regeln solltest du im Kopf haben, vor allem die Destruktiv-Regel (Nr. 2).

### 5.3 Claude Code starten

Falls noch nicht installiert:

```powershell
# PowerShell als normaler User (nicht Admin)
irm https://claude.ai/install.ps1 | iex
claude --version   # muss ≥ 2.1.69 sein
```

Login mit deinem Claude-Pro-Account (nicht API-Key — der Pro-Account ist Pflicht für `/voice`).

### 5.4 Claude Code briefen (Copy-Paste-Prompt)

Wenn du Claude Code im Repo-Ordner öffnest, wird `CLAUDE.md` automatisch geladen. Dann sag ihm:

> Lies `JARVIS_PRP.md` vollständig und dann `HANDOFF.md`. Wir sind in **Phase 1 — Grundsetup**. Gehe die Checkboxen in Section 8 Phase 1 einzeln durch. Für jede: sag mir, was als nächstes passiert, warte auf mein "ja", dann führ aus. Wenn du ein Tool oder eine Installation brauchst, fass in einem Satz zusammen, was du tun willst, bevor du es tust (CLAUDE.md Regel 2).

Ab hier führt er dich.

---

## 6. Wie du mit Claude Code am besten arbeitest

- **Sei explizit, nicht knapp.** "Installiere Tailscale" ist ok; "Installiere Tailscale, aber ich will den Login-Prozess manuell machen, weil ich 2FA habe" ist besser.
- **Commits nur auf Ansage.** In `CLAUDE.md` Regel 10 steht, dass Claude keine Commits ohne dein "ja" macht. Halte dich selbst auch dran und frag explizit nach, wenn du committen willst.
- **Bei Voice-Fehlern ruhig bleiben.** Voice-Transkription kann Fehler machen. Wenn Claude nachfragt, ist das kein Bug — das ist die Regel aus `CLAUDE.md` Nr. 1.
- **Phasen nicht überspringen.** Verlockung groß, gleich Phase 6 (Agent-Desktop) auszuprobieren. Nicht tun. Phase 1 muss stabil laufen, bevor Voice Sinn macht.
- **Workflow-Kritik jederzeit willkommen.** Wenn in Section 3 ein Schritt nicht passt ("Schritt 3 in Workflow 1 nervt mich"), sag das. Architektur wird dann angepasst.

---

## 7. Getroffene Entscheidungen (NICHT rückwärts laufen)

Diese Punkte sind beschlossen, um nicht im Kreis zu diskutieren. Wenn du sie ändern willst, ok — aber bitte bewusst, nicht versehentlich.

| # | Entscheidung | Begründung |
|---|---|---|
| 1 | **OS: Windows nativ**, kein WSL2 für Claude Code selbst | Audio-Passthrough für `/voice` einfacher. Native Install seit 2025 stabil. |
| 2 | **Sprachstack: lokal** (Whisper + Kokoro), nicht OpenAI-Cloud | Null laufende Kosten, Privacy. Cloud nur als Fallback. |
| 3 | **PTT ist Default**, Wake-Word opt-in | Privacy-first. Wake-Word wird gezielt per `jarvis-mode wake` gestartet. |
| 4 | **PRP als Living Document** | Fortschritt wird abgehakt, neue Erkenntnisse fließen zurück in `JARVIS_PRP.md`. |
| 5 | **Tailscale Free-Tier** reicht | Persönlicher Gebrauch, wenige Geräte, SSH + MagicDNS sind im kostenlosen Plan enthalten. |
| 6 | **Bachelorarbeit: LaTeX via Overleaf** | Empfehlung: Overleaf-Git-Sync aktivieren, dann Filesystem-MCP aufs lokale Clone zeigen. |

Volle Tabelle inkl. noch offener Punkte: `JARVIS_PRP.md` Section 11.

---

## 8. Vorab-Klärung — bevor du Phase 1 startest

Vier Minuten-Fragen. Wenn du eine beantwortest, ist das schon Fortschritt:

1. **Tailscale-Account:** Schon einer vorhanden oder frisch anlegen? (Free-Tier: ja.) Falls neu: mit Google/GitHub-Login, damit du keinen weiteren User/PW brauchst.
2. **Claude Pro aktiv:** ✅ Bestätigt (20 €). `/voice` sollte funktionieren.
3. **PC-Hostname:** Such dir einen aus, den du nicht später ändern willst. Er erscheint als Tailscale-MagicDNS-Name (z. B. `workshop.tailxyz.ts.net`).
4. **tmux via Git Bash / MSYS2 oder via `scoop install tmux`:** Beide gehen. Empfehlung: `scoop` ist moderner und einfacher zu maintaind. Falls scoop nicht da: `irm get.scoop.sh | iex`.

Keine dieser Entscheidungen ist endgültig — aber sie beeinflussen die Installationsreihenfolge.

---

## 9. Stolpersteine — was dich wahrscheinlich ausbremst

1. **`/voice` braucht Claude-Login, nicht API-Key.** Wenn du zufällig einen API-Key in der Env hast, stört das. Check: `claude config` — unter Auth sollte "OAuth" / "Claude.ai" stehen.
2. **PowerShell Execution Policy.** Falls `irm … | iex` meckert: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.
3. **Windows Defender SmartScreen** blockt manchmal das Claude-Installer-Skript. Einmal "Trotzdem ausführen" klicken.
4. **Bluetooth-Headset-Latenz unter Windows.** A2DP-Codec funktioniert für Audio out, aber beim Mic-Input schaltet Windows oft auf Hands-Free-Profile (HFP) mit schlechter Qualität. Workaround: im Sound-Setting manuell das Headset als Default-Input setzen und HFP-Codec auf mSBC oder höher zwingen.
5. **Tailscale-SSH vs. OpenSSH-Server.** Wenn du Tailscale-SSH nutzt (empfohlen), brauchst du **keinen** Windows-OpenSSH-Server. Aber wenn du später mosh willst, doch. Plan in der PRP: OpenSSH-Server *zusätzlich* aktivieren, Tailscale regelt den Access.
6. **tmux unter Git Bash:** Ältere Git-for-Windows-Versionen hatten kein tmux im Bundle. Dann via `scoop install tmux` oder `pacman -S tmux` (falls MSYS2 da ist).
7. **Voice in tmux:** Unsicher, ob `/voice` in Claude Code innerhalb von tmux unter Git Bash einwandfrei ans Windows-Audio kommt. **Falls nicht:** Claude Code zunächst direkt in PowerShell starten (ohne tmux), Voice testen, danach tmux-Variante debuggen. Die Alternativen stehen in Phase-2-Notizen.

---

## 10. Was du explizit NICHT tun sollst

- ❌ **Nicht** Phase 5 (Wake-Word) vor Phase 2 (PTT stabilisieren) angehen.
- ❌ **Nicht** Hardware (Stream Deck Pedal, Shokz Headset) vor Abschluss Phase 2 kaufen. Erst testen, dass Voice mit vorhandener Hardware klappt.
- ❌ **Nicht** committen, pushen oder PRs öffnen ohne explizites "ja". (Das gilt auch für dich gegenüber Claude Code — sag es deutlich.)
- ❌ **Nicht** den `.context/`-Ordner committen. Der ist gitignored und bleibt lokal.
- ❌ **Nicht** das `.git` in diesem Ordner anfassen — das ist ein Conductor-Worktree-Link, kein klassisches `.git`-Verzeichnis.

---

## 11. Nützliche Kommandos — Spickzettel

### Installation (Phase 1, in Reihenfolge)

```powershell
# Git for Windows
winget install --id Git.Git -e

# Claude Code
irm https://claude.ai/install.ps1 | iex

# Tailscale
winget install --id Tailscale.Tailscale -e

# GitHub CLI
winget install --id GitHub.cli -e
gh auth login

# Docker Desktop (erst für Phase 6 nötig, aber kann schon dabei sein)
winget install --id Docker.DockerDesktop -e

# scoop (optional, für tmux)
irm get.scoop.sh | iex
scoop install tmux
```

### Claude Code starten

```bash
# in Git Bash im Projekt-Ordner
tmux new-session -A -s jarvis
claude
```

### Tailscale prüfen

```powershell
tailscale status
tailscale ip -4
```

### Von außen testen (Handy später)

```bash
# auf Android in Termux
ssh deinuser@<pc-tailscale-name>.ts.net
tmux attach -t jarvis
```

---

## 12. Wenn du feststeckst

1. **Claude-Code-Fehler "nicht erkannt":** PATH-Problem. Neustart der Shell oder des ganzen Terminals.
2. **Tailscale verbindet nicht:** UDP-Ports nicht blockiert? Firewall erlaubt Tailscale?
3. **`/voice` reagiert nicht:** Mic-Permission für Terminal/Claude in Windows Settings → Privacy → Microphone explizit erlauben.
4. **Allgemein hängen:** `CLAUDE.md` Regel 14 — "wenn du in eine Schleife gerätst, halt an." Das gilt auch für dich. Lieber Pause, dann frag Claude: "Ich komm hier nicht weiter, hilf mir Schritt für Schritt zu debuggen."

---

## 13. Nächstes Etappenziel (Definition of Done für Phase 1)

Du bist mit Phase 1 fertig, wenn **alle** folgenden Punkte klappen:

- [ ] Vom Handy via Tailscale kannst du deinen PC anpingen (`ping pc.ts.net`).
- [ ] `claude --version` zeigt ≥ 2.1.69.
- [ ] Du kannst in Git Bash eine `tmux`-Session `jarvis` starten, darin `claude` starten, mit `Ctrl+b d` detachen und wieder attachen.
- [ ] GitHub CLI ist eingeloggt (`gh auth status` grün).
- [ ] GitHub MCP und Filesystem MCP sind in Claude Code konfiguriert und antworten auf eine Testfrage.
- [ ] Vom Handy kannst du via Termux (nach Installation) in dieselbe tmux-Session attachen.

Dann weiter mit Phase 2 in `JARVIS_PRP.md`.

---

## 14. Frag, wenn was unklar ist

Wenn dich an der PRP etwas stört oder du einen Workflow-Schritt anders haben willst, sag das Claude Code gleich zu Beginn — *bevor* du anfängst zu installieren. Änderungen am Konzept sind billig, Änderungen an einer schon laufenden Installation sind teuer.

Viel Erfolg. 🏠
