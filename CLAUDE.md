# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The website for Bergslagens Fight Club (https://bergslagensfightclub.com), a Swedish martial arts club. It is a static site built with [Zola](https://www.getzola.org/). All site content is in Swedish. The actual site lives in `zola-site/`; the repo root contains nothing else of interest.

## Commands

All commands run from `zola-site/`:

- `zola serve` — local dev server with live reload
- `zola build` — build the site into `public/` (gitignored)
- `zola check` — validate internal/external links

There are no tests or linters.

### Deploying

The site deploys to AWS Amplify as a manual zip deployment (no CI):

- `./really-deploy.sh` — the actual deploy entry point. It is **gitignored on purpose** because it contains the Amplify app ID; never commit it. It just calls `./deploy.sh <app-id> public`.
- `deploy.sh` — builds with Zola, zips `public/`, creates an Amplify deployment via the AWS CLI, uploads, starts the job, and polls until SUCCEED/FAILED. Requires `aws` and `jq` configured locally.

## Architecture

This is mostly a single-page site: the homepage (`themes/zola-grayscale/templates/base.html`) renders all sections in order — masthead, about (with embedded Instagram), training times, priser (prices), contact — and the navbar (`navigation.toml`) links to anchors on that page (`#about`, `/#training`, `/#priser`, `#contact`).

### Theme override pattern

The theme `themes/zola-grayscale` (a Zola port of the Bootstrap "Grayscale" template, vendored into this repo and edited directly) defines `base.html` as a series of Tera blocks. Site-level templates in `zola-site/templates/` extend it and override individual blocks, e.g. `templates/index.html` only overrides `projects_id` to rename the anchor to `training`. When changing page structure, check whether the block lives in the theme's `base.html` or a site template override.

### "Latest page wins" content sections

The homepage's training and priser sections pull content dynamically: `base.html` does `get_section(path="training/_index.md")` (and same for `priser`), sorts pages by date, and renders **only the most recent page's** markdown body. So to publish new training times or prices, add a new markdown file (e.g. `content/training/varen-2026.md`) with a newer `date` in its front matter — no template changes needed. The markdown bodies are plain tables that get styled by the theme's `_training_table.scss`.

### Config split

Site data lives in TOML files loaded at template render time via `load_data`, not in `config.toml`:

- `navigation.toml` — navbar items
- `contact.toml` — contact cards and social links (footer/contact section)

These exist both at `zola-site/` (live values) and inside the theme (theme defaults); the site-level copies win because Zola resolves `load_data` paths relative to the site root.

## History note

The repo path mentions Dioxus because an earlier proof-of-concept (a Rust/Dioxus member registry) lived here; it was removed in commit b342734. Don't be confused by the path — there is no Rust code.
