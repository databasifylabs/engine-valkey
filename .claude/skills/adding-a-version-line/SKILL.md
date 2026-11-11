---
name: adding-a-version-line
description: "Add a new Valkey version line to this engine repository: the versions.json entry and its pinned tools, any per-line differences in the engine code, and the local build, smoke, conformance and capabilities check. Use when asked to support a new Valkey major or minor release line. Do not use for patch bumps inside an existing line (the weekly refresh opens those)."
---

# Adding a Valkey version line

A Valkey line is a major (`9`). It takes the newest release of its major. The weekly refresh never moves a pin out of its line.

## The versions.json entry

1. Copy the newest `valkey-*` entry under `images` to the key `valkey-<line>`. Set `line`, `release` (the exact official tag), and `base.tag` to the official tag, keeping the suffix the other lines use.
2. Set `base.digest`. Get it from `docker buildx imagetools inspect <image>:<tag> --format '{{json .Manifest}}' | jq -r .digest`.
3. Set every package version to `"0"`, then run `$IMAGES refresh valkey-<line>`. It pins the packages and moves the base, copies, modules and sources to the newest patch of the line. It may also move the shared `go` digest: revert that unless you mean to rebuild every image.
4. Run `$IMAGES check`. It fails when a key isn't `<engine>-<line>`, when a pin left its line, or when `sdk` differs from `go.mod`.

## What differs per line in this engine

The image sets `DATABASIFY_LINE` from the line, and the engine reads it where releases differ:

- The engine code is the `Valkey` flavor of [engine-sentinel-family](https://github.com/databasifylabs/engine-sentinel-family), shared with engine-redis. Its `catalog.go` offers live `io-threads` changes only from the line in `liveIoThreadsFrom`, using `version.NewerThan` on `DATABASIFY_LINE`; a per-line difference is a pull request there first, then a bump in this repository's `go.mod`. Read the release notes for other settings that changed.
- redis-shake (`sources.redis-shake`) does the migrations. Check that it supports the new major's replication protocol and RDB version.
- `smoke/check.sh` checks `valkey_version`, `valkey-sentinel` and `redis-shake`.

Add or extend a Go test for each per-line branch you add.

## Check the new line

`$IMAGES` stands for `tools/images.sh` in a clone of [engine-sdk](https://github.com/databasifylabs/engine-sdk) at the SDK version this repository's `go.mod` requires. Run it from this repository's root:

```bash
go vet ./... && go test -race -count=1 ./...
$IMAGES check
$IMAGES build valkey-10 --load --tag smoke:valkey-10
$IMAGES smoke smoke:valkey-10 valkey
echo '{}' | docker run --rm -i --network none --entrypoint databasify smoke:valkey-10 engine capabilities | jq -S .
echo '{}' | docker run --rm -i --network none --entrypoint databasify smoke:valkey-10 config fields | jq -S .
```

- `smoke` runs `images/valkey/smoke/check.sh` against the new line, then the SDK's conformance suite.
- Compare `engine capabilities` and `config fields` with the closest existing line. Every difference should be one you meant, such as a password encryption or setting the new release adds.
- Also run the CI scan (`TRIVY_IMAGE` in the SDK's `.github/workflows/engine-images.yml`) on `smoke:valkey-10`.
- Put the results in the pull request with the line and OS you ran them on.

## The pull request

Open an issue first, branch from `main`, and open one pull request against `main` with the `versions.json` entry, any per-line code and tests, and the README's Lines section. The app registers the new line in its own repository ([databasifylabs/databasify](https://github.com/databasifylabs/databasify)), after this one merges.
