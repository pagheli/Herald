# Herald — daily-fetch scheduled-task prompt

This is the prompt to paste into a Claude scheduled task (Claude Code /
Cowork "scheduled task", not a raw cron job) so it maintains your Herald
artifact automatically. It assumes you've already published `template.html`
as a Claude Artifact with the `artifact` capability (see `SKILL.md`).

Fill in the four placeholders marked `{{...}}` before saving it, then adjust
the source list in STEP 3 to match your own interests — the version below
covers tech / startups / stocks / healthcare / biotech / telecom as a
starting point, but nothing about the mechanism is specific to those topics.

```
{{PRODUCT_NAME}} — DAILY EDITION (automated, unattended run)

You are a fresh session with no memory of any prior conversation. This
prompt is fully self-contained. Do not ask the user anything — this runs on
a schedule with nobody watching. Do not send the user a message at the end
unless the safety rule below tells you to; on a normal successful run, just
publish.

GOAL
Maintain {{OWNER_NAME}}'s personal news artifact "{{PRODUCT_NAME}}" at:
{{ARTIFACT_URL}}

Each run must PILE new stories on top of whatever the user hasn't acted on
yet — never replace, re-summarize, or reorder articles the user hasn't
clicked Read or Skip on. Articles the user has clicked Read or Skip on live
in the Archive and must also be preserved (up to a cap — see STEP 6).
Separately, the user can Save any article (regardless of its Read/Skip/
Archive status) to bookmark it for later under a Saved tab — saved articles
must NEVER be dropped or pruned, ever, no matter how old.

ARCHITECTURE NOTE: this page is not a static HTML snapshot. Every article
lives as one JSON object in a `<script type="application/json"
id="articles-data">` block, and a `<script id="app-logic">` block
(reproduced verbatim below — do not rewrite or "improve" it) makes each
click call the `artifact` capability's `publish()` method directly from the
browser, so the user's Read/Skip/Save clicks save themselves immediately and
survive a reload. Your job on each run is only to fold in fresh stories and
republish using the same structure — never regress to a plain-HTML-only
template without data-id attributes or the app-logic script; that loses
click persistence entirely.

CRITICAL SAFETY RULE — READ THIS FIRST: this page's Read/Skip/Save state
lives only in the articles-data JSON on the live page — there is no other
backup. If STEP 1 fails to load the live page for ANY reason (tool error,
empty/garbled result, missing or unparseable articles-data block, cannot
reach the artifact host, anything), you MUST STOP IMMEDIATELY and NOT call
the Artifact tool's publish at all this run. Publishing a rebuilt page
without first confirming current state risks silently discarding every
Read/Skip/Save action the user has made. Skipping a scheduled run harmlessly
is always the safe fallback; wiping the user's saved state is not. If you
must abort, end the run without publishing, but DO send the user one short
message explaining that this run was skipped and briefly why (e.g.
"{{PRODUCT_NAME}}: skipped this run — couldn't confirm the current
read/skip/save state, so nothing was published, to avoid losing your saved
data.").

STEP 1 — Read the current live page.
Use the Artifact tool with action="read" and url = the URL above. This
returns the exact, byte-for-byte published HTML for an artifact you own —
unlike WebFetch, it is not passed through a summarizing model, so it won't
silently drop or "normalize" JSON field values (this matters once
articles-data grows past 10-15KB). For a page this size the tool will save
the full HTML to a local file and tell you its path — read that file in
full before proceeding. Locate the
`<script type="application/json" id="articles-data">...</script>` element
and parse its content as JSON. Each element has this shape:
{"id": "...", "category": "...", "tag": "...", "source": "...", "date":
"...", "title": "...", "url": "...", "summary": "...", "feedback": ""|"read"|
"skip", "archived": true|false, "pinned": true|false}.
Only if the Artifact tool is unavailable in this session, fall back to
WebFetch on the same URL asking it to return the articles-data element
verbatim and completely — but treat that as lower-confidence, sanity-check
it (don't let a lossy fetch quietly zero out every `pinned: true` article),
and STOP per the safety rule above rather than publish on a read you don't
trust.

STEP 2 — Classify what you found.
`archived: false` (feedback will be "") → UNREAD, still active in the pile.
`feedback: "skip"` (archived will be true) → SKIPPED.
`feedback: "read"` (archived will be true) → READ.
`pinned: true` → SAVED, independent of the other three — an article can be
unread-and-saved, read-and-saved, or skipped-and-saved. Do not alter,
re-word, or drop any article you found, and keep every field (especially
`id` and `pinned`) exactly as extracted. Never fabricate an article that
wasn't actually on the page, and never silently unsave something.

STEP 3 — Fetch fresh candidate stories.
Replace this list with whatever sources and categories you actually care
about — RSS feeds, Hacker News, a paid news API, an MCP connector, anything
WebFetch or an available tool can reach. Example starting point:

  Technology: TechCrunch RSS (https://techcrunch.com/feed/), The Verge RSS
  (https://www.theverge.com/rss/index.xml), Hacker News front page
  (https://news.ycombinator.com/).
  Startups: TechCrunch Startups RSS
  (https://techcrunch.com/category/startups/feed/), Hacker News Show HN.
  Stocks: a market/financial news source or MCP tool you have access to.
  Healthcare: STAT News RSS (https://www.statnews.com/feed/).
  Biotech: STAT News RSS (biotech-tagged items), FierceBiotech RSS
  (https://www.fiercebiotech.com/rss/xml).
  Telecom: Light Reading RSS (https://www.lightreading.com/rss.xml),
  Fierce Network RSS (https://www.fierce-network.com/rss.xml).

Use WebFetch (or an allowlisted MCP tool) for all of these — arbitrary
outbound network access via raw curl/bash is typically blocked in a Claude
sandbox.

RECENCY FILTER: only carry forward candidates actually published within the
last {{RECENCY_HOURS}} hours, relative to this run's start time. Compute
"now" once, up front (e.g. via a shell date command), and reuse that same
timestamp for every source's cutoff. For RSS feeds, use each item's
<pubDate> (or <published>/<updated>) field. For Hacker News, use the
displayed relative age next to each story and skip anything a day old or
older. For a news API with its own timestamp field, use that field directly,
passing a `time_from`-style parameter to filter server-side where the tool
supports it. Discard anything older than the cutoff before it ever reaches
STEP 4 — this is a hard cutoff in addition to, not instead of, the dedupe
check below.

STEP 4 — Dedupe against what's already there.
Drop any fetched candidate whose headline or story matches — even a
differently-worded write-up of the same underlying story — an article you
already classified in STEP 2 (UNREAD, READ, or SKIPPED, doesn't matter
which). Only genuinely new stories move on to STEP 5.

STEP 5 — Curate this run's new stories.
From the deduped candidates, pick roughly 4-8 of the most substantive
stories per category with real new candidates (skip a category entirely
this run if nothing new and worthwhile turned up — don't pad). Write
medium-depth summaries: 3-5 sentences with real context, specific and a
little wry, explaining why it matters rather than just what happened. Use
the SKIPPED list from STEP 2 as the only negative-preference signal —
deprioritize or drop new candidates that closely resemble the topic,
source, or angle of something the user has skipped. Read items carry no
preference signal at all — never treat them as liked or disliked. New
articles are never saved (pinned: false) on creation — saving is something
only the user does by clicking. Give each new article a fresh, stable `id`
(e.g. "a" followed by 10 lowercase hex characters derived from its URL) and
check it against every id from STEP 1/2, regenerating on collision.

STEP 6 — Assemble the full page.
Build the ARTICLES array, in this order:
  1. This run's new articles (STEP 5), most significant first.
  2. All UNREAD articles from STEP 2, reproduced exactly as extracted (all
     fields, especially id and pinned), in their original relative order,
     untouched.
  3. All READ and SKIPPED (archived) articles from STEP 2, reproduced
     exactly as extracted, with feedback and archived: true set correctly,
     and pinned preserved exactly.

Cap on block 3 only: if the combined READ+SKIPPED count would exceed
{{ARCHIVE_CAP}}, drop the oldest ones by date (oldest first) until at most
{{ARCHIVE_CAP}} remain — but SAVED (pinned: true) articles are exempt from
this cap, no matter how old; never drop a saved article for any reason.
Blocks 1 and 2 (the active pile) are never capped or trimmed.

Compute today's real date/time for the header, in the exact format
"Updated [DD] [MON] [YYYY] &middot; [HH:MM]" (sentence case "Updated", no
weekday, no timezone suffix) — compute it live, don't reuse any example
value verbatim.

Build the "Archived so far — Read: ...; Skipped: ..." text (goes in the
`<p id="feedback-log">` element) listing every article now classified as
READ or SKIPPED after applying the cap, in the exact "Headline [category];
Headline [category]" format joined with "; ". If there are none of a kind,
write "none" for that half.

ARTICLE TEMPLATE — use this exact HTML structure for every article in all
three blocks, filling in the bracketed values (do not add or remove any
element, do not change any onclick call):

<article class="item" data-id="ID" data-category="CATEGORY" data-feedback="FEEDBACK" data-archived="ARCHIVED" data-pinned="PINNED">
  <div class="item-card">
    <div class="item-meta"><span class="tag">TAG</span><span>&middot;</span><span>SOURCE</span><span>&middot;</span><time>DATE</time><span class="archived-status"></span><span class="pin-indicator"><svg viewBox="0 0 24 24" width="10" height="10" aria-hidden="true"><path d="M19 21l-7-5-7 5V5a2 2 0 0 1 2-2h10a2 2 0 0 1 2 2z"/></svg></span></div>
    <h2 class="item-title"><a href="URL" target="_blank" rel="noopener">HEADLINE</a></h2>
    <p class="item-summary">SUMMARY</p>
    <button class="expand-btn" onclick="toggleExpand(this)">Read more</button>
    <div class="item-actions">
      <button class="pin-btn" aria-label="Save for later" onclick="togglePin('ID')"><svg class="pin-icon" viewBox="0 0 24 24" width="12" height="12" aria-hidden="true"><path d="M19 21l-7-5-7 5V5a2 2 0 0 1 2-2h10a2 2 0 0 1 2 2z"/></svg><span class="pin-label">Save</span></button>
      <button class="fb-btn" data-fb="skip" aria-label="Skip — fewer like this" onclick="setFeedback('ID','skip')">&#10005; Skip</button>
      <button class="undo-btn" aria-label="Undo" onclick="undoFeedback('ID')">&#8634; Undo</button>
      <button class="fb-btn" data-fb="read" aria-label="Mark as read" onclick="setFeedback('ID','read')">&#10003; Read</button>
    </div>
  </div>
</article>

`item-actions` is nested INSIDE `item-card` (a child of it, after the
expand-btn, not a sibling of item-card) and renders as a horizontal row
aligned to the bottom-right. Button order matters: Save, then Skip, then
Undo, then Read (Undo only shows once archived, replacing Skip/Read; Save is
always visible). CATEGORY/TAG pairs used by the default template: tech/
Technology, stocks/Stocks, health/Healthcare, biotech/Biotech, startup/
Startups, telecom/Telecom — add your own pair by also adding a CSS variable
and a nav `<option>` (see `SKILL.md`). Always use the plain classes shown
above (no "active" suffix) regardless of state — the app-logic script's
`init()` calls `applyDom()` for every article on load, which sets the
correct active classes and button label automatically. Your only job is to
get the four `data-*` attributes right on the `<article>` tag.

STEP 7 — Publish (only if STEP 1 succeeded — see the safety rule at the top).
Write the fully assembled content (matching the page shell in
`template.html` exactly — same `<style>`, header, filters nav, footer, and
the three script blocks — with your STEP 6 output filling `<main>` and the
articles-data JSON) to a local `.html` file. Then call the Artifact tool
with: file_path = that file, url = "{{ARTIFACT_URL}}", favicon =
"{{FAVICON_EMOJI}}", capabilities = {"artifact": {}}, description =
"{{DESCRIPTION}}". Do not change contract. If the tool reports a version
conflict, re-read the URL once more, re-merge your STEP 6 output on top of
whatever changed, and retry — only force as an absolute last resort, and
only after confirming you are not discarding newer viewer-made changes.
```

## Placeholders to fill in

- `{{PRODUCT_NAME}}` — what you're calling your feed (e.g. "Herald").
- `{{OWNER_NAME}}` — your name, used in the GOAL line.
- `{{ARTIFACT_URL}}` — the `claude.ai/code/artifact/...` URL from publishing
  `template.html` (see `SKILL.md`).
- `{{RECENCY_HOURS}}` — how far back to look for "new" stories (24 is a
  reasonable default for a once-a-day run).
- `{{ARCHIVE_CAP}}` — how many read+skipped articles to keep before pruning
  the oldest (60 is what the original build uses).
- `{{FAVICON_EMOJI}}` — one or two emoji for the artifact's favicon.
- `{{DESCRIPTION}}` — a one-line description passed to the Artifact tool.

## Scheduling it

Use your client's scheduled-task feature (in Claude Code / Cowork, the
`create_trigger` tool or its UI equivalent — never a plain cron job, since a
local cron loses its schedule when the session ends) to run this prompt on
whatever cadence you want. A daily cadence matches the "PILE new stories on
top" design; running it more often just means smaller batches each time.
