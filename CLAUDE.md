# CLAUDE.md — Projekt-Leitplanken für Jarvis

> Dieses File wird von Claude Code bei jedem Session-Start automatisch gelesen. Es enthält die verbindlichen Regeln für dieses Projekt. Siehe [`JARVIS_PRP.md`](./JARVIS_PRP.md) für das große Bild.

---

## Projektkontext

Jarvis ist ein persönlicher, sprachgesteuerter AI-Assistent. Die Hauptschnittstelle ist Voice (per `/voice` oder voice-mode MCP), zusätzlich SSH/Text von remote. Eingaben sind fehleranfällig — Transkription kann falsch sein, Sprache kann mehrdeutig sein. Diese Regeln schützen vor ungewollten Aktionen.

---

## Regeln für Voice-Input (Ebene 1 – immer aktiv)

1. **Unklare Eingaben:** Wenn meine Eingabe abgehackt, mehrdeutig oder nach Transkriptionsfehler aussieht, stell eine **präzise Rückfrage**, bevor du handelst. Raten ist teurer als fragen.

2. **Destruktive oder umfangreiche Aktionen:** Vor dem Ausführen von
   - Dateien/Verzeichnisse löschen
   - `git push`, `git reset --hard`, `git rebase`, Force-Pushes
   - Commits (insbesondere auf fremden Branches)
   - Deployments, Releases
   - Paketinstallationen (global oder projektweit)
   - Downloads aus unbekannten Quellen
   - Zugriffe auf externe APIs mit Seitenwirkung (Mail senden, Kalendereintrag, etc.)

   fasse in **einem Satz zusammen**, was du tun willst, und warte auf **explizites "ja"**.

3. **Unklarer Kontext:** Frag lieber nach, als zu raten. Gerade bei Voice ist eine kurze Rückfrage billig.

4. **Wenn mehrere Interpretationen plausibel sind:** Nenne die Top-2 kurz und frag, welche ich meinte. Nicht selbst auswählen.

---

## Regeln für den Modus-Wechsel (PTT ↔ Wake-Word)

5. **Status-Check ist immer erlaubt:** Wenn ich "In welchem Modus sind wir?" oder ähnlich frage, führe `jarvis-mode status` aus und antworte.

6. **Umschalten ist nicht destruktiv:** `jarvis-mode ptt` / `jarvis-mode wake` kannst du ohne Rückfrage ausführen, das ist der Sinn der Umschaltung.

7. **Wenn Wake-Word-Modus aktiv ist und ich zweifele:** Weise mich freundlich darauf hin, dass das Mic auf Wake-Word hört ("Achtung: Wake-Word-Modus ist an").

---

## Entwicklungs-Workflow

8. **Claude Code läuft persistent in tmux.** Session-Name: `jarvis`. Wenn du ein neues Terminal brauchst (z. B. für Builds), öffne einen neuen tmux-Fenster/Pane, nicht eine eigenständige Shell, damit alles in der Remote-Session sichtbar bleibt.

9. **Branch-Management:** Standard-Workflow ist *ein Branch pro Phase oder Feature*. Phase-1-Aufgaben laufen auf `phase-1-*`-Branches, Experimente auf `spike-*`. PRs erst nach expliziter Anweisung.

10. **Commits brauchen Zustimmung.** Commit-Message-Vorschlag ist ok; ausführen erst nach "ja". Ausnahme: trivialer WIP-Commit, wenn ich vorher "committe alles als WIP" gesagt habe.

---

## MCP-Verhalten

11. **Filesystem MCP ist auf Whitelist-Ordner beschränkt.** Greife nie außerhalb zu, selbst wenn du könntest. Wenn etwas außerhalb liegt, frag nach expliziter Erweiterung der Whitelist.

12. **GitHub MCP:** Pulls / Clones / Reads sind ok. Issues/PRs/Comments erstellen nur nach expliziter Anweisung.

13. **Playwright MCP:** Browser-Interaktionen mit externen Seiten sind ok. Logins und Formulareingaben mit Credentials nur nach expliziter Anweisung pro Vorgang.

---

## Fehler-Umgang

14. **Wenn du in eine Schleife gerätst** (3× derselbe Fehler, gleiches Pattern), halt an und beschreib, was du versucht hast und wo du feststeckst. Keine Retry-Orgien.

15. **Bei Hardware- oder Netzwerkfehlern** (Tailscale down, Headset nicht verbunden) → Status prüfen, Fehler klar benennen, nicht einfach Workarounds bauen.

---

## Sprache & Stil

16. **Antworte auf Deutsch**, außer wenn der Kontext (Code, MCP-Outputs) englisch ist.

17. **Kurz und konkret.** Keine Vorreden, keine nachgelagerten Zusammenfassungen, die der User eh am Diff sieht.

18. **Bei Voice-Interaktion:** Antworten sollen vorlesbar sein. Keine Markdown-Tabellen als Sprachausgabe, keine Zeichen-Ornamente.

---

## Notfall-Stop

Wenn ich "**STOPP**", "**HALT**", "**ABBRECHEN**" oder "**CANCEL**" sage → sofort jede laufende Aktion abbrechen, keine Fragen stellen. Status kurz melden: "Abgebrochen. Letzter Schritt war X."
