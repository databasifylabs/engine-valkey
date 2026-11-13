# Databasify Valkey Engine

- [Introduction](#introduction)
- [Lines](#lines)
- [Features](#features)
- [Ports](#ports)
- [Running Locally](#running-locally)
- [Adding a Version Line](#adding-a-version-line)

## Introduction

The Valkey engine for [Databasify](https://github.com/databasifylabs/databasify), built on the [engine SDK](https://github.com/databasifylabs/engine-sdk).

Each image is the official `valkey` image plus its tools and `databasify`, the [engine SDK](https://github.com/databasifylabs/engine-sdk) 1.0 runtime that runs as PID 1. Images are published as `ghcr.io/databasifylabs/databasify-valkey`.

Engine behaviour lives in the Databasify app: its PHP driver for this engine runs Bash scripts inside the container through the runtime. This repository changes only when the database or one of its tools moves to a new version.

## Lines

Valkey 7, 8 and 9. Each line takes the newest release of its major and never leaves it. The weekly refresh moves the pins in `versions.json`.

## Features

| Feature | Valkey |
| --- | --- |
| Node lifecycle, status, metrics, TLS | ✓ |
| Databases and users with read/write grants | one password |
| Password encryptions | — |
| High availability | Sentinel |
| Automatic and manual failover, force quorum | ✓ |
| Standbys (up to 2) and read-only nodes (any number) | ✓ |
| Primary key required in a cluster | — |
| Connection pools | — |
| Backups and point-in-time restore | — |
| Continuous migration | redis-shake |
| Fork (one-shot copy) | ✓ |
| Query insights | `SLOWLOG` |
| Host tuning | ✓ |

## Ports

| Port | Use |
| --- | --- |
| 25061 | `client`, for applications |
| 6379, 26379 | `peers`: replication and Sentinel |

## Running Locally

`$IMAGES` is `tools/images.sh` in a clone of [engine-sdk](https://github.com/databasifylabs/engine-sdk) at the tag the `sdk` pin in `versions.json` names (`v1.0.0`). Run it from this repository's root:

```bash
$IMAGES check
$IMAGES build valkey-9 --load --tag smoke:valkey-9
$IMAGES smoke smoke:valkey-9 valkey
```

The smoke test starts the image, runs `images/valkey/smoke/check.sh` and then the SDK's runtime conformance suite.

## Adding a Version Line

Follow `.claude/skills/adding-a-version-line/SKILL.md`. It covers the `versions.json` entry, what differs per line, and the checks to run.
