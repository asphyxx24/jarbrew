# STT / TTS — Optionen & Kosten

> Arbeitsdokument zur Entscheidungsfindung. Stand: 2026-04-28.
> Annahmen für Kostenrechnung: **90 min STT pro Tag** (= 45 h/Monat) und **90 min TTS pro Tag** (≈ 2 Mio Zeichen/Monat bei ~750 Zeichen/min Sprechrate).

---

## Anforderungen

- Hauptkriterium STT: **Verständnis** (gemischt Deutsch + Englisch, Anglizismen).
- Hauptkriterium TTS: **Verständlichkeit** > Klangschönheit; "nicht giga arsch klingen".
- Wiederverwendbarkeit ist wichtiger als rein-lokal: Adapter-Architektur, Backends austauschbar (Chef könnte Mac nutzen).
- Lokale Backends als optionale Schnittstelle pro User, kein Self-Hosting auf EC2 (siehe verworfene Optionen).
- Cloud akzeptabel, Kosten transparent.

---

## Ziel-Hardware (Hauptnutzer)

| Komponente | Wert | Implikation |
|---|---|---|
| CPU | Ryzen 7 5800X (8C/16T, Zen 3) | locker für CPU-STT (`medium`), Wake-Word, Container |
| RAM | 32 GB | mehr als genug für alle Phasen |
| GPU | AMD Radeon RX 6700XT, 12 GB VRAM | **kein CUDA**; Vulkan-Backend möglich, ROCm auf Win löchrig |
| OS | Windows 11 | passt zur PRP-Default-Wahl |
| Storage | NVMe SSD | unkritisch |

**Konsequenz für lokale STT-Optionen:**
- `faster-whisper`: nur CPU-Pfad nutzbar → `medium` machbar, `large-v3` zu langsam.
- `whisper.cpp`: hat **Vulkan-Backend**, das auf 6700XT funktioniert → `large-v3` realtime möglich, aber Setup nicht trivial.

---

## STT — Speech-to-Text

| Anbieter | Modell | Lokal/Cloud | Preis | Kosten/Monat (45 h) | Code-Switching DE/EN | Latenz | Anmerkung |
|---|---|---|---|---|---|---|---|
| **Groq** | Whisper-large-v3 | Cloud | $0.111 / h | **~$5** | gut (auto-detect) | sehr niedrig (<1 s) | Bestes Preis-Leistung |
| **Groq** | Whisper-large-v3-turbo | Cloud | $0.04 / h | **~$1.80** | gut | sehr niedrig | **Default-Wahl** |
| **Deepgram** | Nova-3 (streaming) | Cloud | $0.0043 / min | **~$12** | sehr gut, expliziter Multilingual-Modus | extrem niedrig (<300 ms) | Backup-Kandidat |
| **AssemblyAI** | Universal | Cloud | $0.37 / h | **~$17** | gut | mittel | Diarization etc. |
| **Anthropic** | Claude Code `/voice` | Cloud (eingebaut) | im Pro-Abo | **0 €** zusätzlich | gut | mittel | Claude.ai-Login Pflicht |
| **faster-whisper** | medium | Lokal CPU | gratis | **0 €** | gut | ~2–3× Realtime auf 5800X | Einfacher Lokal-Fallback |
| **faster-whisper** | large-v3 | Lokal CPU | gratis | **0 €** | gut | <Realtime ohne CUDA | Auf 6700XT nicht praktikabel |
| **whisper.cpp** | medium / large | Lokal (CPU + Vulkan) | gratis | **0 €** | gut | mit Vulkan realtime | Setup-Aufwand höher |
| **Distil-Whisper** | small.en | Lokal | gratis | **0 €** | nur Englisch | sehr schnell | Für DE/EN-Mix ungeeignet |

---

## TTS — Text-to-Speech

| Anbieter | Modell / Stimme | Lokal/Cloud | Preis | Kosten/Monat (~2 M chars) | Deutsch-Qualität | Anmerkung |
|---|---|---|---|---|---|---|
| **Microsoft Edge-TTS** | Neural-Stimmen (de-DE) | Cloud (inoffiziell) | gratis | **0 €** | sehr gut, natürlich | **Default-Wahl** |
| **Azure** | Neural TTS (de-DE) | Cloud | $16 / 1M chars (FT 500k/Monat) | **~$32** | sehr natürlich, viele Stimmen | Offizielle Variante von Edge-TTS |
| **Google Cloud** | Neural2 / WaveNet | Cloud | $16 / 1M chars (FT 1M chars) | **~$32** | sehr gut | Free-Tier üppig |
| **ElevenLabs** | Multilingual v2 | Cloud | Business-Tarif nötig | **~$330+** | beste Qualität | Anforderung "muss nicht arsch klingen" → unrentabel |
| **Piper** | de_DE-thorsten / kerstin | Lokal CPU | gratis | **0 €** | ok-roboterhaft, sehr verständlich | Winzig (~30 MB), sehr genügsam |
| **Kokoro** | 82M-Modell | Lokal CPU/GPU | gratis | **0 €** | englisch top, deutsch eingeschränkt | Kein Vorteil ggü. Piper für DE |
| **Coqui XTTS-v2** | Multilingual | Lokal (GPU) | gratis | **0 €** | gut, Voice-Cloning | GPU empfohlen, Setup aufwändig |

---

## Verworfene Optionen (für Nachvollziehbarkeit)

| Option | Warum verworfen |
|---|---|
| **OpenAI Whisper API** ($16/Mo) | Groq Whisper ist dasselbe Modell, 3–10× günstiger und schneller |
| **OpenAI TTS-1 / TTS-1-HD** ($30–60/Mo) | Edge-TTS liefert vergleichbare Qualität für 0 €; OpenAI-Konto bewusst nicht im Stack |
| **AWS Polly Neural** ($32/Mo nach Free-Tier) | Klangqualität wie Edge-TTS, aber kostenpflichtig nach 12 Monaten; Free-Tier-Credits sinnvoller in Phase 6/11 |
| **AWS Polly Generative** ($60/Mo) | Premium-Niveau nicht gefordert |
| **AWS Transcribe** ($65/Mo) | Schlechter und teurer als Groq Whisper |
| **EC2 Self-Hosting (Piper / whisper.cpp)** | $60–725/Monat 24/7-Instanz vs. $1.80 Groq + 0 € Edge-TTS — ökonomisch nicht zu rechtfertigen. Latenz-Vorteil real nicht gegeben |
| **ElevenLabs** | Premium-Klang nicht nötig, ~$330/Monat für Bedarf zu teuer |
| **Distil-Whisper** | Englisch-only, scheitert an DE-Anteilen |

---

## Adapter-Architektur

Ziel: STT und TTS sind austauschbare Backends hinter einem stabilen Interface. Konfiguration per Env-Var oder Config-File entscheidet, welches Backend aktiv ist.

```
TTS-Adapter
  ├─ EdgeTTSBackend       (Cloud, gratis)        [Primary]
  ├─ PiperBackend         (lokal, CPU)           [Lokal-Fallback]
  ├─ AzureNeuralBackend   (Cloud, $)             [Backup-Slot, optional]
  └─ GoogleNeural2Backend (Cloud, $)             [Backup-Slot, optional]

STT-Adapter
  ├─ GroqWhisperBackend       (Cloud, $1.80)     [Primary]
  ├─ FasterWhisperBackend     (lokal CPU)        [Lokal-Fallback einfach]
  ├─ WhisperCppBackend        (lokal Vulkan/Metal) [Lokal-Fallback performant]
  ├─ DeepgramBackend          (Cloud, $)         [Backup-Slot]
  └─ ClaudeVoiceBackend       (Cloud, eingebaut) [Sonderfall: Claude-Code-/voice]
```

**Kompatibilitätsmatrix Backends × OS:**

| Backend | Win 11 | macOS (Intel) | macOS (Apple Silicon) |
|---|---|---|---|
| Edge-TTS | ✅ | ✅ | ✅ |
| Piper | ✅ | ✅ | ✅ (sogar performant) |
| Groq STT | ✅ | ✅ | ✅ |
| faster-whisper CPU | ✅ | ✅ | ✅ |
| whisper.cpp | ✅ (Vulkan) | ✅ | ✅ (Metal-Backend, sehr schnell) |

→ Setup auf Mac (Chef-Use-Case) ist eher einfacher als auf Win, dank besserer Apple-Silicon-Unterstützung in whisper.cpp.

---

## Code-Switching DE/EN — was zählt

- Whisper (lokal & Groq) erkennt Mischsprache nur, wenn Sprache auf **`auto`** steht, nicht hart auf `de`. Sonst werden englische Wörter eingedeutscht.
- Deepgram Nova-3 hat einen expliziten Multilingual-Modus.
- Für TTS irrelevant: der Output ist sprachlich kontrolliert.

---

## Zwischen-Entscheidungen (Stand 2026-04-28)

- ✅ **Architektur: Adapter-Pattern** — STT/TTS als austauschbare Backends, Config-Switch zur Laufzeit.
- ✅ **STT-Anbieter (Primary): Groq** (Whisper auf LPU, OpenAI-API-kompatibel).
- ✅ **STT-Modell: Whisper Large v3 Turbo** (~$1.80/Monat bei 45 h).
  - `language="auto"` zwingend, niemals hart `de`.
  - Eskalation auf `whisper-large-v3` (~$5/Monat) bei Qualitätsproblemen.
- ✅ **TTS-Anbieter (Primary): Microsoft Edge-TTS** (gratis, deutsche Neural-Stimmen).
  - Stimme noch zu wählen: z. B. `de-DE-KillianNeural` (m), `de-DE-KatjaNeural` (w), `de-DE-ConradNeural` (m).
  - Inoffizielle API; falls sie kippt, Switch via Adapter zu Piper (lokal) oder Azure Neural.
- ✅ **Lokal-Fallback TTS: Piper** (CPU-only, ~5–10× Realtime, läuft Win + Mac).
- ✅ **Lokal-Fallback STT: faster-whisper `medium` CPU** als einfache Variante; **whisper.cpp mit Vulkan/Metal** als performante Variante (Phase 5+, optional).
- ✅ **Cloud-Backup-Slots STT: Deepgram, Claude `/voice`** (im Adapter konfigurierbar, nicht von Tag 1 nötig).
- ✅ **Cloud-Backup-Slots TTS: Azure Neural, Google Neural2** (optional, falls Edge-TTS-API mal wegbricht).
- ❌ **Verworfen: OpenAI (STT + TTS), AWS Polly, AWS Transcribe, ElevenLabs, EC2 Self-Hosting für STT/TTS.**

---

## Offene Punkte

- [ ] Konkrete Stimm-Auswahl für Edge-TTS (Probehören: Killian / Katja / Conrad / weitere).
- [ ] Edge-TTS Stabilität im Alltag testen, bevor Setup darauf aufgebaut wird.
- [ ] Setup-Aufwand whisper.cpp + Vulkan auf Win 11 verifizieren (Doku-Recherche).
- [ ] Entscheidungen in `JARVIS_PRP.md` Tabelle 11 nachpflegen.
- [ ] Adapter-Interface skizzieren (Funktionssignaturen, Config-Format) — eigenes Doc oder direkt Code in Phase 2.
