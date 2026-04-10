# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EpiSave marketing website — a Hugo static site with the PaperMod theme (vendored), deployed on Cloudflare Pages. Multilingual (English + French) with Decap CMS for content editors. The product is an open-source medical device for seizure detection using Android smartwatches.

**GitHub repo**: `Appik-Studio/episave-landing`

## Commands

```bash
# Dev server (with drafts)
hugo server -D

# Production build
hugo --minify

# Clone (simple — no submodules needed)
git clone <repo-url>
```

No `package.json` or Node.js dependencies. Hugo binary only. Required version: `0.147.0`.

## Architecture

```
episave-site/
├── archetypes/blog.md              # Blog post scaffold template
├── assets/css/landing.css          # All custom styles (~1450 lines, Hugo Pipes processed)
├── content/                        # Flat structure with language suffixes
│   ├── _index.md / _index.fr.md    # Homepage (EN / FR) — all section text in frontmatter
│   └── blog/                       # Blog section
│       ├── _index.md / _index.fr.md
│       ├── introducing-episave.md
│       └── presentation-episave.fr.md
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
├── static/
│   ├── admin/                      # Decap CMS (index.html + config.yml)
│   └── images/hero-watch.png       # Hero smartwatch image
├── themes/PaperMod/                # Vendored (v8.0+, MIT license)
└── hugo.toml                       # All site config
```

## Content Structure

Two content types only: **homepage** and **blog posts**.

Content uses **flat files with language suffixes** (not directory-based i18n):
- English: `_index.md`
- French: `_index.fr.md`

**Homepage frontmatter** — all text is in structured frontmatter (editable via Decap CMS):
```yaml
title: "..."          # SEO title
description: "..."    # SEO description
hero:                 # Hero section (eyebrow, title, subtitle, CTAs, statuses)
features:             # 4 feature cards (title, subtitle, feature1-4_title/desc)
how_it_works:         # 3 steps (title, step1-3_title/desc)
stats:                # 3 stats (stat1-3_number/label)
cta:                  # CTA section (title, description, button_text)
news:                 # News section (title, read_more, view_all)
```

**Blog posts frontmatter:**
```yaml
title: ""
date: 2025-01-15
description: ""
tags: ["announcement", "launch"]
draft: false
```

## Key Conventions

- **EN has no URL prefix** — `defaultContentLanguageInSubdir = false`. EN: `/`, FR: `/fr/`
- **Pages**: only homepage (`/` and `/fr/`) and blog (`/blog/` and `/fr/blog/`)
- **Menus are per-language** in `hugo.toml` under `[languages.en.menu]` and `[languages.fr.menu]`
- **All homepage text in frontmatter** — use `{{ .Params.section.field | default "fallback" }}` in partials
- **Internal links use `relLangURL`** for language-aware routing
- **Anchor links on homepage** — `/#features`, `/#how-it-works`, `/#donate`, `/#news`
- **Content must stay in sync** between EN and FR (`_index.md` / `_index.fr.md`)
- **PaperMod is vendored** — prefer overriding via `layouts/`, edit `themes/PaperMod/` only if needed
- **Goldmark allows unsafe HTML** — `[markup.goldmark.renderer] unsafe = true`

## Design System (CSS Variables)

Defined in `assets/css/landing.css` — "Clinical Blue & Slate" theme:

```css
--epi-navy: #0F172A       /* navy — headings */
--epi-blue: #0284C7       /* clinical blue — primary CTA, buttons */
--epi-teal: #0D9488       /* teal — secondary accent */
--epi-indigo: #4F46E5     /* indigo — accent */
--epi-surface: #F8FAFC    /* cool white — background */
--epi-white: #FFFFFF      /* cards */
--epi-border: #E2E8F0     /* borders */
--epi-text: #0F172A       /* body text */
--epi-text-soft: #64748B  /* secondary text */
```

**Typography**: IBM Plex Sans (400-700 + italics) via Google Fonts — single typeface for institutional clinical precision.

**Key CSS features**: subtle SVG hero blob (30s morph cycle), scroll-triggered animations via Intersection Observer, sticky header with backdrop blur, dark mode variables, `prefers-reduced-motion` support.

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
- **Collections**: Blog posts (i18n enabled), Homepage EN (structured frontmatter), Homepage FR (structured frontmatter)
- **Media**: Uploads to `/static/images/`

## Hugo Specifics

- Go templates (`{{ }}` syntax) — [Hugo docs](https://gohugo.io/templates/)
- **Hugo Pipes** processes `assets/` (CSS fingerprinting, minification) — not served as-is
- **`static/`** files copied verbatim to output
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

## Design Context

### Users
Mixed audience in roughly equal measure:
- **Patients & caregivers** — people living with epilepsy and their families seeking safety solutions
- **Medical professionals** — neurologists, researchers, and clinicians evaluating the device
- **Donors & partners** — NGOs, foundations, and individuals considering funding or collaboration

### Brand Personality
**Clinical, precise, credible.** The site should feel like a serious medical-tech product (think Withings/Oura — clean, premium, quietly authoritative). Target emotion: confidence & trust.

### Aesthetic Direction
- Clean medical-tech aesthetic. Premium and restrained, not warm or playful.
- References: Withings, Oura. Anti-references: charity/nonprofit aesthetics, consumer tech hype.
- Light mode primary. Current purple & amber palette is **open to reconsideration** for better clinical fit.
- Typography: IBM Plex Sans (400-700 + italics) — single typeface for institutional clinical precision.

### Design Principles
1. **Clinical credibility over warmth** — restrained color, precise alignment, generous whitespace.
2. **Information density matched to audience** — stats and evidence immediately scannable.
3. **Quiet confidence** — clarity over flashiness, subtle animations respecting `prefers-reduced-motion`.
4. **Accessible by default** — WCAG AA baseline. Avoid rapid flashing given epilepsy context.
5. **Bilingual parity** — EN/FR equally polished, accommodate French text expansion (~15-20%).
