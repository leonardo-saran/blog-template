# Customization Guide

This guide covers everything needed to personalize and deploy a fork of `blog-template`: adaptive defaults, the `settings.txt` reference, custom domains, localization, content authoring, local development, deployment, site generation, and pre-publication checks.

## Adaptive defaults

The template has no hard-coded first-visit language or theme. When a visitor arrives with no stored choice, both defaults are detected from the environment: and the site keeps following the environment until the visitor makes an explicit choice.

**Language.** On first access the active language is detected from the browser: `navigator.languages` is read in preference order, and the first entry matching English (`en`) or your configured second-language code (`lang.code`) wins. If the browser speaks neither, English is the universal fallback. The detected value never persists: it is re-evaluated on every visit, and a browser `languagechange` event re-detects live. Clicking the language toggle stores the choice in `localStorage` and freezes reactivity from then on.

**Theme.** On first access the theme follows the operating system via `prefers-color-scheme` - dark or light, applied without storing anything. A `change` listener keeps the site in sync in real time: switch your OS theme and the site switches with it. Clicking the theme toggle stores the choice in `localStorage` and freezes reactivity from then on. The dark palette lives in a single `[data-theme="dark"]` token block: a tiny same-origin script (`src/js/theme-boot.js`) in the `<head>` sets `data-theme` synchronously before the stylesheet applies, so a dark-OS visitor gets the dark shell from the first paint - no flash. **Trade-off**: visitors with JavaScript disabled see the light shell - the site is a JS-required SPA. A deeper no-JS fallback is deferred.

**Persistence rule.** Auto-detected values never persist; only explicit toggles persist; a stored choice freezes reactivity.

## Customization: `src/content/settings.txt` is the file to edit

Everything site-wide is driven by a single file: `src/content/settings.txt`. It uses a simple `key = value` format with `#` comments - both full-line comments and inline comments on the same line as a key - and holds three groups of settings:

1. **Site identity** (`site.*`): your name, tagline, domain, job title, social links, email, and copyright line. These feed the footer, the generated `<head>`, Open Graph tags, Twitter Cards, JSON-LD structured data, `sitemap.xml`, and `robots.txt`.
2. **Language configuration** (`lang.*`): enables/disables the bilingual toggle, declares the second-language code and its human-readable label (informational metadata only).
3. **UI translations** (`nav.*`, `common.*`, `breadcrumb.*`, `post.*`, `tag.*`, `loading.*`, `error.*`, `empty.*`, `footer.*`): the second-language text for every interface string. English is embedded in the code as fallback; missing keys render in English.

### Inline comments and AI translation

Every UI translation key carries the English original as an inline comment (`key = value # English original`): the classic translation-file pattern. Both parsers (the runtime `src/js/settings.js` and the generator `scripts/generate-site.js`) apply the same rules:

| Syntax | Meaning |
|--------|---------|
| `key = value # note` | **Inline comment**: everything after the first ` #` is ignored. The UI keys use this to carry the English original, e.g. `nav.about = Sobre # About`. |
| `key = value#note` | A `#` with **no space** before it is a literal part of the value. |
| `post.title = # Post title` | **Dynamic blank value**: the annotation is ignored, so the parsed value remains empty. |
| `# full line` | **Full-line comment**: the entire line is ignored. |

Values are trimmed and used verbatim (no quoting). Unknown keys are stored inert and never executed.

The file itself embeds an AI translation prompt in its header (visible in the sample below). To translate the UI into your second language:

1. **Declare the language first**: set `lang.code` to the 2–3 letter language code (e.g., `de`) and `lang.label` to the language name in its own language (e.g., `Deutsch`). The code is case-insensitive and auto-normalized: `DE`, `de-DE`, `pt_BR` are all accepted and converted to the lowercase primary code (`de`, `pt`); only non-code values (e.g. `english`) fall back to monolingual mode.
2. **Copy** the ENTIRE content of `src/content/settings.txt`: including the header comment block.
3. **Paste** it into your AI assistant, replacing `<your language>` inside the embedded instruction with the full language name (e.g., "German"), and send it.
4. **Replace** the file with the AI output and save: the AI translates every UI value, keeps every key and every ` # English original` comment unchanged, and updates `lang.code`/`lang.label` to the declared language.

The inline ` # English original` comments survive the round trip, so you can always verify what each translated line means.

### Full settings.txt sample (placeholders)

Copy this into `src/content/settings.txt` and replace every placeholder with your own values:

> **Social link values**: each social key accepts a bare handle (`my-user` - the network base is prefixed), a scheme-less network-domain value (`youtube.com/@my-channel` - normalized to `https://www.youtube.com/@my-channel`), or a full profile URL with an explicit scheme (`https://github.com/my-user` - used verbatim). Without `https://` the value is a handle of that network, unless it starts with the network domain - external domains always need the full `https://` URL. Email always uses `mailto:` + the value. A network is shown only when its `.enabled` flag is `true` and the value is non-empty. Full table: `src/content/README.md`.

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
#     with the language name in its native language.
#     Return the complete file content."
# 5. Replace your file with the AI output and save.
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
# Author name (used in the meta author tag)
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
# Enable the second language: true = bilingual, false = English only
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
# Keys mirror the embedded English dictionary. Missing keys fall
# back to the embedded English. The English original is appended
# after ` # ` on each line for reference.

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

### Customization checklist

| Placeholder | Replace with |
|-------------|--------------|
| `Your Name` | your display name: set once in `site.name`/`site.author`; the About heading and footer copyright line are regenerated automatically from `site.name`/`site.copyright` by `npm run generate` |
| `your-tagline` | a one-sentence site description |
| `yourdomain.com` | your custom domain (`site.domain`; plus a committed `CNAME` file only when using a custom domain: see below) |
| `Your Job Title` | your job title |
| `your-username` | your GitHub/LinkedIn username in the profile URLs |
| `you@example.com` | your contact email |
| `LICENSE:3` | copyright name AND year: update to yours before publishing |

**Custom domain (optional).** The template does not ship a `CNAME` file. Forks WITHOUT a custom domain must NOT create one: GitHub Pages serves the default `https://<username>.github.io/<repo>/` URL automatically. Forks WITH a custom domain create a `CNAME` file at the repository root containing a single line with the domain (e.g. `www.example.com`), commit it normally (`git add CNAME && git push`), and configure the DNS records at the registrar plus the domain in the repository's Pages settings. There is no check on the `CNAME`: committing it with a placeholder domain (e.g. `yourdomain.com`) simply breaks Pages activation instead of failing a gate - the owner must ensure the `CNAME` holds a real domain they control.

Then add your profile image at `src/assets/profile.jpg` (400x400px recommended): it is shown automatically on the About page and in social cards when present.

**Favicon (recommended).** The `<head>` ships a minimal empty favicon link (`<link rel="icon" href="data:,">`) that stops browsers from requesting the default `/favicon.ico` on first visit. To brand your fork with a real icon set, drop the files into the site and add the matching `<link>` tags in `index.html`:

| File | Suggested location | Link tag |
|------|--------------------|----------|
| `favicon.svg` | `src/assets/favicon.svg` (or root) | `<link rel="icon" type="image/svg+xml" href="src/assets/favicon.svg">` |
| `favicon.ico` | repository root (legacy browsers) | replace the `data:,` link with `<link rel="icon" href="/favicon.ico">` |
| `apple-touch-icon.png` (180x180) | `src/assets/apple-touch-icon.png` | `<link rel="apple-touch-icon" href="src/assets/apple-touch-icon.png">` |

The generator preserves committed `<head>` fragments it does not own - favicon links, preconnect, verification tags, and any other custom tag - across regenerations, so your customizations survive every automated head rewrite (only the generated title/description/Open Graph/Twitter/JSON-LD are rewritten). If you edit `index.html` manually (e.g. to add favicon links), run `npm run generate` (or push to `main` - the workflow triggers on `index.html` changes) to regenerate `sitemap.xml`/`robots.txt` from your `settings.txt`.

**License.** `LICENSE` ships with a placeholder copyright: line 3 reads `Copyright (c) 2026 Your Name`. Update the name AND the year to yours before publishing; the rest of the MIT notice stays as is.

### Pre-publication checklist

The template does NOT ship an automated placeholder check: no script scans for placeholders, so nothing fails or warns when one survives a push. Substitution is entirely your responsibility. Before publishing your fork, verify nothing was left unpersonalized:

1. **Search for placeholders** - manually replace every placeholder (`Your Name`, `yourdomain.com`, `your-username`, `you@example.com`, `your-tagline`, `Your Job Title`) in `src/content/settings.txt`, `LICENSE`, and any committed `CNAME`, plus the generated artifacts (`index.html`, `sitemap.xml`, `robots.txt`, `404.html`, `package.json`) before publishing. Publishing without substitution exposes the placeholder text publicly on your live site. The About heading, footer copyright line, and the language-toggle flash code (the uppercase `lang.code`, e.g. `PT`) in `index.html` are regenerated automatically from `site.name`/`site.copyright`/`lang.code` on `npm run generate`, so those body fallbacks need no manual edit - just personalize the settings values.
2. **Update the LICENSE** copyright name and year (see above).
3. **Add your profile image** at `src/assets/profile.jpg` (optional: see above).
4. **Push to `main`** and confirm the GitHub Action regenerates the site files. The workflow commits the regenerated artifacts automatically, and the deployment is green once the checklist is complete.
5. **Verify the live site**: visit every page, toggle both languages and themes, and confirm your social links resolve.

**Offline forks (no GitHub Actions).** The regeneration step only runs on pushes to `main` with Actions enabled. If you publish the fork without the workflow (e.g. copying the folder to another host or a repo with Actions disabled), the committed `sitemap.xml` `lastmod` values and the generated site files go stale as you edit content - run `npm run generate` locally after your final content edit, before publishing, to refresh them. The same applies to language changes: after editing `lang.code` or `lang.label`, run `npm run generate` (or push with the workflow) - the `<head>` and its Open Graph meta (including `og:locale:alternate`) are regenerated from these keys, and skipping the step would publish the old locale in the social tags.

### Language configuration

The template is bilingual by default. To make it your own:

| Setting | Value | Effect |
|---------|-------|--------|
| `lang.enabled = true` | Bilingual | The language toggle appears in the header. |
| `lang.enabled = false` | Monolingual English | The language toggle disappears entirely. Only `index.md` files are resolved. |
| `lang.code = pt` | Second language | 2–3 letter code (ISO 639-1/639-2): content files use `index.{code}.md`, e.g. `index.pt.md`. |
| `lang.label = Português` | Informational name | Informational metadata only: never displayed; the toggle flash shows the uppercase `lang.code` (e.g. PT, DE). |

To change the second language (e.g. from Portuguese to German), update `lang.code = de` and `lang.label = Deutsch`, then **rename the second-language content files** from `index.pt.md` to `index.de.md` across `archive/`, `portfolio/`, and `about/`. The site resolves files strictly by the declared `lang.code`: there is no automatic remapping.

> **Locale note (`pt` → `pt-BR`).** Date formatting maps the second-language code to an Intl locale: `pt` maps to `pt-BR` by design (the template's default author is Brazilian and pt-BR is the largest Portuguese-speaking audience). Other `pt` locales (e.g. European `pt-PT`) are NOT derived automatically: you would need to change the `pt` → `pt-BR` special case in `resolveDateLocale` (`src/js/app.js`). Non-`pt` codes derive their region automatically (`de` → `de-DE`, `fr` → `fr-FR`); English always formats as `en-US`.

### Translating UI terms

The interface ships in English with the second-language values read from `settings.txt`. To translate an interface term, edit the **value on the right side of the equals sign**; the key must stay unchanged because the runtime looks the string up by key, and the ` # English original` inline comment stays unchanged as your reference:

```txt
# settings.txt: before
nav.about = Sobre # About

# settings.txt: after (translating "About" to Spanish)
nav.about = Acerca de # About
```

Any key you leave untranslated (or with an empty value) falls back to the embedded English text. The sample block above shows the pattern for every key group.

### Double-way visibility rule

Content is visible ONLY in the languages in which it exists. `index.md` is English and `index.{lang}.md` is the second language: **neither file is mandatory and there is NO fallback**. If the active-language file is missing, the post or project is hidden from all listings (Home, Archive, Portfolio, search, tag results, pagination) and the detail route shows the not-found state. The About page shows an empty state when the active-language bio is missing. A monolingual English site needs just `index.md`.

> **SEO: static vs runtime language.** `index.html` ships with `<html lang="en">` and the generator preserves it. The template is English-primary by design - `index.md` is always the base content - so a no-JS crawler always receives English. The runtime syncs `document.documentElement.lang` to the active language for JS visitors (which is what JS-executing search engines observe). `lang.code` is never injected into the static tag.

> **JavaScript required.** This is a JavaScript-required SPA: the runtime fetches the content, renders the Markdown, and applies the interface labels, language, and theme. Without JavaScript, visitors see a limited static shell - the About section shows the placeholder initials and the untranslated static English labels - and no content (posts, projects, bio) renders. There is no `<noscript>` fallback by design: the template targets modern browsers and keeps the zero-build, zero-dependency footprint.

> **Console logging policy.** The runtime logs load failures to the console deliberately (e.g. `section:post load failed`) so a broken deployment is diagnosable from the browser. Every call logs a single **static identifier** - never dynamic data such as paths, slugs, URLs, or error messages - so failures surface without leaking site-specific information. Keep it that way: do not add dynamic content to `console.error` calls.

## Adding content

The full authoring guide is in [`src/content/README.md`](../src/content/README.md). Short version:

- Posts live under `src/content/archive/YYYY/MM/DD/your-slug/`.
- Portfolio projects live under `src/content/portfolio/YYYY/MM/DD/your-slug/`.
- The About page lives at `src/content/about/index.md` and `index.{lang}.md` (e.g. `index.pt.md`).
- `src/content/about/.gitkeep` is a placeholder directory keeper: it exists only so the empty `about/` folder is tracked by git. Remove it once you add the first About content file (`index.md` and/or `index.{lang}.md`); until then the About section shows an empty state when no active-language bio exists (double-way rule).
- `index.md` is English, `index.{lang}.md` is the second language: both optional, no fallback.
- Copy a starter from `src/content/templates/`, edit it, save into the right dated folder, commit, push.
- **Slugs must match the runtime contract** `/^[a-zA-Z0-9]+(\/[a-zA-Z0-9\-]+)*$/`: date-prefixed paths of the form `{yyyy}/{mm}/{dd}/{slug}` where the first segment is alphanumeric only (no leading hyphen), later segments may add hyphens, segments are separated by single slashes, and leading, trailing, or consecutive slashes are rejected. No Unicode characters, no underscores, no spaces: so a folder like `meu-post-ç/` simply will not appear.

> **Images and CLS.** Markdown images have no semantic dimensions, so the browser cannot reserve space before the lazy-loaded image arrives - a layout shift is possible on slow connections. The stylesheet mitigates it: `height: auto` preserves the aspect ratio as the width resolves, and `display: block` removes the inline baseline gap. Fully eliminating the shift would require width/height attributes from build-time image analysis - deliberately out of scope for the zero-build pipeline. If CLS matters most for a specific image, set `width` and `height` attributes in the Markdown HTML directly. Inline `data:image/*` srcs are allowed for images (the CSP allows `img-src data:`); that carve-out applies to `<img>` ONLY - never extend it to iframe/object/embed, where inline data would become executable content.

> **Math (LaTeX).** Posts are LaTeX-friendly: math is rendered by a self-hosted MathJax 3 bundle under `src/assets/vendor/mathjax/`. The syntax and the money-safe rule are documented in [the authoring guide](../src/content/README.md); loading is lazy, so only pages that contain math pay the ~1.3MB one-time download. No CSP changes are needed: the bundle, its CHTML stylesheet, and the fonts are all served same-origin.

The GitHub Action rewrites the content registry for you, so you do not have to edit `index.json` by hand.

## Local development

The site has no build pipeline. From the repository root, run any static file server:

```bash
python3 -m http.server 8000
# then open http://localhost:8000 in your browser
```

Equivalent alternatives: `npx serve .` or any other static server pointed at the project root. No installation step is required.

> **Local previews must use `localhost`.** The Content-Security-Policy includes `upgrade-insecure-requests`, which upgrades HTTP fetches to HTTPS: so a preview opened over a network IP (e.g. `http://192.168.x.x:8000`) will not load content. Serve from `localhost` or with TLS; the documented `python3 -m http.server 8000` example (localhost:8000) is unaffected.

## Deployment

The site is designed for GitHub Pages.

1. Push your customized repo to GitHub.
2. Open Settings → Pages.
3. Under "Build and deployment", choose "Deploy from a branch", branch `main`, folder `/ (root)`.
4. (Optional) To use a custom domain, add a `CNAME` file with your domain at the repository root and commit it (`git add CNAME && git push`), then configure the DNS records at your registrar to point to GitHub Pages.

> **Deep links and the 404 fallback.** The template is a History API SPA: every page lives at a real path (e.g. `/post/2026/08/15/my-post-slug/`). GitHub Pages only serves files that exist, so the committed [404.html](../404.html) is what makes deep links work - for any unknown path it stores the requested path in `sessionStorage` and redirects to the deployment root: `/` for root deployments (e.g. `user.github.io/`), `/repo/` for project pages (`user.github.io/repo/` - detected when the first pathname segment is not a known route such as `about`, `archive`, `portfolio`, `post`, or `tag`); on boot the app restores the path from `sessionStorage` and renders the deep-linked page. The storage write is best-effort (wrapped in try/catch): when `sessionStorage` is denied - e.g. Safari Private Mode - the redirect still runs, so a visitor is never stranded on a blank page. Keep `404.html` in place; deleting it breaks direct links to individual posts.

> **Sitemap URLs and the 404 shell.** `sitemap.xml` lists the real section and post URLs (`/about`, `/archive`, `/portfolio`, `/post/{slug}`), but those paths have no physical file on plain GitHub Pages: a request returns HTTP 404 with the shell, and the page only appears after the shell's client-side redirect is followed by a JS-capable client. Crawlers that execute JavaScript (e.g. Googlebot) follow the redirect and index the SPA content; crawlers that do not see 404s - an inherent trade-off of the zero-cost Pages hosting model. If your fork needs crawl-time HTTP 200 responses for every sitemap URL, you can deploy on a SPA-fallback host (Netlify, Vercel, or any server with rewrite rules), but ONLY when the rewrite is configured correctly: it must serve `index.html` for unknown SPA routes WHILE excluding the real asset paths (e.g. `/src/*` must pass through to the actual files), and the site must be served at the domain root - the app derives its base path from the module script location, so subpath deployments need the committed base handling. A blanket Netlify rule like `/* /index.html 200` alone is NOT enough: without a `/src/*` exclusion, the browser resolves the page's relative assets against the deep path and receives the HTML shell instead of CSS/JS. The supported zero-configuration path remains GitHub Pages with the committed 404.html flow.

> **Search-engine indexing is crawler-only.** On plain GitHub Pages, indexing depends on the crawler executing JavaScript: crawlers that do not run JS will not index beyond the root page (see the sitemap and 404 shell note above); JS-capable crawlers follow the 404.html redirect and index the SPA content.

> **Local serving for fork users.** When testing locally, serve the fork with an SPA fallback (e.g. `npx serve -s .`) or your own Pages deployment: a plain static server has no fallback and direct URLs (like `/post/...`) return a raw 404 because those routes are not physical files.

> **HTTP headers vs `<meta>`.** Two security directives are header-only and have no effect inside a `<meta>` tag: `frame-ancestors` (frame embedding control, part of the Content-Security-Policy) and `X-Content-Type-Options: nosniff`. GitHub Pages cannot set custom HTTP headers, so the template relies on the meta CSP for the directives the browser accepts from meta. If you host your fork on infrastructure that supports custom headers, set `Content-Security-Policy` and `X-Frame-Options` / `X-Content-Type-Options` as real HTTP headers to gain the full protection of those directives.

> **iframe embedding.** On GitHub Pages, iframe embedding of the site is NOT blocked (`frame-ancestors` is a header-only directive and Pages cannot set custom headers); hosts that support custom HTTP headers SHOULD set `Content-Security-Policy: frame-ancestors 'none'` (or `X-Frame-Options: DENY`).

## How the site files are generated

Two zero-dependency Node scripts run automatically on every push that touches Markdown or settings files (via the GitHub Action):

- `scripts/generate-index.js`: regenerates `src/content/archive/index.json` and `src/content/portfolio/index.json` by walking the dated folder hierarchy. A folder counts as content when it contains an `index.md` or `index.{lang}.md` file.
- `scripts/generate-site.js`: regenerates the `<head>` of `index.html` (title, meta description, Open Graph, Twitter Cards, JSON-LD), the two static **body fallbacks** (the About heading from `site.name` and the footer copyright line from `site.copyright`), `sitemap.xml`, and `robots.txt` from `src/content/settings.txt`.

You do not need to edit `index.json`, `sitemap.xml`, or `robots.txt` by hand. If you want the files regenerated before pushing, run the scripts locally:

```bash
node scripts/generate-index.js
node scripts/generate-site.js
```

The action commits the regenerated files back to your branch, so your deployed site always matches your content and settings. That auto-commit carries `[skip ci]`, so it does not re-trigger the workflow: a content push runs it once, and the committed artifacts land without spawning a second redundant run.

**The workflow is green by default.** The regenerate workflow runs the generators and commits the results; it performs no placeholder scan and does not gate publication. Substituting placeholders is your responsibility (see the [pre-publication checklist](#pre-publication-checklist)).

## Tech stack

- Vanilla HTML, CSS, and JavaScript: no frameworks, no bundler.
- Markdown with YAML frontmatter for content.
- GitHub Pages for hosting and global CDN delivery.
- GitHub Actions for the content registry and site file automation.
- Zero runtime dependencies; zero supply chain risk.
- No client-side framework means no hydration, no virtual DOM, and no framework upgrade cycle.

> **Performance note.** The archive and portfolio listings fetch one request per post in parallel (`Promise.all`): instant at personal-blog scale (dozens of posts). A large archive (hundreds of posts) would want batching or pre-rendered metadata; deliberately out of scope for the zero-build design.

## License

MIT: see [LICENSE](../LICENSE). The MIT license covers the template code. Content you add to your fork is yours.