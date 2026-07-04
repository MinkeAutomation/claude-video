# Optional: thorough capture + missing-asset follow-up

By default `/watch` samples frames and answers your question. If you're using it to **build a knowledge base** — where *missing* a diagram, a repo link, or a config shown on screen is a real loss — paste the instruction block below into your prompt (or your `CLAUDE.md` / system prompt) so every `/watch` runs thoroughly and *tells you* when something couldn't be resolved.

Vault-agnostic and optional. Skip it and `/watch` behaves normally.

---

## Copy-paste instruction

> When watching a video to document it, run thoroughly and leave nothing on the floor:
>
> **1. Capture visuals, don't sample blindly.**
> - Use `--detail balanced` for normal videos; `--detail token-burner` for dense/important ones (courses, coachings, UI-heavy walkthroughs) so **every scene change** becomes a frame.
> - Read **all** the frames, not a sample. Explicitly preserve anything that is *only* visible and not in the transcript: UI details, on-screen text, diagrams, settings/menus, dashboards, code, terminal output.
> - After reading the transcript, re-run with `--timestamps ...` on any moment the speaker points at ("look here", "as you can see") that the frame pass may have missed.
>
> **2. Extract every link and asset — from three places.**
> - The **transcript** (things said/spelled out).
> - The **video description** (pull it via `yt-dlp --print "%(description)s" <url>`), including chapters.
> - The **on-screen frames** — URLs shown in the browser **address bar** or as on-screen text. Read the frame and transcribe the URL.
> - List repos, templates, prompts, configs, slides, downloads, and any **other videos** referenced.
>
> **3. Then actively flag what could NOT be resolved — do not let it slip silently.**
> Present a clearly separated block, e.g. **"⚠️ I need from you"**, listing each unresolved item with a specific question:
> - a link that is **truncated / unreadable** in the frame → ask for the full URL;
> - an asset **behind a login / paywall** (community, course platform) → mark as "you'll need to provide this";
> - a **referenced-but-not-shown** repo / template / download / linked video → propose fetching it or ask where it is.
> Affiliate/redirect short links (amzn.to etc.) are not content — skip those.
>
> The user should **never have to ask** whether something is still open. Surfacing it is your job.

---

## Why this matters
A "watch and answer" pass is fine for a quick look. But for a durable knowledge base, the value is in the details that *only* appear on screen and in the links buried in a description — and in **knowing what you're still missing**. This instruction turns `/watch` from "summarize it" into "capture it completely, and tell me what you couldn't get."
