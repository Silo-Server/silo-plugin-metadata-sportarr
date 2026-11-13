# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```sh
make build          # Build binary → ./plugin
make test           # go test ./...
make lint           # golangci-lint run ./...
make clean          # Remove binary
make build-all      # Cross-compile for linux/amd64, linux/arm64, darwin/arm64
go test -run TestName ./provider/  # Run a single test
go vet ./...        # Static analysis (also runs in CI)
```

Version is injected at build time via `-X main.version=...` from git tags.

## SDK Dependency

The SDK (`github.com/Silo-Server/silo-plugin-sdk`) is a private module. For local development, use a `go.work` file or temporary `replace` directive pointing at a local checkout — but never commit filesystem-local replaces. CI enforces this check and runs with `GOWORK=off`.

CI also sets `GOPROXY=direct`, `GOPRIVATE=github.com/Silo-Server/*`, `GONOSUMDB=github.com/Silo-Server/*`.

## Architecture

This is a Silo metadata-provider plugin that serves sports league data from the Sportarr API as TV series metadata (leagues→series, seasons→seasons, events→episodes).

**Plugin entry point (`main.go`):** Embeds `manifest.json`, configures the gRPC plugin server via the SDK's `runtime.Serve`. Two server structs — `runtimeServer` handles `GetManifest`/`Configure`; `metadataServer` handles all metadata RPCs (Search, GetMetadata, GetSeasons, GetEpisodes, GetImages, ResolveImageURL/URLs). Image URLs are converted to/from a `sportarr://` canonical scheme for storage-agnostic persistence.

**`provider/`:** Business logic layer. `Provider` wraps `Client` and translates between the `metadata` domain types and the Sportarr HTTP API. `Client` is the HTTP client with rate limiting (`golang.org/x/time/rate`) and retry logic (exponential backoff on 5xx/429). API types are in `types.go`.

**`metadata/`:** Domain types only — `SearchQuery`, `MetadataResult`, `SeasonResult`, `EpisodeResult`, `RemoteImage`, etc. No logic.

**Testing:** Tests use `httptest.NewServer` to mock the Sportarr API. `newTestProvider` in `provider_test.go` wires up a test server → client → provider.

## Release

A release is a merged pull request that sets `version` in `manifest.json`, then a maintainer pushes the matching `v<version>` tag on that commit. The tag runs `release.yml`, which calls the shared `Silo-Server/.github` plugin release workflow to build, sign, publish, and notify the `silo-plugins` catalog. There is no manual-dispatch release. See [CONTRIBUTING.md](CONTRIBUTING.md#release).
