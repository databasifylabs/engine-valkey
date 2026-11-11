# Databasify Valkey Engine

- [Introduction](#introduction)
- [Lines](#lines)
- [Features](#features)
- [Ports](#ports)
- [Engine Notes](#engine-notes)
- [Running Locally](#running-locally)
- [Adding a Version Line](#adding-a-version-line)

## Introduction

The Valkey engine for [Databasify](https://github.com/databasifylabs/databasify), built on the [engine SDK](https://github.com/databasifylabs/engine-sdk).

Each image is the official `valkey` image plus `databasify`, the engine binary that runs as PID 1 and answers the SDK's typed commands. Images are published as `ghcr.io/databasifylabs/databasify-valkey`.

The commands and the flows Databasify runs are the same for every engine; see the SDK's [Flows](https://github.com/databasifylabs/engine-sdk#flows).

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
| Log forwarding and host tuning | ✓ |

A command the engine doesn't offer answers `unsupported`; `engine capabilities` lists the ones it does.

## Ports

| Port | Use |
| --- | --- |
| 25061 | `client`, for applications |
| 6379, 26379 | `peers`: replication and Sentinel |

## Engine Notes

- The engine code is the `Valkey` flavor of [engine-sentinel-family](https://github.com/databasifylabs/engine-sentinel-family). This repository holds only the entry point and the images.
- Persistence is the `appendonly` setting.
- To remove a node, every remaining node runs `node reset` with the remaining hosts, instead of `member forget`.
- `node metrics` also reports `keys`, `hits`, `misses` and `evicted`.

## Running Locally

`$IMAGES` is `tools/images.sh` in a clone of [engine-sdk](https://github.com/databasifylabs/engine-sdk) at the version `go.mod` requires. Run it from this repository's root:

```bash
go vet ./... && go test -race -count=1 ./...
$IMAGES build valkey-9 --load --tag smoke:valkey-9
$IMAGES smoke smoke:valkey-9 valkey
```

The smoke test starts the image, runs `images/valkey/smoke/check.sh` and then the SDK's conformance suite.

## Adding a Version Line

Follow `.claude/skills/adding-a-version-line/SKILL.md`. It covers the `versions.json` entry, the code that differs per line in this engine, and the checks to run.
