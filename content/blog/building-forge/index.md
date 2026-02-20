---
title: "Building Forge: A Static Site Generator Designed for AI Workflows"
date: 2026-02-18
draft: false
description: "How I built a Hugo-inspired static site generator in Go with a built-in MCP server, and why the AI-native development loop changes how you think about tooling."
tags:
  - go
  - web
  - mcp
  - open-source
  - claude-code
project: forge
cover:
  image: "cover.png"
  alt: "Forge Static Site Generator"
series: "Building Enterprise at Home"
---

This is the second post in the **Building Enterprise at Home** series. Where [the first post](/blog/building-localcloud/) covered building a local cloud emulator, this one is about the tool generating the page you're reading right now.

## Why Build a Static Site Generator

I've used Hugo for years and genuinely liked it. It's fast, the template system is powerful, and the single-binary distribution model is exactly right. But I kept running into the same friction: Hugo's complexity outpaced what I needed, the template function library has its own learning curve, and extending it to work with AI tooling meant bolting things on from the outside.

I wanted something with Hugo's core strengths — Go, speed, Markdown, single binary — but stripped down to what I actually use, with first-class support for AI content workflows built in from day one. Not as an afterthought, not as a plugin, but as a core design constraint.

That constraint is the MCP server. More on that below.

The other motivation is the same one behind everything in this series: getting better at building with AI. Every project is an exercise in prompting, workflow design, and understanding where the boundaries are between what I should specify and what I should delegate. A static site generator is a good candidate because the domain is well-understood, the output is deterministic, and the feedback loop is immediate — you can see whether it works by opening a browser.

## What is Forge

Forge is a static site generator written in Go. Markdown content with YAML frontmatter goes in, a complete static site comes out. Single binary, no Node.js, no webpack, no runtime dependencies.

The feature set:

- **Parallel content processing** — builds target sub-second for sites under 500 pages
- **Tailwind CSS** — standalone CLI integration, automatically downloaded and managed
- **Syntax highlighting** — Chroma with 200+ languages, light and dark theme support
- **Live reload** — WebSocket-based, rebuilds and refreshes the browser on file save
- **Taxonomy system** — tags, categories, custom taxonomies with auto-generated term pages
- **RSS and Atom feeds** — generated from configurable content sections
- **Client-side search** — Fuse.js with a pre-built JSON index
- **SEO output** — sitemap, robots.txt, OpenGraph, Twitter Cards, JSON-LD, web manifest
- **S3 + CloudFront deploy** — content-hash diffing with cache header management
- **MCP server** — full Model Context Protocol integration for AI-assisted content workflows

The theme system follows Hugo's conventions. Content lives in `content/`, themes provide layouts and static assets in `themes/`, and root-level `layouts/` can override any theme template. Configuration is a single `forge.yaml`.

## The Development Process

Like LocalCloud, Forge was built entirely with [Claude Code](https://docs.anthropic.com/en/docs/build-with-claude/claude-code/overview). The workflow at this point is well-practiced: start with a spec, build iteratively, review carefully, refine.

The build order was deliberate:

1. **Configuration and content layer** — YAML/TOML config loading via Viper, content discovery, frontmatter parsing, and Markdown rendering with Goldmark. This is the foundation everything else depends on.
2. **Template engine** — Go's `html/template` with a custom function library, theme resolution, partial support, and block inheritance. Getting the template context right — what data is available where — took the most iteration.
3. **Build pipeline** — parallel page rendering, static file copying, Tailwind CSS compilation, and output writing. The pipeline is orchestrated but each stage is straightforward.
4. **Dev server** — HTTP file server with WebSocket live reload and a file watcher that triggers rebuilds on content or layout changes.
5. **Ancillary generators** — sitemap, robots.txt, RSS/Atom feeds, search index, web manifest. Each is a self-contained function that takes structured input and produces output bytes.
6. **MCP server** — the integration point for AI workflows, built on top of the existing internal packages.
7. **Deploy** — S3 sync with content-hash diffing and CloudFront invalidation.

The architecture benefited from keeping packages small and focused. `internal/content` handles discovery and parsing, `internal/template` handles rendering, `internal/build` coordinates the pipeline, `internal/seo` generates ancillary files. The MCP server in `internal/mcpserver` is a thin layer that calls into these same packages — it doesn't duplicate any logic.

One thing I noticed building this compared to LocalCloud: the AI-assisted workflow gets more efficient with practice. Not because the tool improves between projects, but because you get better at specifying what you want. You learn which details to nail down upfront (data structures, interfaces, file paths) and which to leave flexible (formatting, error messages, variable names). The spec for Forge was tighter than the one for LocalCloud, and the result was fewer iterations to get things right.

## The MCP Server

This is the part that makes Forge more than just "another SSG." The MCP server exposes the site's content graph, build system, and configuration to AI editors over the [Model Context Protocol](https://modelcontextprotocol.io).

In practice, this means Claude Code can:

- **Query content** — search and filter pages by section, tags, date range, draft status, and full text
- **Read pages** — get full frontmatter, Markdown body, word count, and reading time for any page
- **Create content** — write new posts, pages, and projects with valid frontmatter
- **Build the site** — trigger a full build and get structured results back
- **Deploy** — push to S3 and invalidate CloudFront
- **Inspect templates** — see what data a layout receives, resolve which layout a page will use
- **Validate frontmatter** — check YAML against the expected schema before writing

The server also exposes resources (`forge://config`, `forge://pages`, `forge://taxonomies`, `forge://build/status`) and prompt templates for common workflows like creating a blog post or diagnosing build errors.

The idea is straightforward: if the AI has structured access to your site's content and toolchain, it can do more than just edit files. It can understand the site's structure, create content that fits the existing taxonomy, and verify its work by building the site — all without you having to explain the file layout or frontmatter format in every prompt.

This post was written using that exact workflow. The MCP server is running, Claude Code has access to the content graph, and the feedback loop is: write, build, check, iterate.

## Architecture Highlights

A few decisions worth noting:

**Embedded default theme.** The default theme is compiled into the binary via `go:embed`. When you scaffold a new site with `forge new site`, it copies the theme out. This means the tool works out of the box without downloading anything — and the embedded theme serves as a fallback if a user's theme is missing a layout.

**Tailwind CSS management.** Forge downloads the Tailwind standalone CLI automatically and caches it in `~/.forge/bin/`. The CSS input file uses Tailwind v4's CSS-first configuration — `@theme`, `@utility`, `@plugin` — so there's no `tailwind.config.js` and no Node.js anywhere in the pipeline. A version marker file ensures the cached binary is re-downloaded when Forge upgrades to a new Tailwind version.

**Content-addressed deploy.** The deploy command computes MD5 hashes for local and remote files, uploads only what changed, sets cache headers based on content type (immutable for hashed assets, short TTL for HTML), and issues a targeted CloudFront invalidation for changed paths. This keeps deploys fast and bandwidth-efficient.

**Scaffold system.** `forge new site` generates a complete, working site with sample content, a homepage, an about page, a blog post, and a project entry. The seed content is enough to see all features working without reading documentation first.

## What's Next

Forge is functional and powers this site, but there are areas I want to improve:

- **Image processing** — automatic resizing, WebP conversion, responsive `srcset` generation
- **Incremental builds** — only re-render pages whose content or dependencies changed
- **More MCP capabilities** — content suggestions based on existing posts, broken link detection, SEO analysis

The broader goal is to make Forge the backbone of an AI content pipeline where the MCP server is the primary interface. Write a prompt, get a post, build the site, deploy — all from a conversation. The tooling is getting closer to making that loop seamless.

Source is at [github.com/ellingwood/forge](https://github.com/ellingwood/forge).
