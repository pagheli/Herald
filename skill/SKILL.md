---
name: newsfeed
description: Set up a personal, self-updating news feed as a Claude Artifact — a live page with Read/Skip/Save/Archive that a scheduled Claude session tops up daily.
---

# Newsfeed skill

See the repo's [`README.md`](../README.md) for the full description, setup
steps, the self-save mechanism, the article data model, and design notes.

This folder holds the two files a setup actually needs:

- `template.html` — the starter page to publish as your first Claude
  Artifact (has the `artifact` capability wired up already).
- `daily-fetch-prompt.md` — the scheduled-task prompt that fetches fresh
  stories once a day and republishes the page, with placeholders for your
  artifact URL, product name, and a few tunable settings.
