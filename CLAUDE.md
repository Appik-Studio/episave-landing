# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EpiSave marketing website — a Hugo static site with the PaperMod theme (git submodule), deployed on Cloudflare Pages. Multilingual (English + French) with Decap CMS for content editors. The product is an open-source medical device for seizure detection using Android smartwatches.

**GitHub repo**: `Appik-Studio/episave-landing`

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

```
episave-site/
├── archetypes/blog.md              # Blog post scaffold template
├── assets/css/landing.css          # All custom styles (~1450 lines, Hugo Pipes processed)
├── content/                        # Flat structure with language suffixes
│   ├── _index.md / _index.fr.md    # Homepage (EN / FR)
│   ├── about.md / about.fr.md
│   ├── how-it-works.md / how-it-works.fr.md
│   ├── contact.md / contact.fr.md
│   └── blog/                       # Blog section
│       ├── _index.md / _index.fr.md
│       ├── introducing-episave.md
│       └── presentation-episave.fr.md
├── i18n/{en,fr}.yaml               # 48 translation keys per language
├── layouts/
│   ├── _default/baseof.html        # Base layout (requires Hugo 0.146.0+)
│   ├── index.html                  # Homepage — composes landing partials
│   ├── partials/
│   │   ├── landing/                # Modular landing sections (6 partials)
│   │   │   ├── hero.html           # Hero with animated SVG blob
│   │   │   ├── features.html       # 4 feature cards with Carbon icons
│   │   │   ├── how-it-works.html   # 3-step process (Wear → Detect → Alert)
│   │   │   ├── stats.html          # Key statistics section
│   │   │   ├── cta.html            # Donation/contact CTA
│   │   │   └── latest-posts.html   # Latest 3 blog posts
│   │   ├── extend_head.html        # Custom fonts, CSS, scroll animation JS
│   │   ├── extend_footer.html      # Footer extension point (currently empty)
│   │   ├── footer.html             # PaperMod footer override
│   │   └── json-ld.html            # Schema.org structured data
│   └── shortcodes/
│       └── donate-button.html      # Stripe donation button
├── static/
│   ├── admin/                      # Decap CMS (index.html + config.yml)
│   └── images/hero-watch.png       # Hero smartwatch image
├── themes/PaperMod/                # Git submodule (v8.0+, MIT license)
└── hugo.toml                       # All site config
```

## Content Structure

Content uses **flat files with language suffixes** (not directory-based i18n):
- English: `about.md`, `_index.md`
- French: `about.fr.md`, `_index.fr.md`

**Frontmatter patterns:**

Blog posts:
```yaml
title: ""
date: 2025-01-15
description: ""
tags: ["announcement", "launch"]
draft: false
```

Static pages:
```yaml
title: ""
description: ""
layout: "single"
url: "/custom-url/"
```

## Key Conventions

- **EN has no URL prefix** — `defaultContentLanguageInSubdir = false`. EN: `/about/`, FR: `/fr/about/`
- **Menus are per-language** in `hugo.toml` under `[languages.en.menu]` and `[languages.fr.menu]`
- **All UI text in `i18n/` files** — use `{{ i18n "key" | default "fallback" }}`, never hardcode
- **Internal links use `relLangURL`** for language-aware routing
- **Anchor links on homepage** — `/#features`, `/#how-it-works`, `/#donate`, `/#news`
- **Content must stay in sync** between EN and FR (parallel files)
- **PaperMod is a submodule** — override via `layouts/`, never edit `themes/PaperMod/`
- **Goldmark allows unsafe HTML** — `[markup.goldmark.renderer] unsafe = true`

## Design System (CSS Variables)

Defined in `assets/css/landing.css`:

```css
--epi-navy: #0B1D3A       /* headings */
--epi-blue: #0284C7       /* primary CTA */
--epi-teal: #0D9488       /* secondary */
--epi-indigo: #4F46E5     /* accent */
--epi-surface: #F8FAFC    /* background */
--epi-white: #FFFFFF      /* cards */
--epi-border: #E2E8F0     /* borders */
--epi-text: #0F172A       /* body text */
--epi-text-soft: #475569  /* secondary text */
```

**Typography**: Lexend (headings, 400-800) + Source Sans 3 (body, 400-600) via Google Fonts.

**Key CSS features**: animated SVG hero blob (20s morph cycle), scroll-triggered animations via Intersection Observer, sticky header with backdrop blur, dark mode variables, shimmer hover effects on buttons.

## Structured Data (JSON-LD)

Injected via `layouts/partials/json-ld.html`:
- **Organization** — all pages (name, logo, sameAs)
- **WebSite** — all pages (inLanguage per locale)
- **BreadcrumbList** — non-homepage pages
- **Article** — blog posts (headline, dates, author, publisher)
- **MedicalDevice** — homepage only (seizure detection purpose)

## Decap CMS

- **Admin UI**: `/admin/` (static HTML, no backend)
- **Backend**: GitHub OAuth via Cloudflare Worker (`sveltia-cms-auth.contact-247.workers.dev`)
- **Workflow**: Editorial (draft → publish)
- **Collections**: Blog posts (i18n enabled), EN pages (3 files), FR pages (3 files)
- **Media**: Uploads to `/static/images/`

## Hugo Specifics

- Go templates (`{{ }}` syntax) — [Hugo docs](https://gohugo.io/templates/)
- **Hugo Pipes** processes `assets/` (CSS fingerprinting, minification) — not served as-is
- **`static/`** files copied verbatim to output
- **Shortcodes** invoked in Markdown: `{{</* shortcode-name */>}}`
- **Partials** invoked in templates: `{{ partial "name.html" . }}`
- **Extension points**: `extend_head.html`, `extend_footer.html` (PaperMod hooks)
- **Output formats**: HTML, RSS, JSON (home page)

## Cloudflare Pages Deployment

- **Base URL**: `https://episave-landing.pages.dev`
- **Staging**: Every branch push creates a preview URL
- **Production**: Merge to `main` auto-deploys
- **Build command**: `hugo --minify`
- **Deploy command**: `npx wrangler pages deploy public`
- **Env var**: `HUGO_VERSION=0.147.0`
