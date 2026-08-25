# Content Authoring Guide

This directory holds all the textual content for the blog. The static site reads Markdown files at runtime and renders them with the bilingual layout. No build step is required.

## Directory Structure

```
src/content/
├── settings.txt        # Site identity, language config, UI translations
├── about/              # About-page bio (fixed path)
│   ├── index.md            # English (optional)
│   └── index.{lang}.md     # Second language, e.g. index.pt.md (optional)
├── archive/            # Blog posts (dated hierarchy)
│   ├── index.json
│   └── {yyyy}/{mm}/{dd}/{post-slug}/
│       ├── index.md            # English (optional)
│       └── index.{lang}.md     # Second language, e.g. index.pt.md (optional)
├── portfolio/          # Portfolio projects (same dated hierarchy as archive)
│   ├── index.json
│   └── {yyyy}/{mm}/{dd}/{project-slug}/
│       ├── index.md            # English (optional)
│       └── index.{lang}.md     # Second language, e.g. index.pt.md (optional)
└── templates/          # Copy-paste starter files (archive-post.md, portfolio-project.md, about-bio.md)
```

## File Naming

| File | Language |
|------|----------|
| `index.md` | English |
| `index.{lang}.md` | Second language: the code comes from `lang.code` in `settings.txt` (e.g. `index.pt.md` for Portuguese) |

Double-way visibility rule: content is visible ONLY in the languages in which it exists. Neither file is mandatory and there is NO fallback: if the active-language file is missing, the post/project is hidden from all listings (Home, Archive, Portfolio, search, tag results, pagination) and the detail route shows the not-found state. The About page shows an empty state when the active-language bio is missing. A monolingual English site needs just `index.md`.

## Settings File (`src/content/settings.txt`)

The single user-editable configuration file, `key = value` format with `#` comments: both full-line comments and inline comments on the same line as a key:

- `site.*` keys: identity used for the `<head>`, footer, and JSON-LD: `site.name`, `site.tagline`, `site.domain`, `site.author`, `site.jobTitle`, `site.linkedin`, `site.github`, `site.email`, `site.copyright`.
- Language config - `lang.enabled` (`true`/`false`), `lang.code` (2–3 letter code - ISO 639-1/639-2 - of the second language), `lang.label` (informational metadata only - the language name in its own language, never displayed; the toggle flash shows the uppercase `lang.code`, e.g. `PT`, `DE`). Set `lang.enabled = false` for a monolingual English site (the language toggle disappears). Deleting the file entirely also yields a monolingual English site (embedded defaults).
- UI translation keys (`nav.about`, `common.search`, `footer.copyright`, ...): values for the second language; English is embedded in the code as fallback.

### Inline comments and AI translation

Every UI translation key carries the English original as an inline comment (`key = value # English original`). Comment rules, applied identically by the runtime `src/js/settings.js` and the generator `scripts/generate-site.js`:

| Syntax | Meaning |
|--------|---------|
| `key = value # note` | **Inline comment**: everything after the first ` #` is ignored. The UI keys use this to carry the English original, e.g. `nav.about = Sobre # About`. |
| `key = value#note` | A `#` with **no space** before it is a literal part of the value. |
| `post.title = # Post title` | **Dynamic blank value**: the annotation is ignored, so the parsed value remains empty. |
| `# full line` | **Full-line comment**: the entire line is ignored. |

Values are trimmed and used verbatim (no quoting). Unknown keys are stored inert and never executed.

The file embeds an AI translation prompt in its header (see the sample below). Workflow:

1. **Declare the language first**: set `lang.code` to the 2–3 letter language code (e.g., `de`) and `lang.label` to the language name in its own language (e.g., `Deutsch`).
2. Copy the ENTIRE content of `settings.txt`: including the header block.
3. Paste it into your AI assistant, replacing `<your language>` in the embedded instruction with the full language name (e.g., "German"), and send it.
4. Replace the file with the AI output and save: the AI translates every UI value after `=`, keeps every key and the inline ` # English original` comments unchanged, and updates `lang.code`/`lang.label` to the declared language.

The inline ` # English original` comments survive the round trip, so you can always verify what each translated line means.

Edit this file, then commit and push. The GitHub Action regenerates the `<head>` of `index.html`, `sitemap.xml`, and `robots.txt` from it automatically (locally: `node scripts/generate-site.js`).

### Changing the second language

To switch the second language (e.g. from Portuguese to German), edit the `lang.*` keys:

```txt
lang.code = de
lang.label = Deutsch
```

Then **rename the second-language content files accordingly**: every `index.pt.md` in `archive/`, `portfolio/`, and `about/` becomes `index.de.md`. The site resolves files strictly by the declared `lang.code`; there is no automatic remapping, and files under the old code will no longer be found.

### Translating UI terms

Translate the **value on the right side of `=`**; the key must stay unchanged because the runtime looks the string up by key, and the ` # English original` inline comment stays unchanged as your reference:

```txt
# settings.txt: before
nav.about = Sobre # About

# settings.txt: after
nav.about = Acerca de # About
```

Any key left untranslated (or empty) falls back to the embedded English text. The full list of keys mirrors the English dictionary embedded in `src/js/settings.js`.

### Social links

The footer shows up to six social entries: X, Instagram, YouTube, GitHub, LinkedIn, and Email. Each network has a value key (`site.x`, `site.instagram`, `site.youtube`, `site.github`, `site.linkedin`, `site.email`) and a companion `site.<network>.enabled` flag. Values accept three forms:

| Form | Example | Result |
|------|---------|--------|
| **Handle**: a bare username/handle | `my-user` | Network base prefixed (e.g. X → `https://x.com/my-user`) |
| **Normalized**: no scheme, starting with the network domain | `youtube.com/@my-channel` | Rewritten to the canonical base (`https://www.youtube.com/@my-channel`) |
| **Full URL**: with an explicit scheme | `https://github.com/my-user` | Used verbatim (any domain) |

| Network | Key | Base prefix |
|---------|-----|-------------|
| X | `site.x` | `https://x.com/` |
| Instagram | `site.instagram` | `https://instagram.com/` |
| YouTube | `site.youtube` | `https://www.youtube.com/@` |
| GitHub | `site.github` | `https://github.com/` |
| LinkedIn | `site.linkedin` | `https://linkedin.com/in/` |
| Email | `site.email` | `mailto:`: always prefixed |

**Scheme-less values**: a value without a scheme is either a handle (`my-channel` → `https://www.youtube.com/@my-channel`) or, when it starts with the network domain, a normalized URL (`youtube.com/@my-channel` → `https://www.youtube.com/@my-channel`). For an external domain, always write the full `https://` URL: or leave the network disabled.

**Enabled flags**: a network appears only when `site.<network>.enabled = true` AND the value is non-empty. `false` hides it; an empty value hides it even when enabled. An absent flag keeps the network visible (backward compatible with forks that predate the flags). Leave unused networks disabled.

### Sample settings.txt (placeholders)

```txt
# ============================================================
# Site settings: single source of truth for site identity,
# language configuration, and UI translations.
# Format: `key = value` lines. Lines starting with `#` are
# comments. Values are trimmed and used verbatim (no quoting).
# Inline comments: `key = value # note`: everything after the
# first ` #` is ignored; a `#` with no space before it stays
# part of the value. Unknown keys are stored inert and never
# executed. The dynamic blank post title uses `post.title = # Post
# title`; its annotation is ignored and the parsed value stays empty.
# ============================================================

# ============================================================
# HOW TO TRANSLATE THIS FILE WITH AN AI ASSISTANT
# 1. Set the language fields below first:
#      lang.code  = 2–3 letter language code (e.g., pt, de, fr)
#      lang.label = language name in its own language
#                   (e.g., Português, Deutsch, Français)
# 2. In the instruction below, replace <your language> with the
#    full language name (e.g., "Portuguese").
# 3. Copy the ENTIRE content of this file (including this header).
# 4. Paste it into your AI assistant and send exactly this:
#    "Translate the UI values after '=' into <your language>.
#     Keep every key and every '#' English original unchanged.
#     Update lang.code with the 2–3 letter code and lang.label
#     with the language name in its own language.
#     Return the complete file content."
# 5. Replace this file with the AI output and save.
# ============================================================

# ===== Site identity =====
# Site display name (used in title, JSON-LD, About heading)
site.name = Your Name
# Site tagline (used in meta description fallback and JSON-LD)
site.tagline = your-tagline
# Site meta description (used in <meta name="description"> and JSON-LD)
site.description = Personal blog and portfolio: your-tagline
# Site domain (used for canonical URLs, sitemap.xml, robots.txt)
site.domain = yourdomain.com
# Author name (used in meta tags and JSON-LD Person)
site.author = Your Name
# Author job title (used in JSON-LD Person)
site.jobTitle = Your Job Title
# Social link values accept three forms:
#   handle: a bare username/handle (e.g. my-user): the network base is prefixed
#   normalized: no scheme, starting with the network domain (e.g. youtube.com/@my-channel)
# - rewritten to the canonical base
#   url: full profile URL with an explicit scheme (e.g. https://github.com/my-user)
# - used verbatim (any domain)
# Any other value WITHOUT a scheme is treated as a HANDLE and prefixed; for an
# external domain, always write the full https:// URL (or leave the network disabled).
# Email always uses mailto: + value.
# X (Twitter) profile URL or handle (footer link; hidden until enabled with a value)
site.x = your-username
# Show the X link in the footer: true = show, false = hide
site.x.enabled = false
# Instagram profile URL or handle (footer link; hidden until enabled with a value)
site.instagram = your-username
# Show the Instagram link in the footer: true = show, false = hide
site.instagram.enabled = false
# YouTube channel URL or handle (footer link; hidden until enabled with a value)
site.youtube = your-username
# Show the YouTube link in the footer: true = show, false = hide
site.youtube.enabled = false
# GitHub profile URL or handle (footer link + JSON-LD sameAs)
site.github = https://github.com/your-username
# Show the GitHub link in the footer: true = show, false = hide
site.github.enabled = true
# LinkedIn profile URL or handle (footer link + JSON-LD sameAs)
site.linkedin = https://linkedin.com/in/your-username
# Show the LinkedIn link in the footer: true = show, false = hide
site.linkedin.enabled = true
# Contact email (footer email link + JSON-LD)
site.email = you@example.com
# Show the email link in the footer: true = show, false = hide
site.email.enabled = true
# Copyright line shown in the footer; {year} expands to the current year on generate
site.copyright = © {year} Your Name - All rights reserved. # {year} expands to the current year on generate

# ===== Language =====
# Enable second language: true = bilingual, false = English only
lang.enabled = true
# Language code: 2–3 letters (ISO 639-1/639-2). Auto-normalized: uppercase and regional
# forms are accepted and converted (DE → de, de-DE → de, pt-BR → pt). Used for
# index.{code}.md files; the date locale derives from it (de → de-DE, pt → pt-BR).
# Non-code values (e.g. "english") disable the second language: the toggle hides.
# Common codes:
#   pt Portuguese · de German · fr French · es Spanish · it Italian
#   nl Dutch · sv Swedish · no Norwegian · da Danish · fi Finnish
#   pl Polish · cs Czech · hu Hungarian · ro Romanian · el Greek
#   tr Turkish · ru Russian · uk Ukrainian · ja Japanese · ko Korean
#   zh Chinese* · ar Arabic · he Hebrew · hi Hindi · th Thai
#   * zh maps to zh-CN via the locale map.
# Full list: https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes
lang.code = pt
# Human-readable language name: informational only, not displayed
lang.label = Português

# ===== UI translations (second language: Portuguese) =====
# Keys mirror src/js/settings.js EN_TRANSLATIONS. Missing keys fall
# back to the embedded English dictionary. The English original is
# appended after ` # ` on each line for reference.

# --- Navigation ---
nav.about = Sobre # About
nav.archive = Arquivo # Archive
nav.portfolio = Portfólio # Portfolio

# --- Common ---
common.search = Buscar... # Search...
common.orderBy = Ordenar por # Order by
common.newest = Mais recentes # Newest
common.oldest = Mais antigos # Oldest
common.az = A-Z # A-Z
common.za = Z-A # Z-A
common.noResults = Nenhum post encontrado para sua busca. # No posts found matching your search.
common.noPortfolioResults = Nenhum projeto encontrado para sua busca. # No portfolio items found matching your search.
common.loading = Carregando... # Loading...
common.skipToContent = Pular para o conteúdo # Skip to content
common.prevPage = Página anterior # Previous page
common.nextPage = Próxima página # Next page
common.page = Página {n} # Page {n}
common.copyCode = Copiar código # Copy code
common.toggleTheme = Alternar tema # Toggle theme
common.toggleLanguage = Alternar idioma # Toggle language
# {label} is replaced at runtime with the active language name
common.langToggleCurrent = Alternar idioma - atual: {label} # Toggle language - current: {label}
common.sortPosts = Ordenar posts # Sort posts
common.sortPortfolio = Ordenar portfólio # Sort portfolio

# --- Breadcrumb ---
breadcrumb.archive = Arquivo # Archive
breadcrumb.portfolio = Portfólio # Portfolio
breadcrumb.post = Post # Post
breadcrumb.notFound = Não encontrado # Not Found

# --- Post ---
# Keys published as stable contract but currently unused by the runtime: post.back, post.notFound.body, tag.prompt: translate them for future-proofing; the UI does not render them today.
post.title = # Post title
post.back = Voltar # Back
post.readMore = Ler mais # Read more
post.tags = Tags # Tags

# --- Post not-found state ---
post.notFound.body = O post que você procura não existe ou foi removido. # The post you're looking for doesn't exist or has been removed.
# {label} is replaced at runtime with lang.label (e.g. "Português")
post.notFound.unavailable = Este post ainda não está disponível em {label}. # This post is not available in {label} yet.

# --- Page not-found state (unknown routes) ---
page.notFound.title = Página não encontrada # Page not found
page.notFound.body = A página que você procura não existe. # The page you are looking for does not exist.

# --- About ---
# OPTIONAL override for the display name on the About page. Leave this line
# commented out and site.name is used automatically.
# about.name =

# --- Tag page ---
tag.prompt = Digite uma tag para filtrar os posts. # Type a tag to filter posts.

# --- Loading / error / empty states ---
loading.posts = Carregando posts... # Loading posts...
loading.projects = Carregando projetos... # Loading projects...
error.posts = Falha ao carregar os posts. # Failed to load posts.
error.projects = Falha ao carregar os projetos. # Failed to load projects.
empty.posts = Nenhum post ainda. # No posts yet.
empty.projects = Nenhum projeto ainda. # No projects yet.

# --- Footer ---
footer.copyright = © {year} Your Name - Todos os direitos reservados. # © {year} Your Name - All rights reserved.
```

Replace every placeholder (`Your Name`, `your-tagline`, `yourdomain.com`, `Your Job Title`, `your-username`, `you@example.com`) with your own values. There is **no automated placeholder check**: substitution is the owner's responsibility before publishing - a fork published without it exposes the placeholder text publicly.

## Frontmatter

Every Markdown file starts with YAML frontmatter delimited by `---`:

```markdown
---
title: "Post Title"
tags: [example, getting-started]
---

Body content in Markdown...
```

- Inline arrays (`tags: [a, b]`) split on `,` without respecting quotes, so a tag containing a comma is not representable in the inline form: use a multi-line array for such tags.
- Multi-line arrays require EXACTLY 2 spaces of indentation (`  - item`); any other indentation is silently ignored by the parser.

| Field | Required | Notes |
|-------|----------|-------|
| `title` | yes | Display title shown in listings and the post header. |
| `tags` | no | Array of strings. Used for tag-filtered views. |
| `date` | no | Falls back to the date encoded in the folder path. |
| `url` | no | Portfolio only: external link instead of in-site detail page. |

## Images

Markdown images must be served by the site itself. External `https://` image URLs are intentionally blocked by the Content-Security-Policy (`img-src 'self' data:`) - a privacy feature, not a bug: the site never contacts third-party hosts, so external images never load. Use site-served images (e.g. `src/assets/photo.jpg`, a relative path resolved against the site root) or inline `data:image/*` URLs. The leading-slash form `/src/assets/photo.jpg` is also accepted and equivalent - both resolve correctly at root and subpath deployments. A ready-to-use example image ships at `src/content/templates/example-image.svg` and is referenced by the sample post (`templates/archive-post.md`).

## Math (LaTeX via MathJax)

Posts are LaTeX-friendly: standard LaTeX is rendered by a self-hosted MathJax 3 bundle - no CDN, no external requests. The bundle and its fonts live under `src/assets/vendor/mathjax/` and load lazily: a page without math never downloads anything, so the ~1.3MB one-time cost applies only to posts that actually contain math.

| Syntax | Renders as |
|--------|------------|
| `` `$x^2 + \alpha$` `` | Inline math |
| `` `$$E = mc^2$$` `` (own line) | Display math, centered block |

The full LaTeX command set is available: fractions (`\frac{a}{b}`), roots (`\sqrt{x}`, `\sqrt[3]{x}`), sums/products with limits (`\sum_{i=1}^{n}`, `\prod_{i=1}^{n}`), Greek letters (`\alpha`, `\beta`, ...), operators, and more.

Math REQUIRES backtick delimiters around the dollars: `` `$...$` `` inline (one line) and `` `$$...$$` `` on its own line. Money text like `$5` or `R$ 100` stays literal without them. Math inside code blocks, link text, or image alt text is never interpreted. Invalid commands render as red literal text (MathJax's own error display) and never crash the page; markup or scripts inside the trigger are escaped and inert.

## Creating a Blog Post

1. Pick a date and slug: `src/content/archive/2026/08/15/my-post-slug/`
2. Copy `src/content/templates/archive-post.md` into that folder: it is a fully-exemplified sample post: it demonstrates every supported feature (paragraphs, nested lists, task lists, tables, inline and display math, fenced code with language, inline code, external and internal links, strikethrough, blockquotes, and a working image) with real content. Replace it with your own.
3. Save the copy as `index.md` and edit the content (English).
4. (Optional, bilingual sites) Save another copy as `index.{lang}.md`, e.g. `index.pt.md`, and write the second-language version.
5. Add the slug to `src/content/archive/index.json`:

   ```json
   ["2026/08/15/my-post-slug"]
   ```

6. Commit and push. The GitHub Action regenerates `index.json` automatically on push, so you can also skip step 5 and let the action add the entry.

## Creating a Portfolio Project

Same workflow as a blog post, under `src/content/portfolio/`. Start from `src/content/templates/portfolio-project.md`. Add the date-prefixed slug to `src/content/portfolio/index.json` (or let the action do it).

## Editing the About Page

The About page reads from a fixed path: `src/content/about/index.md` (English) and `src/content/about/index.{lang}.md` (second language). Start from `src/content/templates/about-bio.md` if you want a starter. Edit these files directly; no registry entry is needed.

## Automation

A GitHub Action regenerates `archive/index.json` and `portfolio/index.json` from the folder layout, and `index.html`/`sitemap.xml`/`robots.txt` from `settings.txt`, on every push to `main` that touches Markdown or settings files. You can still edit those files by hand if you prefer, but the action will rewrite them.
