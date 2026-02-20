# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio and blog for Austin Ellingwood, built with [Forge](https://github.com/ellingwood/forge) — a custom Go-based static site generator.

## Commands

```sh
forge serve          # Dev server with live reload at http://localhost:1313
forge build          # Production build to ./public/
```

There are no test, lint, or install commands. Forge is a standalone binary.

## Architecture

**Static site generator:** Forge uses Go HTML templates with a content/theme separation pattern similar to Hugo. Configuration lives in `forge.yaml`.

**Content** (`content/`): Markdown files with YAML front matter. Directory structure maps to URL routes. `_index.md` files create section list pages. Supports `draft: true` in front matter.

**Theme** (`themes/default/`): Contains all layouts, styles, and JS. This is where the bulk of the presentational code lives.

- `layouts/_default/baseof.html` — Base template. Pages fill `{{ block "main" . }}`.
- `layouts/_default/single.html` / `list.html` — Default content and list templates.
- `layouts/index.html` — Homepage (features avatar, projects, blog posts).
- `layouts/projects/` — Project-specific templates override the defaults.
- `layouts/partials/` — Reusable components (header, footer, cards, pagination, etc.).
- `static/css/globals.css` — CSS custom properties for theming (HSL color system).
- `static/js/theme.js` — Dark mode toggle (persisted in localStorage).
- `static/js/nav.js` — Mobile navigation.
- `tailwind.config.js` — Tailwind v3 with `@tailwindcss/typography`. Dark mode via `class` strategy.

**Layout overrides** (`layouts/` at root): Empty by default. Files here override same-path files in the theme.

**Data** (`data/`): YAML data files accessible in templates. Currently holds `skills.yaml`.

**Static assets** (`static/`): Copied directly to `public/`. Favicons and images live here.

## Styling

Tailwind CSS with HSL-based CSS custom properties (shadcn/ui pattern). Colors defined in `globals.css`, consumed via `tailwind.config.js` theme extension. Fonts: Inter (sans), JetBrains Mono (mono).

## Template Syntax

Forge templates use Go `html/template` syntax: `{{ .Title }}`, `{{ range }}`, `{{ partial "name.html" . }}`, `{{ block "name" . }}`. Site-level data accessed via `.Site`, page data via `.Page` or top-level dot context.

## MCP Integration

This project uses the Forge static site generator with an MCP server (`forge mcp`).

Use `create_content` to create new blog posts, pages, and projects — it writes
Markdown files with valid frontmatter. Use `build_site` to render the site.
Use `query_content` and `get_page` to explore existing content.

## Article Voice

When writing articles, include a relevant hero image if possible, something the is generally technology related/relevant to the topic. If including media in the post, be sure to use the proper nested format for posts which is `blog/<example-post>/example-post.md` and `blog/<example-post>/example-image.png`. Blog posts should be informative but light unless specified. Include a hook at the top of the article and a quick "about the author" blurb below that with a call to contact me at my email with a link.
