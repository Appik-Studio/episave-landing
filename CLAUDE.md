# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EpiSave marketing website — a Hugo static site with the PaperMod theme (git submodule), deployed on Cloudflare Pages. Multilingual (English + French) with Decap CMS for content editors.

## Commands

```bash
# Dev server (with drafts)
hugo server -D

# Production build
hugo --minify

# Clone (first time — needs submodules for PaperMod theme)
git clone --recurse-submodules <repo-url>
```

No `package.json` or Node.js dependencies. Hugo binary only. Required version: `0.147.0`.

## Architecture

- **`hugo.toml`** — All site config: languages, menus, params, SEO settings
- **`content/{en,fr}/`** — Markdown pages with YAML frontmatter. Blog posts in `blog/` subdirectory
- **`layouts/index.html`** — Homepage entry point, composes partials from `layouts/partials/landing/`
- **`layouts/partials/landing/`** — Modular landing sections: hero, features, how-it-works, stats, cta, latest-posts
- **`layouts/partials/extend_head.html`** — Injected into `<head>` (custom CSS + JSON-LD)
- **`layouts/partials/json-ld.html`** — Schema.org structured data (Organization, WebSite, Article, MedicalDevice)
- **`layouts/shortcodes/`** — Reusable components (e.g., donate-button)
- **`assets/css/landing.css`** — All custom styles (plain CSS, no build step, processed by Hugo Pipes)
- **`i18n/{en,fr}.yaml`** — All UI strings. Templates use `{{ i18n "key" }}` for translations
- **`static/admin/`** — Decap CMS config (GitHub OAuth backend, editorial workflow)
- **`static/images/`** — Image assets (logo.png, hero-watch.png, og-episave.png)
- **`themes/PaperMod/`** — Theme (git submodule, do not edit directly)

## Key Conventions

- **All URLs include language prefix** (`/en/about/`, `/fr/contact/`) — enforced by `defaultContentLanguageInSubdir = true`
- **User-facing text goes in `i18n/` files**, not hardcoded in templates. Use `{{ i18n "key" | default "fallback" }}`
- **Internal links use `relLangURL`** for language-aware routing
- **Content in both languages must stay in sync** — mirror structure between `content/en/` and `content/fr/`
- **PaperMod theme is a submodule** — override via `layouts/` (never edit `themes/PaperMod/` directly)

## Deployment

- **Hosting**: Cloudflare Pages
- **Staging**: Every branch push creates a preview URL
- **Production**: Merge to `main` auto-deploys to `episave.ch`
- **Build command**: `hugo --minify`
- **Env var**: `HUGO_VERSION=0.147.0`
