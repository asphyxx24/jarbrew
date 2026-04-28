# WORKFLOWS — Living Document

> Wie wird mit Jarvis gearbeitet? Dieses Dokument beschreibt Dev-Workflows, Command-Chaining und Session-Rhythmus. Wird mit dem Projekt mitgepflegt.

---

## Tagesablauf (Standard)

```
Morgens
  → PC starten (oder per WoL wecken)
  → Git Bash → tmux new-session -A -s jarvis → claude
  → Claude liest CLAUDE.md automatisch → meldet sich mit Stand
  → Optional: /prime für vollständigen Briefing

Während der Arbeit
  → Arbeite, Claude handelt auf Anweisung
  → Neue Entscheidung? → /decision [was] oder /adr [was]
  → Kurze Orientierung? → /status

Session beenden
  → /checkpoint
  → Claude fasst zusammen, updatet CHANGELOG.md, hakt Checkliste ab
  → Commit + Push auf Bestätigung

Abends
  → PC schläft nach 60 Min automatisch (S3-Sleep, tmux-Session bleibt)
```

---

## Command-Referenz & Chaining

### `/prime` — Vollständiger Einstieg
**Wann:** Neuer Chat, nach langer Pause, nach Reboot.
**Was:** Liest CHANGELOG + PRP-Zielbild + Phase-Checkliste, gibt Stand + nächsten Schritt.
**Chaining:**
```
/prime → [Arbeiten] → /checkpoint
```

### `/status` — Schneller Puls
**Wann:** Kurze Unterbrechung, Orientierung zwischendurch.
**Was:** CHANGELOG-Header + offene Checkboxen der aktuellen Phase. Max. 10 Zeilen.
**Chaining:**
```
/status → [gezielt weiterarbeiten] → /checkpoint
```

### `/checkpoint` — Session dokumentieren
**Wann:** Am Ende jeder produktiven Session, vor längerem Break.
**Was:** Fasst zusammen was passiert ist, updatet CHANGELOG.md, hakt Checkboxen ab, schlägt Commit vor.
**Chaining:**
```
[Arbeiten] → /checkpoint → [commit+push] → Session Ende
```

### `/adr [Entscheidung]` — Wichtige Entscheidung festhalten
**Wann:** Wenn eine Architektur-, Tech-Stack- oder Workflow-Entscheidung getroffen wird, die später Fragen aufwirft.
**Was:** Erstellt `docs/adr/NNN-[name].md` mit Kontext, Entscheidung, Konsequenzen.
**Chaining:**
```
[Entscheidung diskutiert] → /adr [was entschieden] → /checkpoint
```
**Faustregel:** Wenn du dir vorstellen kannst, in 3 Monaten zu fragen "warum haben wir das nochmal so gemacht?" → ADR schreiben.

### `/decision [Entscheidung]` — Kleine Entscheidung schnell festhalten
**Wann:** Wenn eine Entscheidung zu klein für eine vollständige ADR ist.
**Was:** Eintrag in CHANGELOG.md.
**Chaining:**
```
[Kleinere Entscheidung] → /decision [was] → weiterarbeiten
```

### `/phase-done` — Phase abschließen
**Wann:** Wenn alle Checkboxen einer Phase erledigt sind.
**Was:** Hakt ab, erstellt nächste Phase-Checkliste, updatet CHANGELOG.
**Chaining:**
```
[Letzte Checkbox erledigt] → /phase-done → /prime [nächste Phase starten]
```

---

## Typische Workflows nach Use-Case

### Workflow A — Coding-Session (PC, Voice)

1. tmux-Session `jarvis` attachen oder starten
2. Claude meldet sich automatisch mit Stand (CLAUDE.md Session-Start-Ritual)
3. Feature/Bug per Voice beschreiben
4. Claude liest relevante Dateien via Filesystem-MCP, macht Vorschlag
5. Abnicken per "ja" → Claude schreibt Code
6. Code reviewen, ggf. Korrekturen ansagen
7. Commit: Claude schlägt Message vor → "ja" → commit
8. Session Ende: `/checkpoint`

### Workflow B — Bachelorarbeit-Session

1. `/prime` oder automatischer Stand
2. Kapitel-Abschnitt per Voice beschreiben ("ich brauche zwei Sätze für Kapitel 4...")
3. Claude liest Thesis-Repo via Filesystem-MCP
4. Absatz formuliert → in Zwischenablage oder als Commit-Vorschlag ins Repo
5. `/checkpoint` am Ende

### Workflow C — Neue Entscheidung gefallen

```
User: "Ich hab entschieden, wir nutzen SQLite statt JSON für die Action-Queue"
→ /adr Wir nutzen SQLite statt JSON für Action-Queue-Storage — Begründung: Concurrency-Anforderungen in Phase 5
→ Claude erstellt ADR-Entwurf, wartet auf Bestätigung
→ Bestätigt → Datei erstellt → /checkpoint
```

### Workflow D — Unterwegs (Mobile, Text)

1. Pixel → Termux → `mosh jarvis.<tail-id>.ts.net -- tmux new-session -A -s jarvis`
2. Bestehende Claude-Session attachen oder neu starten
3. Text/Diktat als Eingabe
4. Approval für Actions via CLI oder (Phase 6+) via PWA
5. Kein `/checkpoint` nötig wenn nur kurze Aktion

### Workflow E — Phase starten (Beginn jeder technischen Phase)

1. `/prime` → Stand prüfen
2. `phases/phase-N/spec.md` erstellen: Claude schreibt Spec basierend auf PRP Section 9 + User-Input
3. Spec reviewen, anpassen
4. Implementierung nach Checkliste
5. Am Ende: `/phase-done`

---

## Update-Rhythmus

| Was | Wann | Wie |
|---|---|---|
| `CHANGELOG.md` | Nach jeder Session | `/checkpoint` |
| Phase-Checkliste | Während Implementierung | `/checkpoint` oder manuell |
| ADR | Bei wichtiger Entscheidung | `/adr [Entscheidung]` |
| `JARVIS_PRP.md` | Bei größeren Architektur-Änderungen | Direkt bearbeiten + commit |
| `docs/WORKFLOWS.md` (diese Datei) | Wenn sich Workflows ändern | `/decision` + manuell anpassen |
| `CLAUDE.md` | Wenn Verhaltensregeln sich ändern | Direkt bearbeiten + commit |

---

## Phase-Spec-Format (ab Phase 2)

Vor dem Start jeder technischen Phase wird `phases/phase-N/spec.md` erstellt:

```markdown
# Phase N — [Name] Spec

## Was wird gebaut
[2-3 Sätze: Ziel dieser Phase]

## Komponenten
- [Komponente 1]: [was sie tut, welche Datei/Ordner]
- [Komponente 2]: ...

## Interface / API
[Wie wird die Komponente aufgerufen? Inputs/Outputs]

## Abhängigkeiten
[Was muss vorher existieren?]

## Nicht in dieser Phase
[Was kommt später — verhindert Scope Creep]
```

---

## Troubleshooting

**Claude verliert Kontext mid-Session:** `/status` → orientiert sich neu aus CHANGELOG + Checkliste.

**Große Architektur-Frage taucht auf:** Diskutieren → `/adr [Entscheidung]` → weiter. Nicht einfach durchrauschen.

**Session hat viel verändert aber `/checkpoint` vergessen:** Beim nächsten Start `/prime` tippen → Claude sieht CHANGELOG und erkennt, was noch nicht dokumentiert ist.

**Phase fühlt sich fertig an aber Checkliste hat noch offene Punkte:** Offen klären ob das absichtlich übersprungen wird → wenn ja, Checkbox als `[~]` markieren mit Kommentar (nicht erledigt, aber bewusst geskippt).
