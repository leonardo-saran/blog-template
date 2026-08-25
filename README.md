# blog-template

A minimal, bilingual-capable, zero-dependency static blog template. Fork it, personalize it, deploy it to GitHub Pages: no build step, no frameworks, no database.

> **[View live demo](https://leonardo-saran.github.io/blog-template/)** to see the template running with its sample content.

## Features

- **Pure static**: vanilla HTML, CSS, and JavaScript. No build step, no bundler, no runtime dependencies.
- **Bilingual**: English plus a second language driven by a single settings file, with a one-click toggle.
- **Markdown content**: posts, projects, and the About page are plain `.md` files with YAML frontmatter.
- **LaTeX-friendly**: math via self-hosted MathJax 3 (`src/assets/vendor/mathjax/`), loaded lazily only on pages with math.
- **Double-way visibility**: content appears only in the languages it exists in (no fallback).
- **Adaptive theme & language**: first-visit defaults follow the system and browser, with explicit toggles persisted locally.
- **SPA navigation**: History API routing with real, shareable paths (`/about`, `/archive`, `/portfolio`, `/post/{slug}`, `/tag/{tag}`).
- **Built-in search and sort** on the archive and portfolio pages.
- **SEO ready**: Open Graph, Twitter Cards, JSON-LD, `sitemap.xml`, and `robots.txt` generated from your settings.
- **GitHub Actions automation**: regenerates the content index and site files on every relevant push.

## Quick start

1. **Fork** this repository.
2. **Personalize** `src/content/settings.txt`: replace every placeholder with your own values. The template ships placeholders by design, and there is **no automated placeholder check**: substitution is the owner's responsibility before publishing (see [customization](docs/customization.md)).
3. **Add content** under `src/content/` (see [Adding content](#adding-content)). A single demo post ships in `src/content/archive/`; the portfolio section starts empty.
4. **Deploy**: push to `main` and enable GitHub Pages on the repository (Settings → Pages → deploy from branch `main`, folder `/ (root)`).
5. **Visit** your site at the Pages URL (or your custom domain).

## Adding content

See the [Content Authoring Guide](src/content/README.md) for the full authoring reference.

Posts and projects live under dated folders in `src/content/archive/YYYY/MM/DD/your-slug/` and `src/content/portfolio/YYYY/MM/DD/your-slug/`; the About page lives at `src/content/about/`. Copy a starter from `src/content/templates/`, edit it, commit, and push: the GitHub Action updates the content index for you.

## Documentation

- [Customization guide](docs/customization.md): adaptive defaults, `settings.txt` reference, custom domains, localization, deployment, generation scripts, and security/SEO notes.

## Tech stack

| Layer | Choice |
|-------|--------|
| Frontend | Vanilla HTML, CSS, JavaScript |
| Content | Markdown with YAML frontmatter |
| Hosting | GitHub Pages (global CDN, automatic HTTPS) |
| Automation | GitHub Actions |
| Dependencies | Zero |

## License

[MIT](LICENSE). The license covers the template code; content you add to your fork is yours.