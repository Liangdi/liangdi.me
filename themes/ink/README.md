# ink

> Minimal, typography-first blog theme — the **reference default** for AnyCMS-SSG.

Single column, serif body, ink-on-paper. Built to showcase the theme system
end-to-end: overlay resolution, `params`-driven design tokens, the
theme↔plugin seam (`seo_head` / `head_inject` / `body_inject`), taxonomies,
feeds, sitemap, and code highlighting.

## Install

```sh
anycms theme install ./themes/ink --root ./my-site
```

Then make sure `my-site/site.kdl` references it and that the site's own
templates don't shadow the ones you want from the theme:

```kdl
theme "ink"
```

## Tunable params (in `site.kdl` → `params { … }`)

| param | default | values | effect |
|---|---|---|---|
| `accent` | `#3a6ea5` | any CSS color | links, rules, focus rings |
| `dark` | `auto` | `auto` / `on` / `off` | color scheme (`auto` follows the OS) |
| `font_scale` | `1.0` | number | multiplies the base font size |
| `width` | `normal` | `normal` / `wide` | content column width |

```kdl
params {
    accent "#b91c1c"
    dark "on"
    width "wide"
}
```

## Layouts

`base` · `index` · `section` · `page` · `taxonomy` · `taxonomy_term`
