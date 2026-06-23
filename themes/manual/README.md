# manual

> Sans-serif documentation / handbook theme for AnyCMS-SSG.

Functional, utilitarian, content-density-oriented: a sticky top bar with
built-in search (`/search.json`), a section-document index, and an on-page
table of contents. Light/dark. Sibling to `ink` (which is serif / blog /
typography-first); `manual` is sans-serif / docs / density-first.

## Install

```sh
anycms theme install ./themes/manual --root ./my-site
```

Then make sure `my-site/site.kdl` references it and that the site's own
templates don't shadow the ones you want from the theme:

```kdl
theme "manual"
```

## Tunable params (in `site.kdl` → `params { … }`)

| param | default | values | effect |
|---|---|---|---|
| `accent` | `#2563eb` | any CSS color | links, focus rings, TOC hover |
| `dark` | `auto` | `auto` / `on` / `off` | color scheme (`auto` follows the OS) |
| `font_scale` | `1.0` | number | multiplies the base font size |
| `width` | `normal` | `normal` / `wide` | content column width |

```kdl
params {
    accent "#0891b2"
    dark "on"
    width "wide"
}
```

## Docs-specific features

- **Top-bar search** — `base.html` renders a `#search` input + `#search-results`
  list; a small dependency-free inline script fetches `/search.json` once (the
  engine generates it automatically) and filters by title/body substring.
- **On-page TOC** — `page.html` scans the article for `h2`/`h3`, assigns ids
  where missing, and populates a right-rail `<nav class="toc">`. The nav is
  hidden when fewer than two headings exist.
- **Section index** — `section.html` lists `section.pages` as the document hub.

## Layouts

`base` · `index` · `section` · `page` · `taxonomy` · `taxonomy_term`
