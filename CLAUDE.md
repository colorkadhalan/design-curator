# CLAUDE.md

Guidance for AI assistants working in this repository.

## What this is

A single-page personal design-inspiration site. Static HTML, deployed to GitHub Pages at `https://colorkadhalan.github.io/design-curator/` from the `main` branch. There is no build step, no framework, and no runtime dependencies.

## Layout

- `index.html` — the entire site. Contains the markup, all CSS (inlined in `<style>`), and the currently-displayed inspiration cards written as static `<article class="post">` blocks. This is the file the browser actually loads.
- `inspiration_data.json` — machine-readable snapshot of the same feed (fields: `id`, `image_url`, `title`, `type`, `source_name`, `source_url`, `category`, `date_added`, `why_curated`). It is written alongside `index.html` by the daily-update process but is **not fetched at runtime** — `index.html` renders from its own inline `<article>` markup, not from this JSON. Keep the two in sync when updating the feed.
- `README.md` — user-facing description (aesthetic preferences, source list, live URL).
- `package.json` — metadata only. No dependencies, no real scripts (`test` just exits 1). Don't add a build toolchain unless explicitly asked.

## How the site works

`index.html` is fully self-contained: one `<header>` (title + "Last refreshed: …" timestamp), one `<main class="feed">` of `.post` cards, one `<footer>`. Each `.post` has:

- `<img class="post-image">` sourced from `https://source.unsplash.com/800x600/?<keywords>` (the query encodes the aesthetic keyword, e.g. `film%20grain%20aesthetic`).
- `.post-tags` with a `.tag.type` chip (medium: photography / web / editorial / branding / motion / graphic / art), a `.tag.source` chip (source name), and a plain `.tag` (category).
- `.post-title`, `.post-source` (link to the source site), and a `.why-curated` footer explaining the curation reason.

The dark palette (`#0a0a0a` background, `#1a1a1a` cards, gradient header from `#1a1a2e` → `#16213e`, `#667eea` → `#764ba2` gradient title) is intentional — preserve it unless the user asks for a redesign.

## Daily update pattern

The repo history is dominated by commits titled `Daily inspiration update - $(date)` (the `$(date)` is literal — the automation shipped an unescaped shell placeholder in the message). Each update rewrites `inspiration_data.json` (bumping `last_updated` / `count` / `inspirations[]`) and the matching `<article>` blocks in `index.html`, then also updates the "Last refreshed" timestamp in the header. When making a similar update:

1. Regenerate the inspirations list in `inspiration_data.json` (keep the schema).
2. Rewrite the `<article class="post">` blocks in `index.html` to match, in the same order.
3. Update the `<div class="last-updated">` string in `index.html`.
4. Keep the tag `type`/`source`/category mapping consistent — the CSS colors `.tag.type` (mint) and `.tag.source` (magenta) are keyed off those class names, not the text.

Do not write a new commit if only cosmetic whitespace changed; the deploy target is GitHub Pages and every commit ships.

## Conventions

- **No frameworks, no build.** Edits are direct HTML/CSS/JSON. If you're tempted to add React, a bundler, or Node dependencies — don't, unless the user explicitly asks.
- **Inline styles only.** All CSS lives in the `<style>` block at the top of `index.html`. No external stylesheets.
- **External images by keyword URL.** Image sources use Unsplash's keyword endpoint; don't inline base64 or add asset directories.
- **Accessible defaults kept.** `alt` on every `<img>`, `loading="lazy"`, `target="_blank" rel="noopener"` on outbound links. Preserve these.
- **Responsive breakpoint at 768px** — see the `@media (max-width: 768px)` block. Any new layout needs a matching mobile rule.

## Verifying changes

There is no test suite or linter. To sanity-check a change:

- Open `index.html` directly in a browser (`file://…`) or serve the directory (e.g. `python3 -m http.server`) and visit `/`.
- Confirm the header, all `.post` cards, and the footer render; confirm the "Last refreshed" line matches what you set; click a source link to confirm the URL is correct.
- Validate `inspiration_data.json` parses (`python3 -m json.tool inspiration_data.json`) and its `count` matches `len(inspirations)`.

## Git workflow

- Default branch is `main`; GitHub Pages deploys from it.
- Feature work happens on `claude/<slug>` branches (see the current branch name for the pattern).
- Never push straight to `main` — open a PR against it.
- Commit messages in history are terse (`Daily inspiration update - …`, `Fix images - …`). Match that tone for content updates; use a descriptive subject for structural changes.
