---
title: "Forge"
date: 2026-02-19T00:36:22-08:00
draft: false
description: "A fast, opinionated static site generator written in Go. Built for developers who want Hugo's speed without the complexity."
params:
  tech:
    - Go
    - Tailwind CSS
    - MCP
github: "https://github.com/ellingwood/forge"
---

## Overview

Forge is a static site generator written in Go, designed to be fast and straightforward. It takes Markdown content with YAML frontmatter, runs it through Go HTML templates, and produces a static site. No webpack, no npm, no build pipeline — just a single binary.

It powers this site.

## Features

- **Live reload dev server** — `forge serve` watches for changes and reloads the browser automatically
- **Tailwind CSS support** — built-in processing with no external tooling required
- **Taxonomy system** — tags, categories, and custom taxonomies with auto-generated term pages
- **RSS and Atom feeds** — generated automatically from your content
- **Client-side search** — builds a JSON search index at compile time
- **Clean URLs** — `/blog/my-post/` instead of `/blog/my-post.html`
- **MCP server** — exposes site content and build tools to AI editors via `forge mcp`

## Architecture

Forge follows a theme-based layout system. Content lives in `content/`, themes provide layouts and assets in `themes/`, and root-level `layouts/` can override any theme template.

Templates use Go's `html/template` with added helpers for partials, blocks, and content rendering. Site configuration is a single `forge.yaml` file.

The MCP server lets Claude Code (and other MCP-compatible tools) query content, create pages, and trigger builds without leaving the editor.

## Getting Started

```bash
# Start the dev server
forge serve

# Build for production
forge build
```
