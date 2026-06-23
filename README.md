# Liangdi's Blog

Liangdi 的个人博客,关注开源技术 / Rust / 云原生。

- 引擎:[**anycms-ssg**](https://anycms.org) — Rust 静态站点生成器(文档 / 下载 / 安装见官网)
- 主题:**dream** — [g1eny0ung/hugo-theme-dream](https://github.com/g1eny0ung/hugo-theme-dream) v3 的 anycms 移植(daisyUI + Alpine.js)
- 语言:中文(`default_language "zh"`)
- 线上:<https://liangdi.me>

## 前置:安装 anycms CLI

博客用 `anycms` 命令构建。安装方式(预编译二进制 / 源码)见官网:<https://anycms.org>。

```sh
anycms --version   # 确认可用
```

## 主题

`themes/` 已内嵌四个主题的**实体副本**(从 anycms-ssg 复制而来,非 symlink),本仓库完全自包含:

- `themes/dream` — `site.kdl` 实际使用(g1eny0ung/hugo-theme-dream v3 移植)
- `themes/ink` / `themes/folio` / `themes/manual` — 另外三个官方主题,供 devtools 面板切换预览

```sh
themes/
├── dream/    ← site.kdl theme "dream"
├── ink/
├── folio/
└── manual/
```

副本不会随 anycms-ssg 上游自动更新。要同步上游改动(见 <https://anycms.org>),从 anycms-ssg 源码树重新复制——把 `ANYCMS` 换成你本地的源码路径:

```sh
ANYCMS=<你的 anycms-ssg 源码路径>
for t in dream ink folio manual; do
  rm -rf "themes/$t" && cp -r "$ANYCMS/themes/$t" "themes/$t"
done
```

## 构建与预览

```sh
# 一次性构建到 public/(静态产物,可直接部署)
anycms build

# 开发服务器(默认 1111 端口;文件改动热重建 + 浏览器自动刷新)
anycms serve
anycms serve --port 8080         # 指定端口
anycms serve --host 0.0.0.0      # 局域网预览
```

**DevTools 面板**:serve 时打开站点任意页面,右下角浮动按钮即面板 ——
主题切换、参数调整、草稿预览、viewport 尺寸(mobile/tablet/desktop)、一键重编译。

## 内容结构

```
content/
├── _index.md        首页(标题 + 描述;文章列表由引擎跨 section 聚合)
├── about.md         /about/
├── search.md        /search/(搜索页,template = "search.html")
└── p/               文章 section → /p/<slug>/
    ├── 2023-rust-web-frameworks.md
    ├── rust-hacking-programing-syn-flood-ddos.md
    └── …(共 12 篇)
```

文章 front matter(YAML):

```yaml
---
title: "文章标题"
date: 2023-01-28T01:33:57+08:00
draft: false
slug: "url-slug"           # → /p/url-slug/
tags: ["rust", "web"]       # → /tags/rust/、/tags/web/
---
```

> 中文 tag 完全支持(slug 保留 UTF-8,如 `/tags/云原生/`)。

## 配置速览(site.kdl)

| 项 | 值 | 说明 |
|---|---|---|
| `theme` | `"dream"` | 主题 |
| `base_url` | `https://liangdi.me` | 用于 sitemap/rss/绝对链接 |
| `default_language` | `"zh"` | |
| `taxonomies` | `tags`, `categories` | |
| `params {}` | zenMode / 配色 / 作者 / 格言 / 导航顺序 / 社交链接 | 覆盖 dream 默认 |
| `image.widths` | `640,1024,1920` | 响应式变体宽度(cover + `{{< image >}}` shortcode) |
| `image.quality` | `80` | JPEG/WebP 质量 |

## 输出与缓存

- `public/` — 构建产物(静态文件,部署到任意静态托管:Cloudflare Pages / Vercel / GitHub Pages / Nginx)
- `.anycms/img-cache/` — 图片变体缓存(gitignored;改 `image` 设置后可清理)
- `.anycms/dev-overrides.json` — devtools 运行时状态(主题/参数/草稿覆盖,gitignored)

```sh
anycms clean            # 清图片缓存(下次构建重新生成所有变体)
anycms clean --output   # 连 public/ 一起清
```

## 部署示例(Cloudflare Pages)

- 构建命令:`anycms build`(确保 `anycms` 在 PATH;安装见 <https://anycms.org>)
- 输出目录:`public`
- 环境变量:无需

---

源文章在 `content/p/`,Markdown 即所见。改完 `anycms serve` 本地预览,确认无误后 `anycms build` 出 `public/` 部署。
