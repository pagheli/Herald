# Herald

Most news feeds forget you the moment you close the tab. Herald doesn't. Every Read, Skip, and Save writes itself back into the page immediately, so the next visit picks up exactly where the last one left off. A daily pass adds new stories on top of the pile, never touching what you haven't gotten to, and prunes the read-and-skipped backlog on a cap while anything you've saved stays untouched, indefinitely.

Herald is a single self-contained HTML page, published as a Claude
Artifact, not a normal website. A scheduled Claude session fetches fresh stories once a day, folds them in on top of whatever you haven't acted on yet, and republishes the same page. Every click saves itself immediately through the artifact's own runtime. No scheduled run required for that part.

This requires a Claude client that can publish Artifacts with runtime capabilities and run scheduled tasks (Claude Code, Cowork, or claude.ai with those features enabled). It will not work as a plain hosted webpage.


## Contents
 
```
.
├── LICENSE
├── README.md
└── skill/
    ├── SKILL.md                  the Claude Skill manifest for this folder
    ├── daily-fetch-prompt.md     the scheduled-task prompt that keeps it updated
    └── template.html             the page you publish as your first Artifact
```


## Design

Six built-in categories (technology, stocks, healthcare, biotech, startups, telecom), each with its own accent color used consistently across the category filter, the article card border tint, and the tag label. Read and Skip move an article to the Archive tab (Skip additionally tells the daily fetch to deprioritize similar stories). Save bookmarks an article independently of its read state and exempts it from the Archive's size cap. Both light and dark color schemes are defined, following the OS preference
by default.


## Quick start

Herald is a self-contained Claude skill. For a quick start, copy [`skill/`](/skill/) folder into a Claude skills directory and name the copy `herald` there to install it as a Skill.


## Step-by-step setup

### 1. Publish the template

Publish [`skill/template.html`](/skill/template.html) as a new Claude Artifact with the `artifact` capability declared (`capabilities: {"artifact": {}}`), and give it a one-word icon (the Artifact tool's icon field takes a plain word, not an emoji, so "newspaper" or "globe" work; an emoji doesn't). Once published, note its `claude.ai/code/artifact/...` URL.
 
**Claude prompt:**
> Publish skill/template.html as a new Claude Artifact. Declare the artifact capability (capabilities: {"artifact": {}}) so the page can save its own state. Give it the icon <one word, e.g. newspaper>. Once it's published, give me the resulting claude.ai/code/artifact/... URL.

Save the artifact URL. Everything downstream needs it.

### 2. Try it

Open the artifact at the URL from step 1. It ships with two starter cards so you can confirm Save, Skip, Read, Undo, the Category dropdown, and the Saved/Archive tabs all work before anything is scheduled.

### 3. Customize the template (optional, before or after step 1)

   - **Rename it:**  change every `Herald` occurrence in [`template.html`](/skill/template.html) (the
     `<title>`, the `<h1 class="wordmark">`) and in `daily-fetch-prompt.md`.
   - **Change the tagline:** edit `<p class="tagline">Signal over noise.</p>`.
   - **Add or remove categories:** each category needs a `--<name>` color
     variable in `:root` (and its dark-mode copy), a
     `.filter-btn[data-cat="<name>"] { --cat-color: var(--<name>); }` rule,
     an `.item[data-category="<name>"] { --cat-color: var(--<name>); }`
     rule, a `body[data-local-filter="<name>"]
     .item[data-category="<name>"] { display: flex; }` rule, and an
     `<option>` in the `#category-select` dropdown. Keep the category's
     internal slug lowercase and the display label (the `tag` field) as you
     want it shown.
   - **Change the favicon or accent colors:** the `--accent` and related
     `--bg`/`--ink`/`--line` custom properties at the top of `<style>`
     control the whole palette, light and dark.

### 4. Set up the scheduled fetch 

Fill in the placeholders in [`skill/daily-fetch-prompt.md`](/skill/daily-fetch-prompt.md) yourself first (your artifact URL, product name, owner name, recency window, archive cap, favicon word, description; see that file for the full list). Then ask Claude to create a scheduled task using the filled-in prompt. Run it once manually first to confirm it reads the live page, fetches real stories, and republishes cleanly before trusting it to run unattended.


## Why an artifact, not a plain website

The click-to-save behavior — Read, Skip, Save, and Archive persisting instantly, with no server of your own — relies on the `artifact` runtime capability Claude Artifacts can declare. That capability is what lets a static-looking HTML page save a new version of itself when you click a button in it, and it only exists inside claude.ai, which is why this ships as a Claude Artifact rather than a file you host yourself.


## How the self-save mechanism works

Every published version of the page embeds three `<script>` blocks: the current article data as JSON (`#articles-data`), the interaction logic verbatim (`#app-logic`), and a one-line bootstrap that parses the JSON into an `ARTICLES` array and calls `init()`. Clicking Read, Skip, Save, or Undo calls `persist()`, which rebuilds the *entire* HTML document from the in-memory `ARTICLES` array (via `buildFullHtml()`) and calls `artifact.publish(html)` on it — the whole document is replaced on every click, not patched. That means the static `<main class="feed">` markup and the `renderArticleHTML()` / `buildFullHtml()` JavaScript template must stay structurally identical, or your next click will silently regenerate the page from the JS template and undo any structural change you made only to the static HTML side. If you edit the visual layout, change it in both places.


## Data model

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
  "feedback": "",
  "archived": false,
  "pinned": false
}
```

> `feedback` is the only negative/positive-adjacent signal: `"skip"` is a negative preference the daily-fetch prompt uses to deprioritize similar future stories; `"read"` carries no preference at all. `pinned` (Save) is completely independent of `feedback`/`archived` — an article can be unread-and-saved, read-and-saved, or skipped-and-saved, and saved articles are exempt from the archive-size cap forever.


## License

MIT — see [`LICENSE`](/LICENSE).
