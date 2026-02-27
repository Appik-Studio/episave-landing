# EpiSave Website

Static website for the EpiSave seizure detection project, built with [Hugo](https://gohugo.io/) + [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme.

## Quick Start

```bash
# Clone with submodules (PaperMod theme)
git clone --recurse-submodules https://github.com/Appik-Studio/episave-landing.git
cd episave-landing

# Run locally
hugo server -D

# Build for production
hugo --minify
```

## Project Structure

```
episave-site/
├── archetypes/          # Content templates
├── assets/css/          # Custom CSS (processed by Hugo Pipes)
├── content/
│   ├── en/              # English content
│   │   ├── blog/        # Blog posts (Markdown)
│   │   ├── about.md
│   │   ├── contact.md
│   │   └── how-it-works.md
│   └── fr/              # French content (mirrored structure)
├── i18n/                # Translation strings (en.yaml, fr.yaml)
├── layouts/
│   ├── index.html       # Homepage (landing page)
│   ├── partials/
│   │   ├── landing/     # Landing page sections (hero, features, etc.)
│   │   ├── json-ld.html # Structured data (Schema.org)
│   │   └── extend_head.html  # PaperMod head extension
│   └── shortcodes/      # Hugo shortcodes (donate-button, etc.)
├── static/
│   ├── admin/           # Decap CMS (accessible at /admin/)
│   └── images/          # Static images
├── hugo.toml            # Hugo configuration
└── themes/PaperMod/     # Theme (git submodule)
```

## Content Management (Decap CMS)

Editors can manage content at `https://episave-landing.pages.dev/admin/` — no local setup required.

### Setup GitHub App for Decap CMS

1. Create a GitHub App: https://github.com/settings/apps/new
   - **Homepage URL**: `https://episave-landing.pages.dev`
   - **Callback URL**: `https://sveltia-cms-auth.contact-247.workers.dev/callback`
   - **Permissions**: Contents (Read & Write), Pull requests (Read & Write)
   - Install on `Appik-Studio/episave-landing` only
2. Deploy [Sveltia CMS Auth](https://github.com/sveltia/sveltia-cms-auth) on Cloudflare Workers (one-click deploy)
3. Set `GITHUB_CLIENT_ID` and `GITHUB_CLIENT_SECRET` env vars on the Worker
4. `base_url` is already configured in `static/admin/config.yml`

## Deployment (Cloudflare Pages)

- **Repo**: `Appik-Studio/episave-landing`
- **URL**: `https://episave-landing.pages.dev`

### Staging + Production workflow

1. **Staging**: Every push to any branch creates a preview URL automatically
2. **Production**: Merge to `main` → auto-deploys to production

### Cloudflare Pages Build Settings

- **Build command**: `hugo --minify`
- **Deploy command**: `npx wrangler pages deploy public`
- **Environment variable**: `HUGO_VERSION` = `0.147.0`

## SEO Checklist

- [x] JSON-LD structured data (Organization, WebSite, BreadcrumbList, Article, MedicalDevice)
- [x] Open Graph & Twitter Card meta tags (via PaperMod)
- [x] Multilingual sitemaps (auto-generated)
- [x] robots.txt (auto-generated)
- [x] Semantic HTML structure
- [x] Breadcrumbs in JSON-LD
- [ ] Google Search Console verification
- [ ] Submit sitemaps to Google/Bing

## Customization

### Adding a Stripe donation link

Edit `layouts/shortcodes/donate-button.html` and replace `YOUR_STRIPE_PAYMENT_LINK` with your actual Stripe Payment Link URL.

### Adding a contact form

Options:
- [Formspree](https://formspree.io/) — embed form in `contact.md`
- [Tally](https://tally.so/) — embed form
- Simple `mailto:` link (already in place)

### Images

Place images in `static/images/`. Required placeholders:
- `logo.png` — EpiSave logo
- `hero-watch.png` — Hero section image
- `og-episave.png` — Open Graph sharing image (1200×630px)

## License

[MIT](LICENSE)
