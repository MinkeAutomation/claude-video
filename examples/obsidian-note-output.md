# Optional: turn `/watch` into a structured Obsidian note

`/watch` itself just hands Claude the frames + transcript and answers your question. If you want it to **file the result as a rich Obsidian/Markdown note** — properties, a link list with importance markers, the video description, a timestamped transcript, and the key frames embedded — paste the instruction block below into your prompt (or drop it into your project's `CLAUDE.md` / system prompt so it applies to every `/watch`).

This is completely optional and vault-agnostic: it works in any Obsidian vault or plain Markdown folder. Skip it and `/watch` behaves normally.

---

## Copy-paste instruction

> After running `/watch <url>`, save the result as a Markdown note in this format:
>
> **Frontmatter properties:**
> ```yaml
> ---
> title: "<video title>"
> author: <channel/author>
> channel: <channel URL>
> source: <video URL>
> published: <upload date>
> saved: <today>
> length: <duration>
> ---
> ```
>
> **Sections:**
> 1. `## At a glance` — 3–5 sentence summary of what the video covers.
> 2. `## Key points` — the substantive takeaways, with `[MM:SS]` timestamps where useful.
> 3. `## Links` — every link mentioned in the video or its description, grouped by importance:
>    - 🔴 important (tool / repo / docs / core resource)
>    - 🟡 medium (further reading, related videos)
>    - 🟢 low (sponsor / affiliate / product)
>
>    Format each as: `👉 <url> (<type>)`. Pull links from the on-screen frames (browser address bar) and the video description.
> 4. `## Description` — the full video description + chapter list.
> 5. Embed the **key frames** inline next to the point they illustrate (`![[frame_XX.jpg]]`), especially anything that is *only* visible (UI, on-screen text, diagrams, settings) and not in the transcript.
>
> Save the **full transcript** to a separate `<name> - transcript.md` file, one line per cue with `[MM:SS]` prefixes.

---

## Why the separate transcript file?
Keeping the long raw transcript out of the main note keeps the note readable, while the timestamped transcript stays available for search and citation. The main note is the synthesis; the transcript file is the source of record.

## Why importance markers on links?
Skimming a note later, 🔴/🟡/🟢 tells you at a glance which links are worth opening. The 👉 arrow makes each link visually scannable in a wall of text.
