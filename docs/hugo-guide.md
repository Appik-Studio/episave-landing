# Hugo Quick Reference

## Project Structure

```
hugo.toml          # Global config (title, theme, params, menus)
content/           # Markdown (.md) with YAML/TOML frontmatter
layouts/           # Go templates (override theme)
assets/            # Files processed by Hugo Pipes (CSS, JS)
static/            # Copied as-is to build output (images, favicon)
themes/            # Themes (usually git submodule)
i18n/              # Translations (YAML/JSON)
```

## Go Template Syntax — `{{ }}`

```go
{{ .Title }}                    // Page variable
{{ site.Title }}                // Global variable
{{ .Param "key" }}              // Param (page > site fallback)
{{ site.Params.author }}        // Direct site param

{{ if condition }}...{{ end }}
{{ with value }}...{{ end }}    // if + set context (.)
{{ range .Pages }}...{{ end }}  // loop

{{ partial "name.html" . }}    // include a partial (pass context .)
{{ i18n "key" | default "fb" }}// translation with fallback
{{ "text" | markdownify }}     // pipe = filter (like Jinja)
{{ relLangURL "/about/" }}     // language-relative URL
```

## Template Types

| File | Role |
|------|------|
| `layouts/index.html` | Homepage |
| `layouts/_default/single.html` | Single page (article) |
| `layouts/_default/list.html` | List (section, taxonomy) |
| `layouts/partials/*.html` | Reusable fragments |
| `layouts/shortcodes/*.html` | Components usable in Markdown |

## Content — Frontmatter + Markdown

```markdown
---
title: "My Article"
date: 2026-02-27
draft: false
tags: ["epilepsy"]
---

Markdown content here.

{{</* my-shortcode param="value" */>}}
```

## Hugo Pipes (assets/)

```go
// In a template — process, minify, fingerprint
{{ $css := resources.Get "css/landing.css" | minify | fingerprint }}
<link rel="stylesheet" href="{{ $css.Permalink }}">
```

`assets/` = processed by Hugo. `static/` = copied raw.

## Multilingual

```toml
# hugo.toml
defaultContentLanguage = "en"

[languages.en]
  languageName = "English"
  weight = 1

[languages.fr]
  languageName = "Français"
  weight = 2
```

- Content: `content/en/blog/post.md` + `content/fr/blog/post.md`
- Menus: defined per language in `[languages.fr.menu]`
- UI strings: `i18n/en.yaml` + `i18n/fr.yaml`

## Theme Override

Never edit `themes/`. Copy the file to `layouts/` at the same path:

```
themes/PaperMod/layouts/partials/footer.html  ← original
layouts/partials/footer.html                   ← your override (takes priority)
```

## Commands

```bash
hugo server -D    # Dev with drafts, hot reload on :1313
hugo --minify     # Production build into public/
```

## Lookup Order (simplified)

Hugo looks for templates in this order:

`layouts/` (your project) → `themes/PaperMod/layouts/` (theme)

## Decap CMS (Content Management)

### Access

Editors go to `/admin/` (e.g. `https://episave-landing.pages.dev/admin/`). Authentication is handled via GitHub OAuth through a Cloudflare Worker ([Sveltia CMS Auth](https://github.com/sveltia/sveltia-cms-auth)).

### How It Works

1. Editor logs in with GitHub at `/admin/`
2. Edits content via the visual UI (Markdown editor, image uploads)
3. In `editorial_workflow` mode, changes are saved as **draft PRs** on GitHub
4. Publishing merges the PR → triggers Cloudflare Pages deploy

### Config Files

```
static/admin/
├── index.html      # Loads Decap CMS JS, registers preview templates
└── config.yml      # Collections, fields, backend, i18n settings
```

### config.yml Structure

```yaml
backend:
  name: github
  repo: Appik-Studio/episave-landing
  branch: main
  base_url: https://sveltia-cms-auth.contact-247.workers.dev  # OAuth proxy

publish_mode: editorial_workflow   # Drafts → Review → Publish

media_folder: static/images        # Where uploads are saved
public_folder: /images             # URL path in content

i18n:
  structure: multiple_files        # post.md + post.fr.md
  locales: [en, fr]
  default_locale: en
```

### Collections

| Collection | Type | Content Path |
|-----------|------|-------------|
| `blog` | Folder (create new) | `content/blog/` |
| `pages-en` | File (fixed pages) | `content/about.md`, `content/how-it-works.md`, `content/contact.md` |
| `pages-fr` | File (fixed pages) | `content/about.fr.md`, `content/how-it-works.fr.md`, `content/contact.fr.md` |

### Adding a New Collection

In `static/admin/config.yml`, add to `collections:`:

```yaml
- name: my-collection
  label: "My Collection"
  folder: content/my-collection
  create: true
  i18n: true
  fields:
    - { label: "Title", name: "title", widget: "string", i18n: true }
    - { label: "Date", name: "date", widget: "datetime" }
    - { label: "Body", name: "body", widget: "markdown", i18n: true }
```

### Common Widgets

| Widget | Use |
|--------|-----|
| `string` | Short text (title, description) |
| `markdown` | Rich text body |
| `datetime` | Date picker |
| `boolean` | Toggle (draft, featured) |
| `list` | Tags, categories |
| `image` | File upload to `media_folder` |
| `select` | Dropdown from predefined options |

### Important Notes

- `media_folder` is relative to project root (`static/images`)
- `public_folder` is the URL prefix in generated content (`/images`)
- Blog uses `i18n: true` with `multiple_files` → creates `post.md` (EN) + `post.fr.md` (FR)
- Static pages use `file` type (fixed path, no create)
