# /watch

**🇬🇧 [English version](README.md)**

**Gib Claude die Fähigkeit, jedes Video anzusehen.**

> **Dies ist ein Fork** von Brad Bonannos hervorragendem [`claude-video`](https://github.com/bradautomates/claude-video) (MIT). Er ergänzt ein **optionales lokales Whisper-Backend** (`faster-whisper`, führt `large-v3` auf deiner GPU aus: kostenlos, privat, kein API-Key), ein optionales SABR/403-Download-Schutznetz, einen Windows-Encoding-Fix und ein paar optionale Instruktionsblöcke für Wissensdatenbanken (Obsidian-Notiz-Ausgabe, gründliche Extraktion, Kanal-Highwatermark). **Aller Verdienst für das ursprüngliche Werkzeug gebührt Brad** (siehe die Attribution ganz unten). Der Fork wird von [MinkeAutomation](https://github.com/MinkeAutomation) gepflegt.

Claude Code (empfohlen, aktualisiert sich automatisch über den Marketplace):
```
/plugin marketplace add MinkeAutomation/claude-video
/plugin install watch@claude-video
```

Codex, Cursor, Copilot, Gemini CLI oder einer von über 50 [Agent Skills](https://agentskills.io)-Hosts:
```bash
npx skills add MinkeAutomation/claude-video -g
```
(`-g` installiert global für deinen Benutzer, verfügbar über alle Projekte hinweg. Lass es weg, um pro Projekt zu installieren.)

Weitere Installationsoptionen (claude.ai Web, manuell) findest du im Abschnitt [Install](#install) weiter unten.

Kein Setup nötig zum Loslegen: `yt-dlp` und `ffmpeg` installieren sich beim ersten Lauf über `brew` auf macOS (Linux/Windows geben die exakten Befehle aus). Untertitel decken die meisten öffentlichen Videos kostenlos ab. Ein Whisper-API-Key wird nur gebraucht, wenn ein Video keine Untertitel hat.

---

Claude kann eine Webseite lesen, ein Skript ausführen, ein Repo durchstöbern. Was es von Haus aus nicht kann, ist *ein Video anzusehen*. Du fügst einen YouTube-Link ein und es muss entweder aus dem Titel raten oder ein Transkript ziehen, dem 90 % von dem fehlt, was auf dem Bildschirm passiert.

Mit Claude Video `/watch` fügst du eine URL oder einen lokalen Pfad ein, stellst eine Frage, und Claude holt zuerst die Untertitel, lädt nur herunter was es braucht, extrahiert Frames (szenenbewusst, oder schnelle Keyframes bei `efficient`-Detail), zieht ein Transkript mit Zeitmarken (kostenlose Untertitel wenn verfügbar, Whisper-API als Fallback) und liest jeden Frame per `Read` als Bild ein. Bis es antwortet, hat es das Video *gesehen* und den Ton *gehört*.

```
/watch https://youtu.be/dQw4w9WgXcQ what happens at the 30 second mark?
```

## Wofür die Leute es tatsächlich nutzen

**Fremde Inhalte analysieren.** `/watch https://youtu.be/<viral-video> what hook did they open with?` Claude schaut sich die ersten Frames an, liest das eröffnende Transkript, zerlegt die Struktur. Genauso für Werbeanzeigen, Konkurrenz-Launches, Podcast-Intros, alles wo das *Wie* genauso wichtig ist wie das *Was*.

**Einen Bug aus einem Video diagnostizieren.** Jemand schickt dir eine Bildschirmaufnahme von etwas Kaputtem. `/watch bug-repro.mov what's going wrong?` Claude schaut sich die Aufnahme an, findet den Frame in dem das Problem auftaucht, beschreibt was auf dem Bildschirm ist, erwischt oft die Ursache, ohne dass du je die Datei öffnest.

**Ein Video zusammenfassen.** `/watch https://youtu.be/<long-thing> summarize this` macht das Naheliegende: es zieht die Struktur, die Schlüsselmomente, was tatsächlich gesagt und gezeigt wurde. Schneller als in 2-facher Geschwindigkeit zu schauen.

**Den Hype aus einem Update-Video herausschneiden.** `/watch https://youtu.be/<launch-video> what's actually new - skip the hype` Dampft eine "Game-Changer"-Feature-Ankündigung auf die wenigen Dinge ein, die zählen, damit du die Substanz bekommst ohne zehn Minuten Intro und Überverkauf.

**Eine Playlist in Notizen verwandeln.** `/watch https://youtu.be/<video> summarize this to a note` Lauf es über eine Serie und lege pro Video eine Zusammenfassung ab, damit ein Kanal oder Kurs zu einem durchsuchbaren Notizsatz wird, statt zu Stunden die du absitzen musst.

## Wie es funktioniert

1. **Du fügst ein Video und eine Frage ein.** URL (alles was yt-dlp unterstützt: YouTube, Loom, TikTok, X, Instagram, plus ein paar hundert weitere) oder ein lokaler Pfad (`.mp4`, `.mov`, `.mkv`, `.webm`).
2. **`yt-dlp` prüft zuerst die Untertitel.** Bei `transcript`-Detail liefern untertitelte URLs zurück, ohne das Video herunterzuladen. Andernfalls, oder wenn Whisper Audio braucht, lädt es nur das herunter was der Lauf benötigt.
3. **`ffmpeg` extrahiert Frames im gewählten Detail.** `efficient` dekodiert nur Keyframes (nahezu sofort); `balanced`/`token-burner` bevorzugen Szenenwechsel-Frames und fallen auf den dauerbewussten gleichmäßigen Sampler zurück, wenn sie zu wenig produzieren. JPEGs sind standardmäßig 512 px breit und auf 1998 px Höhe begrenzt, für die Kompatibilität mit Claudes Read.
4. **Das Transkript kommt aus einer von zwei Quellen.** Erster Versuch: `yt-dlp` zieht native Untertitel (manuell oder automatisch generiert) von der Quelle. Kostenlos, sofort, halbwegs genau. Fallback: ein Mono-16-kHz-64-kbps-mp3-Audioclip wird extrahiert (~480 kB/Min.) und an Whisper geschickt: Groqs `whisper-large-v3` (bevorzugt, günstiger und schneller) oder OpenAIs `whisper-1`.
5. **Frames + Transkript werden an Claude übergeben.** Das Skript gibt Frame-Pfade mit `t=MM:SS`-Markern und das Transkript mit Zeitmarken aus. Claude liest jeden Frame parallel per `Read`: JPEGs werden direkt als Bilder in seinem Kontext gerendert.
6. **Claude antwortet, geerdet in dem was tatsächlich auf dem Bildschirm und im Ton ist.** Nicht "laut Beschreibung" oder "gemäß Titel". Es hat die Frames gesehen. Es hat das Transkript gehört. Es antwortet so, wie jemand der das Video geschaut hat.
7. **Aufräumen.** Das Skript gibt am Ende ein Arbeitsverzeichnis aus. Wenn du keine Nachfragen stellst, entfernt Claude es.

## Frame-Budget: warum es zählt

Die Token-Kosten werden von den Frames dominiert. Jeder Frame ist ein Bild; Bild-Tokens summieren sich schnell. Die Auto-fps-Logik des Skripts existiert, damit du dein Kontext-Budget nicht mit einem spärlichen Scan eines 30-minütigen Videos verbrennst, das mit einem gezielten 30-Sekunden-Fenster besser beantwortet worden wäre.

| Dauer | Standard-Frame-Budget | Was du bekommst |
|----------|---------------------|--------------|
| ≤30 s | ~30 Frames | Dicht: praktisch jeder Schlüsselmoment |
| 30 s - 1 min | ~40 Frames | Immer noch dicht |
| 1 - 3 min | ~60 Frames | Komfortabel |
| 3 - 10 min | ~80 Frames | Spärlich, aber brauchbar |
| > 10 min | 100 Frames (gedeckelte Modi) | "Sparse scan"-Warnung: gezielt neu laufen lassen, oder `--detail token-burner` für volle, ungedeckelte Abdeckung |

Wenn der Nutzer einen Moment benennt ("around 2:30", "the last 30 seconds", "from 0:45 to 1:00"), übergib `--start` / `--end`. Der Fokus-Modus bekommt dichtere Budgets pro Sekunde, gedeckelt bei 2 fps. Weit nützlicher als ein spärlicher Durchlauf über das Ganze.

## Frame-Deduplizierung

Die Frame-Auswahl, ob Keyframes (`efficient`), Szenenwechsel-Erkennung (`balanced`/`token-burner`) oder der gleichmäßige Sampler auf den sie zurückfällt, kann trotzdem nahezu identische Frames zutage fördern: eine Bildschirmaufnahme, die eine Folie 90 Sekunden lang hält, erzeugt ein Dutzend davon, jeder als eigenes Bild abgerechnet. Ein Dedup-Durchlauf entfernt sie, bevor die Frames Claude erreichen. Er läuft standardmäßig in jedem Frame-Modus (`--no-dedup` schaltet ihn ab):

1. Ein `ffmpeg`-Aufruf skaliert jedes extrahierte JPEG auf ein 16×16 großes Graustufen-Thumbnail. Alles danach ist reines Standard-Library-Python, keine Bild-Bibliotheken.
2. Für jeden Frame wird die **mittlere absolute Differenz** gegen den *zuletzt behaltenen Frame* berechnet (durchschnittliche Helligkeitsänderung pro Pixel, Skala 0 bis 255).
3. Liegt diese Differenz auf oder unter dem Schwellenwert (`2.0`), ist der Frame ein Beinahe-Duplikat und wird verworfen. Andernfalls wird er behalten und zur neuen Referenz.
4. Die Frame-Budget-Deckelung greift *nach* dem Dedup, damit das Budget für distinkte Frames ausgegeben wird.

Der Vergleich gegen den zuletzt *behaltenen* Frame (nicht den vorherigen) erwischt langsame Überblendungen, die nie einen Frame-zu-Frame-Schwellenwert auslösen. Der Schwellenwert ist bewusst niedrig und misst absolute Helligkeit statt Struktur, sodass ein einzeiliger Code-Diff, ein Terminal das eine Zeile scrollt, oder zwei unterschiedlich gefärbte flache Folien alle überleben.

Die **Frames**-Zeile berichtet, was zusammengefasst wurde, z.B. `6 selected from 14 candidates (… 8 near-duplicates dropped …)`. Bei ständig bewegtem Material wird nichts verworfen und du zahlst, was du ohnehin gezahlt hättest.

## Detail-Modi: gemessen

Der `--detail`-Regler tauscht Geschwindigkeit und Token-Kosten gegen visuelle Treue. Die Zahlen unten stammen aus einem echten Lauf gegen ein **49:08** langes YouTube-Video (1280×720, englische Auto-Untertitel): eine lange, überwiegend statische Bildschirmaufnahme, der Fall der die Deckelungen am härtesten belastet. Die Extraktionszeiten sind lokale CPU gegen eine bereits heruntergeladene Kopie; der einmalige Download betrug **~37 s** / 76 MB, geteilt von den drei Frame-Modi.

| Modus | Engine | Frames | Deckel | Extraktionszeit | Zeitliche Abdeckung | Geschätzte Bild-Tokens |
|------|--------|--------|-----|-----------------|-------------------|-------------------|
| `transcript` | keine (Untertitel) | 0 | - | **~4.5 s** (ein yt-dlp-Aufruf, kein Download) | vollständig (Text) | 0 (≈26.6k Text-Tokens) |
| `efficient` | Keyframe (`-skip_frame nokey`) | 50 | 50 | **~0.5 s** | 0:00 → 49:04 (vollständig) | **~9.8k** |
| `balanced` | Szenenwechsel | 100 | 100 | **~20.9 s** | 0:00 → 48:38 (vollständig) | **~19.7k** |
| `token-burner` | Szenenwechsel | 116 | ungedeckelt | **~21.0 s** | 0:00 → 48:38 (vollständig) | **~22.8k** |

- **Bild-Tokens** verwenden Anthropics `(width × height) / 750`: bei der Standardbreite von 512 px sind diese 720p-Frames 512×288, **≈197 Tokens/Frame**; `--resolution 1024` vervierfacht das grob. Das Transkript wird in jedem untertitelten Modus mitgeliefert und ist bei langen Videos oft der größere Kostenfaktor.
- **Eine Sampling-Regel über alle Frame-Modi.** Jeder erkennt alle Kandidaten über den gesamten Bereich und sampelt dann gleichmäßig (erster + letzter immer behalten) hinunter auf seinen Deckel. Die Modi unterscheiden sich nur in der Kandidaten-*Quelle* (Keyframes vs. Szenenschnitte) und im Deckel, nie darin wie die Abdeckung verteilt wird, sodass der letzte Frame immer am Ende landet, nicht mittendrin.
- **`efficient` ist die Geschwindigkeitsstufe** (~0.5 s): es rekonstruiert nur Keyframes, also ist es ~40-mal schneller als die Szenen-Modi, die jeden Frame dekodieren um Schnitte zu finden. Es kann bei bewegungsarmem Material auch *mehr* Frames liefern als `balanced` (Keyframes übertreffen Szenenschnitte in der Zahl); "efficient" heißt schnelle Extraktion, nicht weniger Frames.
- **`token-burner` weicht erst jenseits des Deckels von `balanced` ab.** Dieser Clip hatte 116 Schnitte, also sampelte `balanced` 100 und `token-burner` behielt alle 116. Bei bewegungsreichem Video mit hunderten Schnitten behält `token-burner` alles (und löst die >250-Frame-Token-Warnung aus), während `balanced` auf 100 ausdünnt.

Von einer kalten URL bis zum Ende ist `transcript` mit Abstand der günstigste Modus; die Frame-Modi addieren den geteilten ~37-s-Download zu den Extraktionszeiten oben hinzu.

## Install

| Oberfläche | Installation |
|---------|---------|
| **Claude Code** | `/plugin marketplace add MinkeAutomation/claude-video` dann `/plugin install watch@claude-video` |
| **Codex, Cursor, Copilot, Gemini CLI, +50 weitere** | `npx skills add MinkeAutomation/claude-video -g` |
| **claude.ai** (Web) | [`watch.skill` herunterladen](https://github.com/MinkeAutomation/claude-video/releases/latest) → Settings → Capabilities → Skills → `+` |
| **Manuell / dev** | `git clone` dann `skills/watch` in das Skills-Verzeichnis deines Hosts symlinken (siehe unten) |

### Claude Code

```
/plugin marketplace add MinkeAutomation/claude-video
/plugin install watch@claude-video
```

Später aktualisieren mit `/plugin update watch@claude-video`.

### Codex, Cursor, Copilot, Gemini CLI und über 50 weitere Hosts

Die [Agent Skills](https://agentskills.io)-CLI installiert den Skill in alle Agenten, die sie erkennt:

```bash
npx skills add MinkeAutomation/claude-video -g
```

`-g` installiert global für deinen Benutzer (`~/.codex/skills`, `~/.cursor/skills`, usw.); lass es weg um stattdessen ins aktuelle Projekt zu installieren. Nützliche Flags:

- `-a, --agent <names…>` - bestimmte Hosts ansteuern, z.B. `-a codex -a cursor`
- `-l, --list` - die Skills in diesem Repo auflisten ohne zu installieren
- `--copy` - Dateien kopieren statt symlinken (für Dateisysteme ohne Symlink-Unterstützung)

Die CLI entdeckt den Skill über `skills/watch/SKILL.md` und kopiert den gesamten Ordner: `SKILL.md` plus seine `scripts/`-Laufzeit, als in sich geschlossene Einheit. `SKILL.md` löst seine eigenen Skripte relativ zum Installationsort auf, sodass es auf jedem Host gleich funktioniert.

Später aktualisieren mit `npx skills update watch -g`.

### claude.ai (Web)

1. [`watch.skill` herunterladen](https://github.com/MinkeAutomation/claude-video/releases/latest) aus dem neuesten Release.
2. Gehe zu Settings → Capabilities → Skills.
3. Klick auf `+` und zieh die Datei hinein.

Aktiviere zuerst "Code execution and file creation" unter Capabilities: der Skill ruft `ffmpeg` und `yt-dlp` per Shell auf, also läuft er ohne das nicht.

### Manuell (Entwickler)

Klone das Repo und symlinke den in sich geschlossenen Skill-Ordner in das Skills-Verzeichnis deines Hosts: der Symlink hält die Installation mit deinem Arbeitsstand synchron, während du editierst:

```bash
git clone https://github.com/MinkeAutomation/claude-video.git
ln -s "$(pwd)/claude-video/skills/watch" ~/.claude/skills/watch   # or ~/.codex/skills/watch
```

Für claude.ai baue das `.skill`-Bundle aus dem Quellcode: `bash skills/watch/scripts/build-skill.sh` erzeugt `dist/watch.skill`.

## Erster Lauf

Beim ersten `/watch`-Aufruf führt der Skill `scripts/setup.py --check` aus. Wenn `ffmpeg` / `yt-dlp` nicht in deinem PATH sind oder kein Whisper-API-Key gesetzt ist, führt er dich durch die Behebung:

- **macOS** - führt automatisch `brew install ffmpeg yt-dlp` aus.
- **Linux** - gibt die exakten `apt` / `dnf` / `pipx`-Befehle aus.
- **Windows** - gibt die `winget` / `pip`-Befehle aus.
- **API-Key** - legt `~/.config/watch/.env` an (Modus `0600`) mit auskommentierten Platzhaltern für `GROQ_API_KEY` (bevorzugt) und `OPENAI_API_KEY`.

Nach dem Setup ist der Preflight still und `/watch` funktioniert einfach. Die Prüfung ist ein Lookup unter 100 ms, bremst dich bei folgenden Läufen also nicht aus.

## Transkription: zwei Varianten (lokal oder Cloud)

Untertitel decken die Mehrheit öffentlicher Videos kostenlos ab. Wenn ein Video **keine** Untertitelspur hat (lokale Dateien, TikToks, manche Vimeos, der gelegentliche untertitelfreie YouTube-Upload), transkribiert `/watch` das Audio, und du wählst **wie**:

| Variante | Was du brauchst | Kosten | Am besten für |
|---------|---------------|------|----------|
| Native Untertitel (immer zuerst versucht) | `yt-dlp` + `ffmpeg` | Kostenlos | jedes untertitelte Video |
| **A) Lokales Whisper** (Standard wenn installiert) | `pip install faster-whisper` - führt `large-v3` auf deiner GPU aus (CPU-Fallback) | **Kostenlos, privat, kein Key** | Privatsphäre, keine API-Rechnungen, offline |
| B) Cloud-Whisper - Groq | [Groq API key](https://console.groq.com/keys) - `whisper-large-v3` | Günstig, schnell | keine GPU, einfachster Start |
| B) Cloud-Whisper - OpenAI | [OpenAI API key](https://platform.openai.com/api-keys) - `whisper-1` | Standard-Preise | Fallback |
| Whisper deaktivieren | `--no-whisper` | Kostenlos, nur Frames wenn keine Untertitel | dich interessiert nur das Visuelle |

**Keine Konfiguration nötig zur Wahl:** wenn `faster-whisper` importierbar ist, nutzt `/watch` automatisch das lokale Backend (kein Key nötig). Andernfalls fällt es auf den gesetzten Cloud-Key zurück. Erzwinge eines mit `--whisper local|groq|openai`.

### Variante A - lokale Transkription (keine Cloud, kein Key)
```bash
pip install faster-whisper            # or: uv tool install faster-whisper
```
Das war's: das nächste `/watch` auf einem untertitelfreien Video transkribiert auf deiner GPU, nichts verlässt die Maschine. Feinjustierung über Umgebungsvariablen (alle optional):

| Umgebungsvariable | Standard | Zweck |
|---------|---------|---------|
| `WATCH_WHISPER_MODEL` | `large-v3` | Modellgröße |
| `WATCH_WHISPER_DEVICE` | `cuda` | `cuda` oder `cpu` |
| `WATCH_WHISPER_COMPUTE` | `float16` | z.B. `int8` um VRAM zu senken (~3 GB statt ~4.5 GB) |
| `WATCH_WHISPER_LANG` | *(auto-detect)* | einen Sprachcode erzwingen |

> Auf einer 8-GB-GPU setze `WATCH_WHISPER_COMPUTE=int8`, damit `large-v3` neben anderen Apps passt.

### Variante B - Cloud-Transkription
Setze einen Key in `~/.config/watch/.env` (`GROQ_API_KEY=` bevorzugt, oder `OPENAI_API_KEY=`). Nutze das, wenn du keine GPU hast oder das denkbar einfachste Setup willst.

## Nutzung

```
/watch https://youtu.be/dQw4w9WgXcQ what happens at the 30 second mark?
/watch https://www.tiktok.com/@user/video/123 summarize this
/watch ~/Movies/screen-recording.mp4 when does the UI break?
/watch https://vimeo.com/123 what tools does she mention?
```

Auf einen bestimmten Abschnitt fokussiert, dichteres Frame-Budget, geringere Token-Kosten:
```
/watch https://youtu.be/abc --start 2:15 --end 2:45
/watch video.mp4 --start 50 --end 60
/watch "$URL" --start 1:12:00            # from 1h12m to end
```

Weitere Stellschrauben (an `scripts/watch.py` übergeben):

- `--detail transcript|efficient|balanced|token-burner` - Treue/Geschwindigkeits-Regler. `transcript` überspringt Frames (nur Transkript); `efficient` nutzt schnelle Keyframes (Deckel 50); `balanced` nutzt szenenbewusste Frames (Deckel 100); `token-burner` ist szenenbewusst und ungedeckelt.
- `--timestamps T1,T2,…` - greif einen Frame an jeder absoluten Zeitmarke ab (`SS`/`MM:SS`/`HH:MM:SS`). Claude liest zuerst das Transkript, dann zielt es auf die Momente, die der Vortragende markiert ("look here", "as you can see"). Zusätzlich zu den Detail-Frames hinzugefügt (gegen den Deckel reserviert); Hinweise außerhalb des Fensters werden im Fokus-Modus verworfen; mit `--detail transcript` werden diese zu den einzigen Frames.
- `--max-frames N` - den Frame-Deckel für ein knapperes Token-Budget senken.
- `--resolution W` - die Frame-Breite auf 1024 px anheben, wenn Claude Bildschirmtext lesen muss (Folien, Terminals, Code).
- `--fps F` - die Auto-fps-Berechnung überschreiben (immer noch bei 2 fps gedeckelt).
- `--whisper local|groq|openai` - ein bestimmtes Whisper-Backend erzwingen. Standard: lokales `faster-whisper` bevorzugen (GPU, kostenlos, kein Key) wenn installiert, sonst Groq, sonst OpenAI.
- `--no-whisper` - Transkription komplett deaktivieren; nur Frames.
- `--no-dedup` - nahezu identische Frames behalten. Standardmäßig verwirft ein Frame-Delta-Durchlauf Frames, die visuell fast identisch zum vorherigen sind (gehaltene Folien, statische Bildschirmaufnahmen, pausiertes Video), damit das Frame-Budget für distinkte Inhalte ausgegeben wird; dieses Flag schaltet das ab.
- `--out-dir DIR` - Arbeitsdateien an einem bestimmten Ort behalten (Standard: automatisch erzeugtes tmp-Verzeichnis).

## Optional: Wissensdatenbank-Workflows

Standardmäßig beantwortet `/watch` einfach deine Frage. Zwei optionale, vault-agnostische Instruktionsblöcke verwandeln es in ein Werkzeug zur Wissenserfassung: leg einen davon in deinen Prompt oder deine `CLAUDE.md`, oder lass sie weg und `/watch` verhält sich normal:

- [`examples/obsidian-note-output.md`](examples/obsidian-note-output.md) - lege jedes Video als reichhaltige Markdown-Notiz ab: Frontmatter-Eigenschaften, eine Link-Liste mit 🔴🟡🟢-Wichtigkeitsmarkern, die Beschreibung, ein Transkript mit Zeitmarken und die eingebetteten Schlüssel-Frames.
- [`examples/thorough-extraction.md`](examples/thorough-extraction.md) - erfasse *alles* (nur-visuelle Details, Links/Assets aus Transkript + Beschreibung + Bildschirm-Frames) und **markiere aktiv, was sich nicht auflösen ließ** (abgeschnittene Links, gesperrte Assets, referenzierte aber fehlende Repos/Downloads), damit nichts still durchrutscht.
- [`examples/channel-highwatermark.md`](examples/channel-highwatermark.md) - folge einem *Kanal* inkrementell: halte einen Highwatermark pro Kanal und pro Plattform, damit "fetch die neuesten von X" nur echte Neu-Uploads verarbeitet, ohne je etwas erneut zu ziehen oder zu übersehen.

## Grenzen

- **Die Genauigkeit bei langen Videos hängt vom Detail-Modus ab.** In den gedeckelten Modi (`efficient`, Standard `balanced`) dünnt die Abdeckung jenseits von ~10 Minuten aus: der Frame-Deckel verteilt sich über den ganzen Clip, also gibt das Skript eine "sparse scan"-Warnung aus und du fährst besser damit, gezielt mit `--start`/`--end` neu zu laufen. `token-burner` hebt den Deckel auf und behält *jeden* Szenenwechsel-Frame über das ganze Video, bleibt also bei längeren Clips vollständig, auf Kosten von mehr Bild-Tokens. Die 10-Minuten-Marke ist eine Richtlinie für die gedeckelten Modi, keine harte Obergrenze.
- **Detail ist ein Regler.** Die Standardwerte sind ausgewogen: szenenbewusste Frames, max. 2 fps, 100-Frame-Deckel. Nutze `--detail efficient` für einen schnellen 50-Frame-Keyframe-Durchlauf, oder `--detail token-burner` für ungedeckelte Szenenkandidaten. Setze `WATCH_DETAIL` in `~/.config/watch/.env`, um den Standard zu ändern.

## Struktur

```
.
├── skills/watch/                 # self-contained skill - copied as a unit by every installer
│   ├── SKILL.md                  # skill contract - the source of truth across all surfaces
│   └── scripts/
│       ├── watch.py              # entry point - orchestrates download → frames → transcript
│       ├── download.py           # yt-dlp wrapper
│       ├── frames.py             # ffmpeg frame extraction + auto-fps logic
│       ├── transcribe.py         # VTT parsing + dedupe + Whisper orchestration
│       ├── whisper.py            # Groq / OpenAI clients (pure stdlib)
│       ├── config.py             # shared config (~/.config/watch/.env)
│       ├── setup.py              # preflight + installer
│       └── build-skill.sh        # build dist/watch.skill for claude.ai upload (dev-only)
├── hooks/                        # SessionStart status hook (Claude Code only)
├── .claude-plugin/               # plugin.json + marketplace.json (Claude Code)
├── .codex-plugin/                # plugin.json - Codex/agents manifest ("skills": "./skills/")
├── .agents/plugins/              # marketplace.json - Agent Skills marketplace listing
├── AGENTS.md → CLAUDE.md         # generic-agent entry point
├── tests/                        # pytest suite (ffmpeg-synthesized clips, no network)
└── .github/workflows/            # release.yml - auto-builds watch.skill on tag push
```

## Entwickeln

```bash
# Run the test suite (stdlib + pytest; ffmpeg required for frame tests):
python3 -m pytest -q

# Build the claude.ai upload bundle:
bash skills/watch/scripts/build-skill.sh      # → dist/watch.skill
```

Release: tagge `vX.Y.Z`, push den Tag. Der Workflow baut `dist/watch.skill` und hängt es an das GitHub-Release. Halte die Version synchron über `skills/watch/SKILL.md`, `.claude-plugin/plugin.json` und `.codex-plugin/plugin.json`.

Die Versionshistorie findest du in [CHANGELOG.md](CHANGELOG.md).

## Open Source

MIT-Lizenz.

Gebaut auf `yt-dlp`, `ffmpeg` und Claudes multimodalem `Read`-Werkzeug. Whisper-Transkription über [Groq](https://groq.com) oder [OpenAI](https://openai.com).

Gebaut von Brad Bonanno: ich mache Content über das Bauen mit KI auf [YouTube (@bradbonanno)](https://www.youtube.com/@bradbonanno) und baue KI-Betriebssysteme für Unternehmen bei [Solaris Automation](https://www.solarisautomation.io/). Wenn `/watch` dir das Durchspulen eines Videos erspart, komm auf dem Kanal vorbei und sag hallo.

---

[github.com/MinkeAutomation/claude-video](https://github.com/MinkeAutomation/claude-video) · [@bradbonanno](https://www.youtube.com/@bradbonanno) · [Solaris Automation](https://www.solarisautomation.io/) · [LICENSE](LICENSE)
