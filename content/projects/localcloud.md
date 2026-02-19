---
title: "LocalCloud"
description: "A wire-compatible AWS and GCP cloud emulator that runs as a single Go binary. Point your existing SDKs at localhost and develop without an internet connection."
params:
  tech:
    - Go
    - SQLite
    - React
    - AWS
    - GCP
github: "https://github.com/ellingwood/localcloud"
---

## Overview

LocalCloud is a local cloud emulator that provides wire-compatible implementations of AWS and GCP services. It runs as a single Go binary — start it with `localcloud start`, point your existing SDKs at `localhost:4566`, and develop without an internet connection or cloud account.

Your existing tools work as-is. Boto3, AWS SDK v2, the gcloud CLI — just change the endpoint URL. Everything else stays the same.

## Supported Services

**AWS (9 services):**
S3, VPC/EC2, Route53, CloudFront, DynamoDB, SQS (standard + FIFO), SNS, EventBridge, and ElastiCache (Valkey/Redis-compatible via RESP protocol).

**GCP (8 services):**
Cloud Storage, VPC Networks, Cloud DNS, Cloud CDN, Firestore, Pub/Sub, Memorystore, and Eventarc.

## Architecture

LocalCloud uses a plugin architecture where every service implements the same interface, registers at compile time via `init()`, and gets its routes mounted automatically. Adding a new service means implementing the interface and adding one import line.

Key design decisions:

- **Single binary** — all 17 services, one process, one port
- **SQLite with WAL mode** for all metadata storage
- **Filesystem-backed object storage** for S3 and GCS
- **Native RESP protocol server** for ElastiCache, so Redis clients connect without any adapter
- **SigV4 auth parsing** for multi-tenant isolation without requiring shared secrets
- **Multi-provider routing** — the gateway detects AWS vs GCP requests from headers and URL patterns, routing through a single port

## Admin UI

LocalCloud ships with a React admin dashboard that provides visibility into everything running locally. Browse S3 buckets, send SQS messages, inspect DynamoDB tables, view request logs, and manage organizations from your browser.

## Getting Started

```bash
# Clone and build
git clone https://github.com/ellingwood/localcloud.git
cd localcloud
go build -o localcloud ./cmd/localcloud

# Start the emulator
./localcloud start

# Use your existing SDKs — just change the endpoint
aws --endpoint-url http://localhost:4566 s3 mb s3://my-bucket
```
