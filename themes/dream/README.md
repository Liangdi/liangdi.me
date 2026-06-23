# dream

A port of [g1eny0ung/hugo-theme-dream](https://github.com/g1eny0ung/hugo-theme-dream) v3
to **anycms-ssg**. Faithful visual reproduction: daisyUI cards, the flip card,
Alpine.js dark-mode store, the masonry grid (non-zen) and single-column reading
layout (zen).

> The original theme is Go-template + Tailwind/daisyUI. This port rewrites the
> 31 Go templates in MiniJinja and ships dream's **precompiled** Tailwind output
> (`output.css`) and JS (`main.js`, `grid.js`) verbatim as static assets — the
> runtime look is identical, only the template language differs.

## Layout

```
themes/dream/
├── theme.kdl              manifest + `params {}` defaults (overrides via site.kdl)
├── static/
│   ├── css/output.css      ← dream's precompiled daisyUI/Tailwind stylesheet
│   └── js/{main,grid}.js   ← dark-mode store + masonry init (verbatim)
└── templates/
    ├── base.html           layout shell (Alpine x-data, flip container, nav, footer)
    ├── index.html          homepage (zen / grid over section.pages)
    ├── section.html        section listing
    ├── page.html           single post (TOC, prev/next, share)
    ├── archive.html        /archive/ timeline (anycms `archive.years`)
    ├── taxonomy.html       taxonomy index (tag cloud)
    ├── taxonomy_term.html  single term (term.pages list)
    ├── search.html         client-side fuse.js over /search.json
    ├── 404.html
    ├── _summary.html       grid card (loop var `p`)
    ├── _zen-summary.html   zen card (loop var `p`)
    ├── macros/nav.html     nav-icon macro
    └── partials/           head, nav, scripts, paginator, footer*, author,
                            share, socials, luxon, back, zen-back, comment*
```

## Params (set in `site.kdl` `params {}`, all string-valued)

| Param | Default | Meaning |
|---|---|---|
| `zenMode` | `"true"` | single-column reading layout vs masonry grid |
| `lightTheme` / `darkTheme` | `emerald` / `forest` | daisyUI theme names |
| `author`, `headerTitle`, `motto`, `avatar` | — | identity |
| `footerBottomText`, `siteStartYear` | — | footer |
| `rss` | `"true"` | show RSS nav icon |
| `stickyNav` | `"false"` | sticky header on scroll |
| `noDefaultSummaryCover` | `"false"` | suppress default cover image |
| `showSummaryCoverInPost` | `"false"` | show cover on single post |
| `showPrevNextPost` | `"true"` | prev/next nav |
| `showTableOfContents` | `"true"` | TOC sidebar |
| `reorderNavItems` | `"search,rss,posts,categories,tags"` | nav icon order |
| `backgroundImage` / `backgroundImageDark` | — | global bg |
| `jsDate` / `jsDateFormat` | `"true"` / `"yyyy-MM-dd"` | luxon client formatting |
| `social` | — | KDL child block `github "..."` → social icons |
| `email`, `favicon` | — | |

## Cover images

anycms has **no per-page front-matter image pipeline** (the `{{< image >}}`
shortcode only processes images inside markdown bodies). So covers are shipped
as **static assets**: put each cover at `static/covers/<slug>.png` and reference
it as a top-level front-matter key (it flattens into `page.extra.cover`):

```toml
+++
title = "..."
cover = "/covers/my-post.png"
+++
```

Optionally pre-generate `<slug>.webp` for smaller payloads.

## Permalinks

To reproduce Hugo's `post: /p/:slug/`, place posts at `content/p/<slug>.md` —
anycms's default `page_url` (`:section/:slug/`) then yields `/p/<slug>/` with
zero permalink config. Top-level pages keep `/<slug>/`.

## Known fidelity gaps (vs original dream)

1. **Reading time** — not computed by anycms; the reading-time line is omitted.
2. **Front-matter cover image pipeline** — see above (static URLs, no WebP
   auto-conversion).
3. **Taxonomy-term pagination** — `term.pages` is the full list; rendered
   without page nav.
4. **Auto-mermaid** — anycms has no `.Store`; mermaid is loaded unconditionally
   (harmless when no diagrams).
5. **i18n `T` / `emojify`** — strings hardcoded to Chinese; `emojify` dropped.

## Dependencies (all CDN, per the keep-Alpine decision)

Alpine.js 3.x, ionicons 7.4, luxon 1.26, mermaid 11.1, and (non-zen only)
imagesloaded + masonry 4.2. `fuse.js` 7.0 powers `/search/`.
