---
name: herald
description: Set up a personal, self-updating news feed as a Claude Artifact: a live page with Read/Skip/Save/Archive that a scheduled Claude session tops up daily.
---

# Herald

A personal news feed that saves its own state on every click and tops
itself up on a schedule. No server of your own.

## Setup

1. Publish `template.html` (in this folder) as a new Claude Artifact.
   Declare the `artifact` runtime capability
   (`capabilities: {"artifact": {}}`) so the page can save its own state.
   Give it a one-word icon. Report back the resulting
   `claude.ai/code/artifact/...` URL: every later step needs it.
2. Open that artifact and confirm Save, Skip, Read, Undo, the category
   dropdown, and the Saved/Archive tabs work, using its two starter
   cards.
3. Optional, before or after step 1: edit `template.html` to rename the
   product, change the tagline, adjust categories, or change colors. If
   this folder sits inside the full repo, the README there has the exact
   lines to touch.
4. Fill in the placeholders in `daily-fetch-prompt.md` (in this folder):
   the artifact URL from step 1, product name, owner name, recency
   window, archive cap, icon word, description. Create a scheduled task
   with the filled-in prompt. Run it once manually first to confirm it
   fetches real stories and republishes cleanly before trusting it
   unattended.

## Files in this folder

- `template.html` — the starter page to publish (has the `artifact`
  capability wired up already).
- `daily-fetch-prompt.md` — the scheduled-task prompt template.

This folder is self-contained: everything the setup steps above need is here. For the self-save mechanism, the article data model, and full design notes, see the repo's `README.md` if you have it alongside this folder.
