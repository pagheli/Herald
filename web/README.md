# Précis — standalone web version

A single self-contained HTML file (`index.html`) with the same feed UI, category
filters, and Read / Skip / Save / Archive behavior as the Claude Artifact
version, minus any dependency on claude.ai. Persistence is plain
`localStorage` in the visitor's own browser instead of the `artifact` runtime
capability.

## Run it

Open `index.html` directly in a browser, or host it anywhere that serves
static files — GitHub Pages, Netlify, a plain S3 bucket, `python3 -m http.server`.
There is no build step and no server-side code.

```bash
python3 -m http.server -d web 8000
# then open http://localhost:8000
```

## What's different from the Claude Artifact version

- **Persistence**: clicking Read / Skip / Save / Undo writes only a small
  `{id: {feedback, archived, pinned}}` map to `localStorage` under the key
  `precis:overrides:v1`, then `init()` re-applies it over whatever articles
  are baked into the page on load. That means you can redeploy `index.html`
  with new articles and a visitor's past read/skip/save state on
  still-present article IDs survives; it does not sync across devices or
  browsers, since nothing is sent to a server.
- **No fetch step**: this file does not go out and find news. The
  Claude-based skill in `../skill/` describes how an agent fetches, dedupes,
  and republishes daily — there's no equivalent here, since that's a Claude
  session doing the work, not client-side JavaScript. To keep this page
  current yourself, either edit the `articles-data` `<script>` block by hand
  or write your own fetch/build script (in any language) that regenerates
  it. Keep the schema below.

## Article schema

Each entry in the `articles-data` JSON array:

```json
{
  "id": "unique-string",
  "category": "tech | stocks | health | biotech | startup | telecom",
  "tag": "Technology | Stocks | Healthcare | Biotech | Startups | Telecom",
  "source": "Publication name",
  "date": "Display string, e.g. \"Sep 16\"",
  "title": "Headline",
  "url": "https://...",
  "summary": "A few sentences of context.",
  "feedback": "" ,
  "archived": false,
  "pinned": false
}
```

To add categories beyond the built-in six, add a `--<name>` color variable
next to the others at the top of `<style>`, a matching
`.filter-btn[data-cat="<name>"] { --cat-color: var(--<name>); }` /
`.item[data-category="<name>"] { --cat-color: var(--<name>); }` rule pair,
a `body[data-local-filter="<name>"] .item[data-category="<name>"] { display: flex; }`
rule, and an `<option>` in the Category `<select>`.

If a generator script produces a full `<article class="item" ...>` block per
entry, keep it byte-identical in structure to the three demo articles already
in the file (same attributes, same nested `item-actions` markup, same
Save → Skip → Undo → Read button order) — `applyDom()` only toggles
attributes on existing elements, it never builds markup, so a structural
mismatch between the static HTML and the JSON silently breaks the buttons for
that article.
