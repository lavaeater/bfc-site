# Bergslagens Fight Club — Website

The website for [Bergslagens Fight Club](https://bergslagensfightclub.com), a martial arts club in Kopparberg, Sweden. It is a static site built with [Zola](https://www.getzola.org/) and hosted on AWS Amplify. All site content is in Swedish.

Everything lives in the `zola-site/` directory — run all commands from there.

## Prerequisites

- **[Zola](https://www.getzola.org/documentation/getting-started/installation/)** — the static site generator (the site is known to build with 0.22.x)
- For deploying only:
  - **[AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)** configured with credentials that can deploy to the club's Amplify app (ask the maintainer for credentials)
  - **jq** and **zip** (usually already installed, otherwise available in your package manager)

## Local development

```sh
cd zola-site
zola serve
```

This starts a local server at http://127.0.0.1:1111 with live reload — edits to content, templates, and styles show up immediately.

Other useful commands:

```sh
zola build   # build the site into public/ (gitignored)
zola check   # validate internal and external links
```

## Editing content

The site is essentially a single page. The homepage renders all sections in order — masthead, about, training times (Träning), prices (Priser), contact — and the navbar links to anchors on that page.

### Updating training times or prices

The training and prices sections on the homepage automatically show **the most recent page** (by `date` in the front matter) from `content/training/` and `content/priser/` respectively. To publish a new schedule or price list, just add a new markdown file — no template changes needed:

```sh
# example: training times for autumn 2026
zola-site/content/training/hosten-2026.md
```

```markdown
+++
title = "Träningstider hösten 2026"
date = "2026-08-15"
+++

## Träningstider hösten 2026

### Motionsgruppen
| Dag      | Tid         |
|----------|-------------|
| Måndag   | 18:30-20:00 |
```

As soon as its `date` is the newest in the section, it becomes the one shown on the homepage. Old files can stay as history. Use plain markdown tables — they are styled automatically.

Prices work exactly the same way in `content/priser/` (see `priser-2026.md` for the format).

### Navigation and contact details

These are plain TOML files in `zola-site/`, loaded by the templates at build time:

- `navigation.toml` — the navbar items and their anchors
- `contact.toml` — address, e-mail, and social media links shown in the contact section

(The theme directory contains copies of these files; the ones at the `zola-site/` root are the live ones.)

### Templates and styling

The theme is `zola-site/themes/zola-grayscale/`, a Zola port of the Bootstrap "Grayscale" template, vendored into this repo and edited directly. Its `templates/base.html` defines the whole homepage as a series of Tera blocks; site-level templates in `zola-site/templates/` extend it and override individual blocks. Styles are SCSS under the theme's `sass/` directory and compiled by Zola.

## Publishing to AWS

Deployment is a manual zip upload to AWS Amplify — there is no CI; pushing to git does **not** deploy the site.

### One-time setup

1. Configure the AWS CLI with the credentials you've been given (`aws configure`, or set up a profile).
2. Create `zola-site/really-deploy.sh` with the Amplify app ID (ask the maintainer for the ID). This file is **deliberately gitignored** — never commit it:

   ```sh
   #!/usr/bin/env bash

   ./deploy.sh <amplify-app-id> public
   ```

   ```sh
   chmod +x zola-site/really-deploy.sh
   ```

### Deploying

```sh
cd zola-site
./really-deploy.sh
```

This runs `deploy.sh`, which builds the site with Zola, zips `public/`, creates an Amplify deployment, uploads it, starts the job, and polls until it reports success or failure. A successful run ends with `✅ Deployment completed successfully!` and the site is live shortly after.
