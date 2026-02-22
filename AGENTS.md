# Theme Blorofin — Development Reference

## Project Overview

A custom micro.blog theme built on Hugo 0.140. Micro.blog themes are Hugo themes with specific conventions for content types, template variables, and deployment.

## Local Development

```bash
mise run serve    # Hugo dev server at localhost:1313
mise run build    # Build to dev-site/public/
mise run clean    # Remove generated files
```

Hugo 0.140 is installed at `~/bin/hugo`. The `dev-site/` directory contains a local Hugo site that symlinks back to this theme. Sample content lives in `dev-site/content/`. The dev scaffold is gitignored.

## Hugo Version: 0.140

This theme targets Hugo 0.140. Key differences from older versions used by other micro.blog themes:

- `.Site.Author` is removed — use `.Site.Params.author` instead
- `paginate` config key is removed — use `pagination.pagerSize`
- `getJSON`/`getCSV` are removed — use `resources.GetRemote`
- `.Page.NextPage`/`.PrevPage` are removed — use `.Page.Next`/`.Page.Prev`

The official micro.blog themes (Marfa, etc.) still use `.Site.Author`. This theme has already been migrated to `.Site.Params.author`.

## Micro.blog Theme Architecture

### Template Lookup Order

Micro.blog checks templates in this order:
1. Your custom theme (`theme-blorofin/layouts/`)
2. The selected built-in design (Marfa, Sumo, etc.)
3. The Blank theme (`microdotblog/theme-blank`)

You only need to provide templates you want to customize. Everything else falls through to the Blank theme.

### Required File Structure

```
theme-blorofin/
├── layouts/
│   ├── _default/
│   │   ├── baseof.html              # Base template — all pages inherit from this
│   │   ├── list.html                # Category/section listing
│   │   ├── list.archivehtml.html    # Archive page (override)
│   │   ├── list.json.json           # JSON Feed for categories
│   │   ├── list.photoshtml.html     # Photos page (override)
│   │   ├── rss.xml                  # RSS feed for categories
│   │   ├── single.html              # Single page (about, etc.)
│   │   └── sitemap.xml              # Sitemap
│   ├── post/
│   │   └── single.html              # Single blog post (optional, falls back to _default/single.html)
│   ├── reply/
│   │   └── single.html              # Reply post (optional)
│   ├── section/
│   │   └── replies.html             # Replies collection (optional)
│   ├── partials/
│   │   ├── head.html                # <head> content (loads microblog_head.html)
│   │   ├── header.html              # Site header/navigation
│   │   ├── footer.html              # Site footer
│   │   ├── custom_footer.html       # Hook for plugins to inject content
│   │   ├── microblog_head.html      # Micro.blog-specific <link> tags (feeds, rel=me, etc.)
│   │   └── microblog_syndication.html  # Bluesky/syndication links
│   ├── index.html                   # Homepage
│   ├── index.json                   # JSON Feed (homepage)
│   ├── index.xml                    # RSS Feed (homepage)
│   ├── list.archivehtml.html        # Archive page
│   ├── list.archivejson.json        # Archive JSON feed
│   ├── list.photoshtml.html         # Photos page
│   ├── list.photosjson.json         # Photos JSON feed
│   ├── list.podcastjson.json        # Podcast JSON feed
│   ├── list.podcastxml.xml          # Podcast RSS feed
│   ├── list.rsd.xml                 # RSD (Really Simple Discovery)
│   ├── newsletter.html              # Email newsletter template
│   ├── redirect/
│   │   └── single.html              # Page redirects
│   └── robots.txt                   # Robots.txt template
├── static/
│   └── custom.css                   # Theme stylesheet (referenced by microblog_head.html)
├── config.json                      # Hugo/micro.blog configuration
├── mise.toml                        # Dev tasks
└── AGENTS.md                        # This file
```

### Key Templates

**baseof.html** — The skeleton every page inherits from:
```html
<!DOCTYPE html>
<html>
  {{ partial "head.html" . }}
  <body>
    {{ partial "header.html" . }}
    {{ block "main" . }}{{ end }}
    {{ partial "footer.html" . }}
    {{ partial "custom_footer.html" . }}
  </body>
</html>
```

All page templates define `{{ define "main" }}...{{ end }}` to inject content into the block.

**index.html** — Homepage. Lists posts using `.Site.RegularPages` or filtered by type/category.

**_default/single.html** — Used for standalone pages (About, etc.) and as fallback for posts if `post/single.html` doesn't exist.

**post/single.html** — (Optional) Dedicated post template. Micro.blog stores blog posts with `Type: "post"`.

## Content Types

### Posts (Type: "post")

Blog posts live in `content/YYYY/MM/DD/slug.md`. Two flavors:

- **Titled posts** (longform): Have a `title` in front matter. Displayed with heading + full content.
- **Untitled posts** (microblog): No `title` field. Displayed as inline content with timestamp.

Branch on `.Title` in templates:
```html
{{ if .Title }}
  <h2><a href="{{ .Permalink }}">{{ .Title }}</a></h2>
{{ end }}
```

### Pages

Standalone pages (About, etc.) created via Posts → Pages in micro.blog. Rendered with `_default/single.html`.

### Replies (Type: "reply")

Stored separately from posts. Available front matter params:
- `.Params.reply_to_url` — URL being replied to
- `.Params.reply_to_username` — Username of reply target
- `.Params.reply_to_avatar` — Avatar of reply target
- `.Params.reply_to_hostname` — Hostname (e.g., "micro.blog")

Templates: `reply/single.html` and `section/replies.html`.

### Photos

Posts with photos get `.Params.photos` (JPEGs only) and `.Params.images` (all image types) set automatically by micro.blog.

Filter photo posts: `{{ $list := where .Site.Pages ".Params.photos" "!=" nil }}`

### Podcasts

Posts with audio get `.Params.audio` and `.Params.audio_with_metadata` (includes `.url`, `.size`, `.duration_seconds`).

## Template Variables

### Site Variables (Hugo 0.140)

| Variable | Description |
|---|---|
| `.Site.Title` | Blog title |
| `.Site.BaseURL` | Base URL |
| `.Site.Params.author.name` | Author display name |
| `.Site.Params.author.avatar` | Author avatar URL |
| `.Site.Params.author.username` | Micro.blog username |
| `.Site.Params.description` | Site description/tagline (supports HTML) |
| `.Site.RegularPages` | All content pages (no sections/taxonomies) |
| `.Site.Pages` | All pages including sections |
| `.Site.Menus.main` | Navigation menu items |
| `.Site.Taxonomies.categories` | All categories |
| `.Site.LanguageCode` | Language code |

### Page Variables

| Variable | Description |
|---|---|
| `.Title` | Page/post title (empty for microblog posts) |
| `.Content` | Rendered HTML content |
| `.Summary` | Auto-generated summary (first 70 words) or `<!--more-->` split |
| `.Truncated` | True if content was truncated for summary |
| `.RawContent` | Raw Markdown source |
| `.Date` | Publication date |
| `.Permalink` | Full canonical URL |
| `.RelPermalink` | Relative URL |
| `.Type` | Content type: "post", "reply", "page" |
| `.Kind` | Hugo kind: "home", "page", "section", "taxonomy", "term" |
| `.IsHome` | True on homepage |
| `.Params` | All front matter params |
| `.Params.categories` | Post categories |
| `.Params.photos` | JPEG photo URLs |
| `.Params.images` | All image URLs |
| `.Params.audio` | Audio file URL |
| `.Params.guid` | Custom GUID (if set) |

### Pagination

```html
{{ $paginator := .Paginate (where .Site.RegularPages "Type" "post") }}
{{ range $paginator.Pages }}
  ...
{{ end }}
```

Pagination is controlled by site params:
- `paginate_home` — Enable on homepage
- `paginate_categories` — Enable on category pages
- `paginate_replies` — Enable on replies page

## config.json Reference

The theme's `config.json` is merged over micro.blog's defaults. Allowed top-level keys:

`params`, `title`, `description`, `defaultContentLanguage`, `disableKinds`, `enableEmoji`, `hasCJKLanguage`, `languageCode`, `languageName`, `pagination.pagerSize`, `related`, `rssLimit`, `summaryLength`, `enableRobotsTXT`, `markup`, `enableInlineShortCodes`, `mediaTypes`, `outputFormats`, `outputs`

Custom params go under `params` and are accessed via `.Site.Params.your_key`.

## CSS

The theme stylesheet is at `static/custom.css` and is loaded by `microblog_head.html`:
```html
<link rel="stylesheet" href="{{ "custom.css" | relURL }}?{{ .Site.Params.theme_seconds }}">
```

The `theme_seconds` query param is a cache-busting timestamp set by micro.blog on each publish.

Plugins can inject additional CSS via `plugins_css` param.

## IndieWeb / Microformats

Micro.blog themes use microformat classes for IndieWeb compatibility. Use these consistently:

| Class | Element | Purpose |
|---|---|---|
| `h-feed` | Feed container | Marks a feed of entries |
| `h-entry` | `<article>` | Individual post |
| `h-card` | Author block | Author identity |
| `p-name` | Title | Post or person name |
| `p-author` | Author ref | Post author |
| `e-content` | Content div | Post body |
| `dt-published` | `<time>` | Publication date |
| `u-url` | `<a>` | Canonical permalink |
| `u-photo` | `<img>` | Photo or avatar |
| `u-in-reply-to` | `<a>` | Reply target URL |
| `p-summary` | Summary text | Post excerpt |

## Plugin System

Micro.blog plugins can inject resources into themes via three params:

- `plugins_css` — Array of CSS URLs, rendered as `<link>` tags in `<head>`
- `plugins_js` — Array of JS URLs, rendered as `<script>` tags in footer
- `plugins_html` — Array of partial template names, rendered in `<head>`

The `custom_footer.html` partial is a hook for plugin content injection.

## Deployment

1. Push theme changes to GitHub
2. In micro.blog: Posts → Design → Edit Custom Themes
3. Create/update theme by cloning the GitHub repo URL
4. Select the theme under Posts → Design
5. Set Hugo version to 0.140 in micro.blog settings

Micro.blog will skip binary files (images, fonts) when importing. Upload those via Posts → Uploads.

## Reference Links

- [Custom Themes docs](https://help.micro.blog/t/custom-themes/59)
- [Parameters in Themes](https://help.micro.blog/t/parameters-in-themes/61)
- [Custom Home Page](https://help.micro.blog/t/custom-home-page/62)
- [Plug-ins docs](https://help.micro.blog/t/plug-ins/104)
- [Photo Pages](https://help.micro.blog/t/photo-pages/80)
- [Hosted Replies](https://help.micro.blog/t/hosted-replies/66)
- [Categories](https://help.micro.blog/t/using-categories/68)
- [Blank theme repo](https://github.com/microdotblog/theme-blank)
- [Marfa theme repo](https://github.com/microdotblog/theme-marfa) — Good reference for a full-featured theme
- [Hugo 0.140 release](https://github.com/gohugoio/hugo/releases/tag/v0.140.0)
- [Hugo template docs](https://gohugo.io/templates/)
