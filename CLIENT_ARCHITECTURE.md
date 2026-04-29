# Client-Architektur — Mobile-App, PWA & Backend

> Arbeitsdokument zur Entscheidungsfindung. Stand: 2026-04-29.
> Frage: *Wie sieht die Client-Seite von Jarvis aus, mit der ich vom Handy spreche, den Agent-Desktop streame und ihn konfiguriere — und die später an andere Personen verteilbar ist?*

---

## TL;DR

**Backend zuerst, dann PWA, dann native Android-App. Keine Entweder-Oder-Wahl, sondern eine Reihenfolge.**

| Komponente | Rolle | Tech | Phase (in [`JARVIS_PRP.md`](./JARVIS_PRP.md)) |
|---|---|---|---|
| **Backend (Jarvis-Host)** | Einzige Schnittstelle für alle Clients | FastAPI auf dem Windows-Host | 5 / 6 |
| **PWA** | Sekundäre View für Browser-Geräte (TV, iPad, fremdes Notebook) | Svelte oder Next.js, vom Host ausgeliefert | 5 / 6 |
| **Native Android-App** | Primärer Daily-Driver, Voice-First | Kotlin + Jetpack Compose | 6+ |

PWA und Native-App sprechen mit demselben Backend — sie sind nur zwei Sichten auf dieselbe Maschine.

---

## Anforderungen, abgeleitet aus der PRP

Aus [`JARVIS_PRP.md`](./JARVIS_PRP.md) Section 1 + 3 (Workflows):

1. **Voice-First** — Hauptschnittstelle ist Sprache, beim Rumlaufen in der Wohnung.
2. **PTT auch bei Display aus** — Hardware-Trigger (Volume-Taste, Headset-Multifunktionstaste) muss funktionieren, ohne dass eine App im Vordergrund ist. Sonst wird Jarvis im Alltag nicht benutzt.
3. **Live-Stream-View** des Agent-Desktops aufs Handy (Phase 6, noVNC).
4. **Modus-Toggles** PTT ↔ Wake-Word, Status-Anzeige, Quick-Commands.
5. **Notifications** (Bestätigungen, Push-Pings, ntfy-artige Meldungen).
6. **Remote-fähig** — funktioniert über Tailscale, nicht nur im Heim-WLAN.
7. **Verteilbar** an Familie / Freunde / später ggf. Kunden.
8. **Privacy-First** — keine Telemetrie an Dritte, alles über eigene Hardware via Tailscale.
9. **Wartbarkeit** über mehrere Jahre, möglichst kleine Angriffsfläche bei Android-Updates.

---

## Optionen — Vergleich

| Ansatz | Stärken | Schwächen | Verteilung |
|---|---|---|---|
| **PWA-only** (Svelte/Next im Browser, "zum Startbildschirm hinzufügen") | Eine Codebase, zero Install-Friction, sofort plattformübergreifend (iOS, Android, Desktop, TV), Updates per F5 | **Kein Hardware-PTT bei Display aus**, kein Bluetooth-Headset-Button als Trigger, brüchige Audio-Sessions im Hintergrund, keine Quick-Settings-Tile / Lockscreen-Integration | URL teilen |
| **Capacitor / Tauri Mobile** (PWA in nativer Hülle) | PWA-Komfort + native Plugins für die kritischen Sachen, optional iOS später | Hardware-PTT-Plugins sind oft halbgare Drittanbieter-Pakete, Bluetooth-Audio-Routing durch Web-Layer instabil, Wear-OS / Android Auto / Tasker schwer/gar nicht integrierbar | APK / Stores |
| **Flutter** (Dart, Cross-Platform Android+iOS) | Eine Codebase iOS+Android, ordentliche UI-Performance, gute Audio-Plugins | Audio + Bluetooth weniger nativ als Kotlin, Dart als Sprache rein für dieses Projekt unverhältnismäßig | Stores |
| **React Native / Expo** (TypeScript) | Großes Ökosystem, JS/TS-vertraut für Web-Devs, viele Mic-Module | Audio-Stack qualitativ unter Kotlin, JS-Bridge-Overhead, Expo-Limits bei tiefen Hooks | Stores / EAS |
| **Native Android (Kotlin + Compose)** | **Beste Audio-Pipeline**, Hardware-Buttons trivial, ForegroundService mit Mic sauber, Quick-Settings-Tile, Lockscreen-Controls, Wear-OS-Companion möglich, langlebigster offizieller Stack | Lernkurve (wenn Kotlin neu), kein iOS, Update-Treadmill jährlich, App-Distribution asymmetrisch zum Server | APK-Sideload + ggf. Play Store |
| **Native iOS (Swift)** | Saubere AirPods-Integration | Kein Hauptbedarf — User hat Android, iOS-Distribution erst sehr viel später | App Store |
| **Bestehende Apps stückeln** (Unified Remote + Browser + ntfy) | Sofort einsatzbereit, null Code | Kein einheitliches UI, drei Icons, nicht verteilbar | nicht möglich |

---

## Entscheidung: Native Android primär, PWA sekundär — gegen ein gemeinsames Backend

### Warum Native für den Daily-Driver

Jarvis ist Voice-First. Genau dort versagt die PWA an drei Stellen, die nicht nice-to-have sind, sondern Kern-Features:

1. **Hardware-PTT bei Display aus.** Volume-Taste-lange-drücken oder Bluetooth-Headset-Multifunktionstaste sind die einzigen UX-vertretbaren Trigger im Realbetrieb. Handy aus der Tasche ziehen, entsperren, in den Browser navigieren, Touch-Button finden → Voice-First-Versprechen verbrannt.
2. **Bluetooth-Audio + Mic-Routing.** Der wackeligste Teil jeder mobilen Voice-Pipeline. Browser-Audio-APIs sind hier historisch unzuverlässig, native `MediaRecorder` + `AudioManager` mit Audio-Focus-Handling sind der robuste Weg.
3. **ForegroundService mit "Jarvis hört zu"-Notification.** Privacy- und Sicherheits-Signal, gleichzeitig technische Voraussetzung für Hintergrund-Mic seit Android 14. Nicht kompromissfähig im Wake-Word-Modus.

Zwischenlösungen (Capacitor mit Mic-Plugins) lösen 1–3 nominell, aber bei einem Voice-First-Produkt willst du dort nicht "passabel", sondern "tadellos". Außerdem fallen Folge-Features (Quick-Settings-Tile, Wear-OS, Android Auto) weg.

### Warum trotzdem auch eine PWA

Es gibt Geräte und Situationen, wo Native-Install nicht passt:
- TV-Browser, um auf den Agent-Desktop zu schauen
- iPad / Mac vom Chef
- Geliehenes Tablet, fremdes Handy, Notebook im Café
- iOS-User später, ohne dass eine zweite native App gepflegt werden muss

Für **Read-only-Stream + Status + Touch-Toggles** reicht eine 1-Wochenende-PWA, die vor demselben Backend sitzt. Niemand erwartet PTT vom TV-Browser. Verlust gegenüber der Native-App: nur die Voice-First-Funktionen — die werden auf diesen Geräten ohnehin nicht gebraucht.

### Warum die Reihenfolge Backend → PWA → Native

- **Backend zuerst** ist sowieso Pflicht, egal welcher Client. Klare Endpoints abstrahieren die Client-Wahl.
- **PWA als erster Client** ist in 1–2 Wochenenden gebaut und gibt dir sofort Stream-View und Toggles aufs Handy. Damit benutzt du Jarvis im Alltag und sammelst Daten, *was wirklich fehlt*.
- **Native Android dann zielgerichtet**: Nach 2–4 Wochen Praxis weißt du exakt, welche PTT-Variante (Volume-Taste, Headset-Button, Lockscreen-Geste) sich richtig anfühlt. Ohne diese Daten zu nativ zu gehen heißt: vermutlich falsch optimieren.
- Keine Arbeit ist verschwendet — die PWA bleibt parallel als zweite Sicht.

---

## Architektur-Skizze

```
                                    ┌─── Native Android (Kotlin + Compose)
                                    │     • PTT (Volume-Long, Headset-Button)
                                    │     • ForegroundService (Mic)
                                    │     • noVNC-WebView eingebettet
                                    │     • Push-Notifications (FCM oder ntfy)
                                    │     • Quick-Settings-Tile (Modus-Toggle)
                                    │
   Jarvis-Host  ───── HTTPS / WS ───┤
   (Tailscale)                      │
                                    └─── PWA (Svelte/Next, statisch ausgeliefert)
                                          • Stream-View (noVNC iframe)
                                          • Status, Modus-Toggles, Quick-Commands
                                          • zero-install, jeder Browser
                                          • Onboarding-QR für die Native-App


   Jarvis-Host (Docker-Stack auf Windows)
   ┌──────────────────────────────────────────────────┐
   │  FastAPI-Backend  (Port 8080)                    │
   │   • POST /audio        → STT → tmux send-keys    │
   │   • POST /mode         → jarvis-mode.ps1         │
   │   • GET  /status       → tmux-Buffer, Services   │
   │   • POST /command      → Quick-Command an Claude │
   │   • WS   /events       → Live-Status, Push       │
   ├──────────────────────────────────────────────────┤
   │  Static-Files: PWA-Bundle  (Port 8080/app)       │
   ├──────────────────────────────────────────────────┤
   │  noVNC  (Port 6080)  — Agent-Desktop Stream      │
   ├──────────────────────────────────────────────────┤
   │  Claude Code in tmux-Session "jarvis"            │
   │  Agent-Desktop-Container (xfce + xvfb + VNC)     │
   └──────────────────────────────────────────────────┘
```

**Datenfluss "PTT in der Küche":**

1. Volume-Taste lange drücken am Handy → Native-App reagiert, ForegroundService startet Mic.
2. App nimmt auf, sendet Audio per HTTPS POST an `/audio` (über Tailscale).
3. Backend pipet zur STT-Pipeline (Groq Whisper Turbo, siehe [`STT_TTS_OPTIONS.md`](./STT_TTS_OPTIONS.md)).
4. Transkript via `tmux send-keys -t jarvis "<text>" Enter` an Claude Code.
5. Claude verarbeitet, antwortet ggf. per TTS (Edge-TTS) → Backend pusht via WS-Event an die App.
6. App spielt TTS ab, Notification erscheint.

---

## Tech-Stack — Defaults

### Backend

| Punkt | Wahl | Begründung |
|---|---|---|
| Sprache / Framework | **FastAPI (Python)** | Passt zur STT-Pipeline (Groq-Client ist eh Python), schnelle Iteration, gute Dev-Tools |
| Async / WebSockets | FastAPI nativ | Reicht für Status-Push, kein zusätzliches Framework nötig |
| Auth (initial) | **Tailscale-only** | Netzwerk-Auth, keine Logins. Backend lauscht nur auf Tailscale-Interface. |
| Auth (Verteilung) | Bearer-Token + QR-Onboarding | Wenn die App an Dritte verteilt wird, der Host generiert beim ersten Start einen Token und zeigt ihn als QR. |
| Verpackung | **Docker-Compose-Stack** | Ein-File-Deployment für die Verteilung, gleiche Umgebung beim Empfänger |

### PWA

| Punkt | Wahl | Begründung |
|---|---|---|
| Framework | **SvelteKit** | Schlank (~30 KB Bundle), schnelle DX, eine Solo-Dev-freundliche Wahl. Alternative: Next.js, wenn React vertrauter ist. |
| PWA-Installation | Manifest + Service Worker | Standardweg, "zum Startbildschirm hinzufügen" |
| Stream-View | `<iframe>` auf noVNC mit `?view_only=true` | Trivialer Embed, kein eigenes WebRTC |
| Push | Web Push (iOS 16.4+, Android, Desktop alle Browser) | Plattformübergreifend ausreichend |

### Native Android

| Punkt | Wahl | Begründung |
|---|---|---|
| Sprache | **Kotlin** | Offizieller Android-Stack, langlebig, beste Tooling-Unterstützung |
| UI | **Jetpack Compose** | 2026 ausgereift, deklarativ wie React/Svelte, sehr viel produktiver als XML-Layouts |
| Min-SDK | API 29 (Android 10) | Deckt 99 % der Geräte ab, vermeidet Legacy-Permission-Modelle |
| Mic-Aufnahme | `MediaRecorder` oder `AudioRecord` in `ForegroundService(microphone)` | Pflicht ab Android 14 für Hintergrund-Mic |
| PTT-Trigger | Volume-Long-Press via `AccessibilityService` **oder** dedizierter Hardware-Key-Receiver für Headset-Buttons | Praxistest entscheidet, was sich besser anfühlt |
| Stream-Embed | `WebView` mit eingebettetem noVNC | Wiederverwendung der Web-Lösung |
| Networking | Ktor-Client oder OkHttp + `kotlinx.serialization` | Standard, schnell |
| Push | **ntfy** (selbst-gehostet) bevorzugt vor FCM | Konsequent Privacy-First, kein Google-Pfad nötig. Ntfy-Service kann im selben Docker-Stack laufen. |

---

## Verteilung an andere

### Modell

Jeder Empfänger betreibt seinen **eigenen Jarvis-Host**. Single-Tenant, kein Multi-User-Backend.

Pakete pro Empfänger:

1. **Docker-Compose-Stack** (Server) — enthält Backend, noVNC, Agent-Desktop-Container, PWA-Static-Files. Ein `docker compose up -d` startet alles.
2. **APK** (Native Client) — Sideload anfangs, später Play Store.
3. **Onboarding** — User installiert APK, scannt QR-Code, der vom Host beim ersten Start ausgegeben wird (enthält Tailscale-URL + Bearer-Token). PWA dient als QR-Anzeige-Tool.

### Distributionswege Native-App

| Weg | Aufwand | Zielgruppe |
|---|---|---|
| **APK über Tailscale-Host oder GitHub Release** | sehr niedrig | Familie, Freunde, Beta-Tester |
| **Play Store** | 25 € einmalig + Review (1–3 Tage initial) | Wenn App stabil, Reichweite gewünscht |
| **F-Droid** | aufwändig (längerer Review, Open-Source-Pflicht) | Privacy-Community, später relevant |

PWA-Distribution ist trivial: vom Jarvis-Host ausgeliefert, "URL aufrufen, zum Homescreen hinzufügen".

### Pflege

- **Server-Updates** über `docker pull` + `docker compose up -d` beim Empfänger.
- **App-Updates** über Store oder neuer APK-Download.
- Backend-API muss versioniert werden (z. B. `/v1/audio`), damit ältere App-Versionen mit neueren Backends weiterleben.

---

## Risiken & Caveats

### Technisch

- **Bluetooth-Audio-Routing auf Android wird der hässlichste Teil des Projekts**, nicht das Streaming, nicht die LLM-Integration. 2–3× Zeit-Puffer einplanen.
- **Permission-Reviews + ForegroundService-Regeln** ändern sich mit jedem Android-Major-Release. Pro Jahr ~1 Wochenende Wartung einplanen.
- **AccessibilityService für Volume-Long-Press** ist mächtig, aber Play-Store-sensibel. Google verlangt eine begründete Privacy-Policy-Erklärung; alternativ Volume-Tasten nur bei aktiver App, dann ist's egal.
- **noVNC im WebView** auf Android frisst Akku, wenn dauerhaft offen. Lifecycle sauber pausieren / fortsetzen.

### Strategisch

- **Wartung über 5 Jahre** ist bei Native-Android höher als bei reiner PWA. Realistischer Posten, der bei Solo-Devs gerne unterschätzt wird.
- **Verteilungs-Asymmetrie** — Server-Update + App-Update gleichzeitig orchestrieren ist nervig. API-Versionierung früh ernst nehmen.
- **iOS-Lücke** — solange iOS-Native nicht gebaut wird, ist die PWA der einzige iOS-Pfad. Damit muss die PWA für iOS-User "gut genug" sein. Bedeutet: PWA darf nicht nur "Stream-Viewer" werden, sondern muss zumindest Touch-PTT (mit allen Limits) anbieten.

---

## Phasen-Mapping

Ergänzt die Phasen aus [`JARVIS_PRP.md`](./JARVIS_PRP.md) Section 8 / 10:

| Phase | Client-Arbeit |
|---|---|
| **Phase 5** (Zwei-Wege-Sprache + Wake-Word) | Backend-Skeleton (FastAPI) anlegen, `/audio`, `/mode`, `/status` minimal funktionsfähig. PWA-Skelett mit Status-Page. |
| **Phase 6** (Agent-Desktop) | noVNC-Embed in PWA, Stream-Live-View. PWA als "Reinschau-Tool" produktiv. |
| **Phase 6.5 (neu)** | Native Android-App, Iteration 1: PTT (Volume-Long-Press), Mic-Capture, Backend-Anbindung, noVNC-WebView. Erste APK an User selbst. |
| **Phase 7+** | Quick-Settings-Tile, Headset-Button-Mapping, Push-Notifications, Onboarding-QR-Code. Distribution erste Beta-Tester. |
| **Phase 8+** | Play Store, Multi-Host-Switching (mehrere Jarvis-Instanzen pro App), Wear-OS-Companion. |

---

## Offene Punkte

Zur Übernahme in [`OPEN_QUESTIONS.md`](./OPEN_QUESTIONS.md):

- [ ] **Backend-Auth-Modell** für die Verteilungs-Phase festlegen (Bearer-Token + QR vs. mTLS vs. nur Tailscale).
- [ ] **ntfy-Push vs. FCM** entscheiden — Privacy-First spricht für ntfy, FCM wäre der Standard-Weg auf Play-Store-Apps.
- [ ] **PTT-Trigger-Hierarchie** im Praxistest klären: Volume-Long-Press, Headset-Multifunktionstaste, Lockscreen-Geste — welche fühlt sich am besten an?
- [ ] **PWA-Framework**: SvelteKit (Empfehlung) oder Next.js?
- [ ] **API-Versionierung** früh festlegen (`/v1/...`), damit Server- und App-Updates entkoppelbar bleiben.
- [ ] **Onboarding-QR-Format** standardisieren (URL + Token in einem QR, ähnlich Tailscale-Auth-URLs).

---

## Verworfene Optionen — Kurzbegründungen

| Option | Warum nicht |
|---|---|
| **PWA-only** | Kompromittiert Voice-First-Kern (Hardware-PTT, Bluetooth-Audio, Hintergrund-Mic). |
| **Capacitor** | Hardware-PTT-Plugins instabil, Bluetooth-Audio über Web-Layer brüchig, blockiert spätere Wear-OS-/Android-Auto-Integration. |
| **Flutter** | Audio-Stack unter Kotlin, Dart als Zweitsprache nur für dieses Projekt unverhältnismäßig. |
| **React Native / Expo** | JS-Bridge-Overhead bei Audio, Expo-Limits bei tiefen Hooks (AccessibilityService, Quick-Settings-Tile). |
| **Native iOS zuerst** | User hat Android. iOS später, wenn Distribution ernsthaft Richtung Apple-Welt geht. |
| **App-Stückwerk** (Unified Remote + Browser + ntfy) | Kein einheitliches UI, drei Icons, nicht verteilbar, kein Onboarding-Pfad. |

---

## Referenzen

- [`JARVIS_PRP.md`](./JARVIS_PRP.md) — Zielbild, Architektur, Phasen.
- [`STT_TTS_OPTIONS.md`](./STT_TTS_OPTIONS.md) — STT/TTS-Backend-Wahl, Adapter-Pattern.
- [`CLAUDE.md`](./CLAUDE.md) — Voice-Sicherheitsregeln (Ebene 1), gelten für alle Clients.
- [`OPEN_QUESTIONS.md`](./OPEN_QUESTIONS.md) — Sammelort für noch zu klärende Punkte.
