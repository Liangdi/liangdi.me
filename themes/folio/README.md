# folio

> Portfolio / visual-work theme for AnyCMS-SSG — gallery grid, bold display
> type, and generous negative space.

Image-forward and asymmetric where `ink` is typographic and `manual` is
documentarian. `folio` is a display piece: confident headings, a responsive
project gallery, and a quiet high-contrast palette in light/dark. Built on the
same theme↔plugin seam (`seo_head` / `head_inject` / `body_inject`) and
`params`-driven design tokens as the reference theme.

## Install

```sh
anycms theme install ./themes/folio --root ./my-site
```

Then make sure `my-site/site.kdl` references it and that the site's own
templates don't shadow the ones you want from the theme:

```kdl
theme "folio"
```

## Tunable params (in `site.kdl` → `params { … }`)

| param | default | values | effect |
|---|---|---|---|
| `accent` | `#111111` | any CSS color | links, rules, hover, focus rings |
| `dark` | `auto` | `auto` / `on` / `off` | color scheme (`auto` follows the OS) |
| `font_scale` | `1.0` | number | multiplies the base font size |
| `width` | `normal` | `normal` / `wide` | content column width |
| `columns` | `3` | `2` / `3` / `4` | project gallery grid columns |

```kdl
params {
    accent "#b91c1c"
    dark "on"
    width "wide"
    columns "4"
}
```

## Layouts

`base` · `index` · `section` · `page` · `taxonomy` · `taxonomy_term`

- **index** — landing: big site title, optional section lede, recent-work grid.
- **section** — gallery grid of project cards (typographic: title + accent rule + date).
- **page** — featured project: oversized title, date, wide readable body column.
- **taxonomy** / **taxonomy_term** — term index and per-term project grid.
