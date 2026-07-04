# Optional: incremental channel fetch (highwatermark)

`/watch` handles one URL at a time. When you follow a *channel* and want "just get me what's new since last time" — without re-processing videos you already have — keep a **highwatermark**: a note of the last video you fetched per channel. Paste the instruction block below into your prompt or `CLAUDE.md`.

Platform-agnostic: works for any channel `yt-dlp` can list (YouTube, TikTok, Vimeo, X, and more). Optional — skip it and `/watch` just fetches whatever URL you give it.

---

## Copy-paste instruction

> Keep a **fetch highwatermark** per channel so "fetch the newest from \<channel\>" only processes genuinely new videos.
>
> Store, per channel, a small marker: **platform · channel URL · newest fetched video (title + ID) · its upload date.** A creator on several platforms gets **one marker line per platform**.
>
> When asked to fetch the newest from a channel:
> 1. **Read the marker** for that channel + platform.
> 2. **List the channel's uploads** newest-first: `yt-dlp --flat-playlist --print "%(upload_date)s %(id)s %(title)s" <channel-url>`.
> 3. **Diff against the marker:** keep only uploads newer than the marker date (or whose ID isn't already recorded). Everything at or below the marker is already done — skip it.
> 4. **Process each new one** with `/watch` (oldest→newest), then
> 5. **Move the marker** to the now-newest fetched video.
>
> This prevents both double-fetching and silently missing uploads. If no marker exists yet, treat the most recent 1-2 uploads as the starting point and set the marker.

---

## Why per-platform
The same creator often posts different cuts to YouTube, TikTok, and X. Tracking one marker per platform means "newest from their TikTok" and "newest from their YouTube" advance independently — you never re-pull one because the other moved.
