# Changelog

**🇬🇧 [English version](CHANGELOG.md)**

Alle nennenswerten Änderungen an `/watch` sind hier dokumentiert.

## [Unreleased] - MinkeAutomation-Fork

### Hinzugefügt
- **Lokales Whisper-Backend (Variante A).** `--whisper local` führt `faster-whisper` aus (`large-v3`, GPU `float16`, CPU/`int8`-Fallback, VAD): kostenlos, privat, kein API-Key, nichts verlässt die Maschine. Wenn `faster-whisper` importierbar ist, wird es zum **Standard**-Backend (`resolve_backend` bevorzugt local, sonst Groq/OpenAI). Justierbar über `WATCH_WHISPER_MODEL/DEVICE/COMPUTE/LANG`. Der Cloud-Pfad bleibt unverändert, rein additiv.
- `setup.py` zählt jetzt eine lokale `faster-whisper`-Installation als gültiges Backend, sodass keylose lokale Installationen `ready` statt `needs_key` melden.
- `examples/obsidian-note-output.md` - optionaler, vault-agnostischer Instruktionsblock, um die `/watch`-Ausgabe als strukturierte Markdown-Notiz abzulegen (Eigenschaften, 🔴🟡🟢-Link-Marker, Beschreibung, Transkript mit Zeitmarken, eingebettete Schlüssel-Frames). Der Kern-Skill bleibt allgemein.
- `examples/thorough-extraction.md` - optionale Instruktion für den Wissensdatenbank-Einsatz: nur-visuelle Details erfassen, Links/Assets aus Transkript + Beschreibung + Bildschirm-Frames extrahieren und aktiv alles Ungelöste markieren (abgeschnittene Links, gesperrte Assets, referenzierte aber fehlende Repos/Downloads), damit nichts still durchrutscht.
- `examples/channel-highwatermark.md` - optionale Instruktion für inkrementelles Kanal-Folgen: einen Highwatermark pro Kanal und pro Plattform halten, damit "fetch die neuesten von X" nur echte Neu-Uploads verarbeitet.
- README: dokumentiert beide Transkriptions-Varianten (lokal / Cloud) und die optionale Notiz-Ausgabe.
- **SABR / 403-Schutznetz** (aus Taelo Kims Fork). `download.py` erzwingt Nicht-Web-Player-Clients (`--extractor-args youtube:player_client=tv,web_safari,mweb`) beim Video-Download und Untertitel-Abruf, mit optionalen Browser-Cookies über `WATCH_COOKIES_BROWSER`. Modernes yt-dlp bewältigt SABR meist von selbst, das hier ist also eine Doppel-Absicherung für den Fall, dass YouTube wieder anzieht: harmlos, wenn nicht gebraucht.

### Behoben
- **Windows-Absturz bei der Report-Ausgabe.** `watch.py` erzwingt jetzt UTF-8 auf stdout/stderr. Auf einer cp1252-Windows-Konsole lösten die Pfeile/Emoji des Reports (z.B. der `→` in der Fokus-Bereichs-Ausgabe mit `--start/--end`) einen `UnicodeEncodeError` aus und brachen den Lauf ab, nachdem die Frames bereits extrahiert waren.

## [0.2.0] - 2026-06-29

### Hinzugefügt
- **`--detail`-Regler** mit vier Modi: `transcript` (nur Untertitel, keine Frames), `efficient` (schneller Keyframe-Durchlauf, Deckel 50), `balanced` (szenenbewusst, Deckel 100, Standard) und `token-burner` (szenenbewusst, ungedeckelt). Setze den Standard mit `WATCH_DETAIL` in `~/.config/watch/.env`.
- **Frame-Deduplizierung** (standardmäßig an; `--no-dedup` zum Deaktivieren). Vor der Budget-Deckelung skaliert ein Durchlauf jeden Frame auf ein 16×16-Graustufen-Thumbnail herunter und verwirft Frames, deren mittlere Pixel-Differenz zum zuletzt *behaltenen* Frame innerhalb des Schwellenwerts liegt, sodass das Budget an distinkte Inhalte statt an gehaltene Folien und statische Aufnahmen geht. Die **Frames**-Report-Zeile zeigt, wie viele Beinahe-Duplikate verworfen wurden.
- **Whisper-Auto-Chunking.** Audio über dem 25-MB-Upload-Deckel wird in gleich große Stücke geteilt, pro Stück transkribiert, wobei die Segment-Zeitmarken zurück in die Quellzeit verschoben werden. Teil-Ausfälle werden toleriert: die Transkription scheitert nur, wenn *jedes* Stück scheitert, sodass die Länge allein sie nicht mehr sprengt.
- **`--timestamps T1,T2,…`** - greif einen Frame an jeder absoluten Zeitmarke ab; gegen den Deckel reserviert und die einzigen Frames, die unter `--detail transcript` erzeugt werden.
- **`--no-whisper`** - Transkription komplett deaktivieren (nur Frames).
- pytest-Suite, die config, dedup, download, fixtures, frames, setup, timestamps, watch und whisper abdeckt (kein Netzwerk; ffmpeg-synthetisierte Clips).

### Geändert
- **Umstrukturiert in ein in sich geschlossenes `skills/watch/`-Paket**, sodass `SKILL.md` und seine `scripts/`-Laufzeit Geschwister in einem Ordner sind. Das behebt Installationen auf Codex, Cursor, Copilot und anderen Agent-Skills-Hosts: `npx skills add` kopiert den Skill jetzt als funktionierende Einheit, statt die Root-`SKILL.md` ohne ihre Skripte zu greifen.
- **Harness-agnostische Pfadauflösung** - `SKILL.md` löst `$SKILL_DIR` von dort auf, wo es gelesen wurde, statt vom nur unter Claude Code verfügbaren `${CLAUDE_SKILL_DIR}`, sodass Skript-Aufrufe auf jedem Host funktionieren.
- `/watch` wird jetzt aus dem `SKILL.md`-Frontmatter abgeleitet; der separate `commands/watch.md`-Wrapper wurde entfernt, um einen doppelten Slash-Command zu vermeiden.
- `balanced` dekodiert jetzt vollständig, um jeden Szenenschnitt über das ganze Video zu erkennen. Der vorherige Early-Exit war schneller, behielt aber nur die ersten Schnitte und verwarf den Schluss langer Videos.
- `token-burner` ist von der "sparse scan"-Warnung für lange Videos ausgenommen, da es jeden Szenenwechsel-Frame behält.
- `--max-frames` ist jetzt ein Override auf den Standard-Deckel jedes Modus, statt eines festen Standardwerts von 80.

### Behoben
- Nicht-Claude-Installationen (`npx skills add`) waren von Anfang an tot: der Installer kopierte `SKILL.md` ohne die `scripts/`, die es per Shell aufruft. Das in sich geschlossene Paket-Layout löst das.

### Entfernt
- Die Planungsdokumente `V2_PLAN.md` und `V2_CONCERNS.md`.

## [0.1.3] - 2026-05-09

### Behoben
- Windows: `video.info.json` wird als UTF-8 gelesen (#4). Zuvor griff `Path.read_text()` unter Windows standardmäßig auf cp1252 zurück und stürzte bei yt-dlps UTF-8-Ausgabe ab, wobei Titel/Uploader still aus dem Report fielen. Derselbe Fix wurde auf die `.env`-Lese-/Schreibvorgänge in `whisper.py` und `setup.py` angewendet.
- `download.py` protokolliert info.json-Parse-Fehler jetzt nach stderr, statt sie zu verschlucken.

### Sicherheit
- Subprozess-argv gegen Option-Injection gehärtet (#2): `--` vor der URL im yt-dlp-argv eingefügt und `is_url` verschärft, um `-`-präfixierte Quellen abzulehnen und eine nicht-leere netloc zu verlangen. Video-/Audio-Pfade vor der Übergabe an `ffmpeg`/`ffprobe` per `Path.resolve()` in absolute aufgelöst, sodass ein relativer Pfad, der mit `-` beginnt, nicht als Flag missdeutet werden kann.

## [0.1.2] - 2026-04-24

### Behoben
- Windows-Konsolen-Absturz: das Emoji aus der Langvideo-Warnung in `watch.py` entfernt; cp1252-Konsolen konnten es nicht kodieren.
- `setup.py` gibt unter Windows jetzt `winget` / `pip`-Installationsbefehle aus statt "unsupported platform", passend zu dem, was die README bereits versprach.

### Geändert
- `SKILL.md` weist darauf hin, dass die Skripte unter Windows mit `python` aufgerufen werden müssen, nicht mit `python3` (letzteres ist der Microsoft-Store-Stub unter Windows).

## [0.1.1] - 2026-04-24

### Behoben
- `commands/watch.md`-Shim hinzugefügt, damit `/watch` aufrufbar ist, wenn es als Claude-Code-Plugin installiert wird. Ohne ihn lud das Plugin, aber der Skill wurde nicht als Slash-Command bereitgestellt.
- `scripts/build-skill.sh` entfernt jetzt `commands/` aus dem claude.ai-`.skill`-Bundle, neben `hooks/` und `.claude-plugin/`.

## [0.1.0] - 2026-04-24

Erstes Marketplace-Release.

### Hinzugefügt
- `/watch <url-or-path> [question]`-Slash-Command.
- yt-dlp-Download mit nativer Untertitel-Extraktion (manuell + Auto-Subs).
- ffmpeg-Frame-Extraktion mit auto-skalierter fps (≤2 fps, ≤100 Frames, dauerbewusstes Budget).
- `--start` / `--end`-Fokus-Modus mit dichterem Frame-Budget und Transkript-Bereichsfilterung.
- Whisper-Fallback (Groq bevorzugt, OpenAI sekundär) für Videos ohne Untertitel.
- `setup.py`-Preflight: stilles `--check`, strukturiertes `--json` und ein Installer, der auf macOS automatisch `brew install` ausführt.
- Session-Start-Hook, der beim ersten Lauf / bei Teil-Konfiguration eine einzeilige Statusmeldung ausgibt.
- `.skill`-Bundle-Verpackung für den claude.ai-Upload über `scripts/build-skill.sh`.
