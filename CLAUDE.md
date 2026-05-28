# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Deploy

```bash
# Build the site locally (output goes to /public at repo root)
hugo -s site/ --minify

# Serve locally with live reload
hugo server -s site/
```

Deployment is fully automated: every push to `main` triggers the GitHub Actions workflow (`.github/workflows/deploy.yml`), which builds with Hugo and deploys to Cloudflare Workers via Wrangler.

## Architecture

This is a static marketing site for SLGlobal. The stack is:

- **Hugo** (static site generator) — source lives in `site/`
- **Cloudflare Workers** — serves the built assets from `public/` via `worker.js` (a thin passthrough to `env.ASSETS`)
- **Wrangler** (`wrangler.toml`) — points Cloudflare at `./public` as the asset directory

### Content model

All page content is stored as YAML front matter in `site/content/*.md`. The markdown body is largely unused — data is pulled from front matter fields in the layouts. Hugo's goldmark renderer has `unsafe = true` to allow raw HTML in content strings.

Each page specifies its layout via `layout:` in front matter, which maps to `site/layouts/_default/<layout>.html`. The home page is special — it uses `site/layouts/index.html` directly (no `layout:` key needed).

### Layout / CSS conventions

- `site/layouts/_default/baseof.html` is the shell (nav + footer). All page layouts use `{{ define "main" }}`.
- All styles are in one file: `site/static/css/style.css`. CSS custom properties (design tokens) are defined in `:root`.
- Background colors by section type: hero = `--blue-darkest`, about/founders = `--blue-frost`, services = `--white`, testimonials = `--blue-mist`, contact = `--blue-darkest`.
- Sections that sit directly under the fixed nav need class `page-top` (adds `calc(7rem + 5rem)` top padding).

### Adding or editing content

- **Bio text, service descriptions, testimonials** — edit the YAML front matter in the relevant `site/content/*.md` file. All strings must be valid YAML double-quoted strings (no backslash escapes except standard ones like `\n`).
- **Adding a page** — create `site/content/<page>.md` with `layout: "<name>"`, add a corresponding `site/layouts/_default/<name>.html`, and link it in the nav in `baseof.html`.