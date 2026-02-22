# Theme Blorofin — Development Reference

## Project Overview

A custom micro.blog theme plugin built on Hugo 0.91 (micro.blog's recommended version). Micro.blog themes are Hugo themes with specific conventions for content types, template variables, and deployment.

## Local Development

```bash
mise run serve    # Hugo dev server at localhost:1313
mise run build    # Build to dev-site/public/
mise run clean    # Remove generated files
```

Hugo 0.91.2 is installed at `~/bin/hugo`. The `dev-site/` directory contains a local Hugo site that symlinks back to this theme. Sample content lives in `dev-site/content/`. The dev scaffold is gitignored. Config file is `dev-site/config.json`.

## Hugo Version: 0.91

This theme targets Hugo 0.91, which is micro.blog's recommended version. Key notes:

- `.Site.Author` — the correct way to access author info (`.Site.Author.name`, `.Site.Author.avatar`, etc.)
- `paginate` — top-level config key for pagination count (NOT `pagination.pagerSize`)
- Config file is `config.json` (NOT `hugo.json`)
- Hugo 0.91 does NOT have `--noBuildLock` flag
- Hugo 0.91 does NOT short-circuit `and` in templates — guard nil checks carefully (e.g., `.Page.Params.redirect` crashes when `.Page` is nil)

**Do NOT use Hugo 0.140 patterns** — `.Site.Params.author`, `pagination.pagerSize`, etc. will not work on micro.blog.

## Micro.blog Theme Architecture

### Template Lookup Order (Plugin Mode)

Micro.blog checks templates in this order:
1. Custom theme (if any)
2. **Plugins** (like Blorofin — `theme-blorofin/layouts/`)
3. Selected built-in design (Default, Marfa, etc.)
4. The Blank theme (`microdotblog/theme-blank`)

This theme is installed as a **plugin**, not a custom theme. The "Default" design is selected as the base.

### Important Plugin Notes

- The `master` branch is what micro.blog clones (not `main`) — push to both branches
- `custom.css` is reserved by micro.blog for user customizations via "Edit CSS" — our theme uses `css/blorofin.css`
- Fonts and CSS are loaded in `baseof.html` to bypass any custom theme `head.html` override
- The Default theme's `post/single.html` overrides `_default/single.html` — we add our own `layouts/post/single.html`
- To update plugin: Themes → Blorofin info page → click refresh/re-clone icon → wait for build

### File Structure

```
theme-blorofin/
├── layouts/
│   ├── _default/
│   │   ├── baseof.html              # Base template — loads fonts/CSS, flex body layout
│   │   ├── list.html                # Category/section listing
│   │   ├── list.archivehtml.html    # Archive page
│   │   ├── list.json.json           # JSON Feed for categories
│   │   ├── list.photoshtml.html     # Photos page
│   │   ├── rss.xml                  # RSS feed for categories
│   │   ├── single.html              # Single page (about, etc.)
│   │   └── sitemap.xml              # Sitemap
│   ├── post/
│   │   └── single.html              # Blog post template (overrides Default theme)
│   ├── partials/
│   │   ├── head.html                # <head> content (loads microblog_head.html)
│   │   ├── header.html              # Site header/navigation
│   │   ├── footer.html              # Site footer (sticky via flexbox)
│   │   ├── custom_footer.html       # Hook for plugins to inject content
│   │   ├── microblog_head.html      # Micro.blog-specific <link> tags
│   │   └── microblog_syndication.html  # Bluesky/syndication links
│   ├── redirect/
│   │   └── single.html              # Page redirects
│   ├── index.html                   # Homepage with post filter
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
│   └── robots.txt                   # Robots.txt template
├── static/
│   └── css/
│       └── blorofin.css             # Theme stylesheet (loaded via baseof.html)
├── config.json                      # Hugo/micro.blog configuration
├── mise.toml                        # Dev tasks (Hugo 0.91.2)
├── theme.toml                       # Hugo theme metadata
├── LICENSE                          # License
└── AGENTS.md                        # This file
```

### Key Templates

**baseof.html** — The skeleton every page inherits from. Loads Google Fonts (Inter + Fraunces) and `blorofin.css` directly in `<head>`:
```html
<!DOCTYPE html>
<html>
  <head>
    <!-- Google Fonts: Inter + Fraunces -->
    <link rel="stylesheet" href="/css/blorofin.css?{{ .Site.Params.theme_seconds }}">
    {{ partial "head.html" . }}
  </head>
  <body>
    {{ partial "header.html" . }}
    {{ block "main" . }}{{ end }}
    {{ partial "footer.html" . }}
    {{ partial "custom_footer.html" . }}
  </body>
</html>
```

Body uses `display: flex; flex-direction: column; min-height: 100vh` for sticky footer layout.

**index.html** — Homepage. Filters posts using `where .Site.Pages.ByDate.Reverse "Type" "post"`. Differentiates longform (titled) vs shortform (untitled) posts.

**post/single.html** — Blog post template. Must exist to override Default theme's version. Uses `.Title` to differentiate longform vs shortform: longform gets heading, reading time, drop cap; shortform gets just date + content.

**_default/single.html** — Used for standalone pages (About, etc.).

## Content Types

### Posts (Type: "post")

Blog posts live in `content/YYYY/MM/DD/slug.md`. Two flavors:

- **Titled posts** (longform): Have a `title` in front matter. Displayed with Fraunces heading, reading time, and purple drop cap on first paragraph.
- **Untitled posts** (shortform): No `title` field. Displayed as inline content with date only. No reading time, no drop cap.

Branch on `.Title` in templates:
```html
{{ if .Title }}
  <h1 class="post-title p-name">{{ .Title }}</h1>
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

### Photos

Posts with photos get `.Params.photos` (JPEGs only) and `.Params.images` (all image types) set automatically by micro.blog.

Filter photo posts: `{{ $list := where .Site.Pages ".Params.photos" "!=" nil }}`

### Categories (No Tags)

Micro.blog uses **categories only** (no tags). Access via `.Params.categories` on posts.

## Template Variables

### Site Variables (Hugo 0.91)

| Variable | Description |
|---|---|
| `.Site.Title` | Blog title |
| `.Site.BaseURL` | Base URL |
| `.Site.Author.name` | Author display name |
| `.Site.Author.avatar` | Author avatar URL |
| `.Site.Author.username` | Micro.blog username |
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

Config: `"paginate": 25` in `config.json` (top-level key, Hugo 0.91 style).

Pagination is controlled by site params:
- `paginate_home` — Enable on homepage
- `paginate_categories` — Enable on category pages
- `paginate_replies` — Enable on replies page

## Design Spec

### Typography
- **Body font**: Inter (Google Fonts), system fallback stack
- **Display font**: Fraunces (Google Fonts) for headings, site title (header + footer)
- **Type scale**: Minor third (1.2 ratio), base 17px via `html { font-size: 106.25% }`
- CSS custom properties: `--step-0` through `--step-5`

### Colors
- **Background**: `#f9f9f7` off-white
- **Text**: `#15171a` near-black
- **Accent**: `#a347bf` purple — used on anchor tags, nav link hover, blockquote left border, drop cap, "Read more →"
- **Border**: `rgba(0, 0, 0, 0.08)`

### Layout
- Body: `display: flex; flex-direction: column; min-height: 100vh` (sticky footer)
- Header: flexbox, max-width 1200px
- Content (`.site-main`): `flex: 1; width: 100%; max-width: 920px; margin: 0 auto`
- Footer: `margin-top: auto`, top border only, Fraunces site name

### CSS Notes
- Stylesheet is `static/css/blorofin.css` (NOT `custom.css` — reserved by micro.blog)
- Cache-busted via `?{{ .Site.Params.theme_seconds }}` query param
- Use flex properties for layout (not float/grid)

## config.json Reference

The theme's `config.json` is merged over micro.blog's defaults. Allowed top-level keys:

`params`, `title`, `description`, `defaultContentLanguage`, `disableKinds`, `enableEmoji`, `hasCJKLanguage`, `languageCode`, `languageName`, `paginate`, `related`, `rssLimit`, `summaryLength`, `enableRobotsTXT`, `markup`, `enableInlineShortCodes`, `mediaTypes`, `outputFormats`, `outputs`

Custom params go under `params` and are accessed via `.Site.Params.your_key`.

## CSS

The theme stylesheet is at `static/css/blorofin.css` and is loaded directly in `baseof.html`:
```html
<link rel="stylesheet" href="/css/blorofin.css?{{ .Site.Params.theme_seconds }}">
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

This theme is deployed as a **plugin** (not a custom theme):

1. Push changes to GitHub (both `main` and `master` branches)
2. In micro.blog: Themes → Blorofin info page → click refresh/re-clone icon
3. Wait ~30 seconds for clone + rebuild
4. Check build logs for "Clone: Done" + "Publish: Done 🎉"

### Micro.blog IDs
- Blog site_id: 298568
- Blorofin plugin theme_id: 95586
- Admin URLs:
  - Design: `https://micro.blog/account/design/298568`
  - Themes: `https://micro.blog/account/themes`
  - Plugin info: `https://micro.blog/account/themes/95586/info`
  - Build logs: `https://micro.blog/account/logs`

## Reference Links

- [Custom Themes docs](https://help.micro.blog/t/custom-themes/59)
- [Parameters in Themes](https://help.micro.blog/t/parameters-in-themes/61)
- [Custom Home Page](https://help.micro.blog/t/custom-home-page/62)
- [Plug-ins docs](https://help.micro.blog/t/plug-ins/104)
- [Photo Pages](https://help.micro.blog/t/photo-pages/80)
- [Hosted Replies](https://help.micro.blog/t/hosted-replies/66)
- [Categories](https://help.micro.blog/t/using-categories/68)
- [Blank theme repo](https://github.com/microdotblog/theme-blank)
- [Default theme repo](https://github.com/microdotblog/theme-default) — The base design currently selected
- [Hugo 0.91 release](https://github.com/gohugoio/hugo/releases/tag/v0.91.0)
- [Hugo template docs](https://gohugo.io/templates/)
