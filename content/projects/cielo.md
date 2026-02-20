---
title: "Cielo"
date: 2026-02-19
draft: false
description: "A Kanban board built for AI agent orchestration. Agents claim tasks, track dependencies, and coordinate work through a built-in MCP server — while humans watch it all happen in real time."
params:
  tech:
    - Go
    - React
    - SQLite
    - MCP
github: "https://github.com/ellingwood/cielo"
---

## Overview

Cielo is a Kanban board designed for multi-agent AI workflows. It looks like Trello — boards, lists, cards, drag-and-drop — but every feature is built with AI agent interaction as a first-class concern. A built-in MCP server exposes 20 tools that let agents read board state, claim tasks, manage dependencies, and update progress programmatically.

## Features

- **MCP server** — 20 JSON-RPC tools for full board manipulation by AI agents
- **Actor attribution** — every mutation is logged with who did it, enabling multi-agent debugging
- **Card dependencies** — directed graph of blocking relationships between tasks
- **Real-time sync** — Server-Sent Events push updates to all connected clients instantly
- **Drag-and-drop UI** — familiar Kanban interface for human operators watching agent workflows
- **Priority and status tracking** — cards flow through unassigned, assigned, in_progress, blocked, and done
- **Full-text search** — filter by title, assignee, status, or label within a board
- **Single binary** — Go backend, React frontend, and SQLite database in one deployable artifact

## Architecture

Cielo runs as a single Go process serving the REST API, MCP server, and embedded React frontend. SQLite with WAL mode handles persistence — no external database needed. An internal event bus decouples mutations from real-time broadcast via SSE.

The MCP server shares the same service layer as the REST API, so agents and the web UI have identical capabilities. Agents identify themselves via an `actor` parameter on every write operation, creating an immutable audit trail.

## Getting Started

```bash
# Build and run
make build && make run

# Or with Docker
docker compose up -d
```

The board is available at `http://localhost:8080`. Point your MCP client at the same address to connect agents.
