# Précis

A personal, self-updating news feed: a feed UI with per-article Read, Skip,
Save, and Archive, organized by category, that tops itself up with fresh
stories once a day instead of replacing what you haven't gotten to yet.

This repo has two independent ways to run it:

| | `skill/` | `web/` |
|---|---|---|
| Where it runs | A Claude Artifact (claude.ai) | Any static web host, or just a local file |
| Daily fetch | A scheduled Claude session reads, dedupes, and republishes | None — you edit the JSON or write your own fetch script |
| Persistence | The `artifact` runtime capability (saves instantly, synced) | `localStorage` in one browser (no sync) |
| Setup | Publish a template, schedule a prompt | Open `index.html` |

Pick `skill/` if you want the original, fully-automated version and already
use Claude. Pick `web/` if you want the same interface with zero
Claude dependency and are fine doing the "find new articles" part yourself
or with your own script.

## `skill/` — the Claude Artifact version

- `SKILL.md` — what to publish and how to wire up the daily fetch.
- `template.html` — the starter page to publish as a Claude Artifact.
- `daily-fetch-prompt.md` — the scheduled-task prompt that keeps it current.

## `web/` — the standalone version

- `index.html` — a single self-contained page, no build step, no server.
- `README.md` — how to run it and the article-data schema.

## Design

Six built-in categories (technology, stocks, healthcare, biotech, startups,
telecom), each with its own accent color used consistently across the
category filter, the article card border tint, and the tag label. Read and
Skip move an article to the Archive tab (Skip additionally tells the daily
fetch to deprioritize similar stories); Save bookmarks an article
independently of its read state and exempts it from the Archive's size cap.
Both light and dark color schemes are defined, following the OS preference
by default.

## License

MIT — see `LICENSE`.
