---
title: "Building LocalCloud: A Local Cloud Emulator in Go"
date: 2026-02-18
draft: true
description: "How I built a wire-compatible AWS and GCP emulator as a single Go binary, and why local cloud development shouldn't require an internet connection."
tags:
  - go
  - aws
  - cloud
  - open-source
  - claude-code
project: localcloud
series: "Building Enterprise at Home"
---

This is the first post in a series I'm calling **Building Enterprise at Home** — a collection of posts about building applications that mimic enterprise cloud services for local development. The idea is simple: if you can run it locally, you can develop faster, test more reliably, and stop worrying about cloud bills during prototyping. This series documents the journey.

## The Origin Story

I'll get this out of the way up front: [LocalStack](https://localstack.cloud/) exists, and it's good. If you need a battle-tested local AWS emulator today, use it. I have. This project isn't trying to replace it.

The seed for LocalCloud came from a pattern I kept seeing at work. A manager on my team was creating one-off mock services for various cloud APIs — little standalone servers that pretended to be S3, or SQS, or whatever the team needed for local testing that week. Each one was its own project, its own binary, its own set of quirks. I kept thinking: why not combine all of these into one tool?

But honestly, the real reason I built it is because it's fun. There's something satisfying about figuring out how these APIs actually work by reimplementing them yourself. You learn more about S3's multipart upload semantics from building a compatible version than you ever will from reading the docs. And building it as a single Go binary — no Docker, no Python, no runtime dependencies — felt like a worthwhile constraint to design around.

That's how LocalCloud started. Not because the world needed another cloud emulator, but because I wanted to build one.

## What is LocalCloud?

LocalCloud is a single Go binary that provides wire-compatible implementations of AWS and GCP services. You use your existing SDKs — Boto3, AWS SDK v2, the gcloud CLI — and just change the endpoint URL. Everything else stays the same.

Here's what it supports today:

**AWS Services:**

- **Storage:** S3 (with filesystem-backed object storage)
- **Networking:** VPC/EC2 (security groups, subnets), Route53 (DNS), CloudFront (CDN with LRU cache)
- **Database:** DynamoDB
- **Messaging:** SQS (standard + FIFO queues), SNS, EventBridge
- **Cache:** ElastiCache (Valkey/Redis-compatible via the RESP protocol)

**GCP Services:**

- Cloud Storage, VPC Networks, Cloud DNS, Cloud CDN, Firestore, Pub/Sub, Memorystore, Eventarc

Compute isn't supported yet — Lambda, ECS, Cloud Functions, and Cloud Run are stubbed but not functional. The most interesting direction there is EKS/GKE support that just spins up a [k3s](https://k3s.io/) cluster under the hood. That would give you a real Kubernetes API locally without the weight of a full cloud control plane, and it would cover a lot of the use cases that individual compute services address. Functions and container services are lower priority by comparison.

All metadata lives in SQLite with WAL mode. Object data (S3, GCS) goes to the filesystem. ElastiCache runs an actual RESP protocol server so your Redis clients connect natively. The whole thing starts up in under a second.

It also ships with a React admin UI that gives you dashboards for every service — you can browse S3 buckets, send SQS messages, inspect DynamoDB tables, view request logs, and manage organizations, all from your browser.

## The Development Process

I built LocalCloud with [Claude Code](https://docs.anthropic.com/en/docs/build-with-claude/claude-code/overview), and the experience fundamentally shaped how the project turned out. The iterative workflow was key — I'd describe what I wanted, review the implementation, refine, and repeat.

The project started with a detailed spec. I laid out the architecture, the service interface, the routing model, and the persistence layer before writing any service code. That spec became the blueprint that kept everything consistent as services were added.

From there, the build order went roughly like this:

1. **Core infrastructure first** — the HTTP gateway, plugin registry, SQLite layer, and SigV4 auth middleware
2. **Foundational services** — VPC, Route53, S3, CloudFront, ElastiCache, and SQS, since these are the services I use most
3. **Expanding coverage** — DynamoDB, SNS, EventBridge
4. **Admin UI** — a React dashboard for visibility into everything running locally
5. **GCP support** — extending the router and auth middleware to handle GCP requests, then implementing 8 GCP services

The plugin architecture turned out to be the best decision early on. Every service implements the same interface, registers itself at compile time via `init()`, and gets its routes mounted automatically. Adding a new service means implementing the interface and adding one import line. The consistency this created across 17 services is something I'm genuinely proud of.

The code quality that came out of the Claude Code workflow impressed me. The services are clean, consistent, and well-tested. When I went back to add GCP support months after the initial AWS implementation, the plugin system made it almost trivial — the patterns were already established and I just had to follow them.

## Architecture Highlights

A few of the more interesting technical decisions:

**Plugin System.** Every service implements a `Service` interface with methods for initialization, route registration, health checks, and lifecycle management. Services register themselves via Go's `init()` mechanism, so the main binary just imports them and the registry handles the rest. This made it possible to grow from 6 services to 17 without any changes to the core framework.

**Wire-Compatible Authentication.** AWS requests use SigV4 signing. Rather than fully validating cryptographic signatures (which would require sharing secret keys in a specific way), LocalCloud parses the SigV4 header to extract the access key, looks it up in the database, and associates the request with the right organization. It's lenient enough for local development while still supporting multi-tenant isolation.

**Dynamic Port Allocation.** Some services need their own listeners — ElastiCache needs a RESP protocol server, CloudFront runs an HTTP proxy. LocalCloud has a port allocator that assigns ports from a configurable range and persists allocations in SQLite so they're stable across restarts.

**RESP Protocol for ElastiCache.** Rather than faking Redis over HTTP, ElastiCache runs an actual RESP protocol server. Your Redis client connects to it like a real Redis instance. This was important for compatibility — most Redis clients don't have a concept of "HTTP mode."

**Multi-Provider Routing.** The gateway detects whether an incoming request is for AWS or GCP based on the Authorization header format, Host header, and URL path patterns, then routes to the appropriate service. This means a single port handles both providers.

## What's Next

This post is the overview — future posts in the series will dive deeper into specific aspects:

- The plugin system design and how it enables rapid service development
- Building wire-compatible SigV4 authentication from scratch
- Implementing the RESP protocol for Redis compatibility
- The admin UI architecture and how it provides observability into a local cloud
- Lessons from building a large Go project with Claude Code

If you're interested in trying LocalCloud, it's open source at [github.com/ellingwood/localcloud](https://github.com/ellingwood/localcloud). Pull it down, run `localcloud start`, and point your SDKs at `localhost:4566`.

More to come.
