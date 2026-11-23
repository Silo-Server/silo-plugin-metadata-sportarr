# Silo Sportarr Plugin

First-party Silo metadata plugin backed by [Sportarr](https://sportarr.net). Provides sports league metadata as TV shows, with series, seasons, and episodes mapped from leagues, seasons, and events.

## Episode Dates and Labels

Sportarr numbers a league's events `S<year>E<n>` so they sort like a TV season.
Those numbers mean nothing to viewers, so series returned by this plugin carry
`metadata.episode_labels = "air_date"`, which asks Silo to label their episodes
by date instead.

An episode's air date is Sportarr's `broadcast_date`: the day of the event in
its broadcast time zone, which is also the date Sportarr writes into file names.
The plugin falls back to `air_date`, the UTC date of the start, only when
`broadcast_date` is missing. `air_date` can be a day off: an evening game in the
Americas lands on the next day, and an afternoon event in Australia on the day
before.

## Dependency Model

This repository consumes `github.com/Silo-Server/silo-plugin-sdk` as a normal Go module dependency. CI and release builds run with `GOWORK=off` and expect the SDK version in `go.mod` to resolve from a published semver tag.

For local multi-repo development, use a temporary `replace` or a local `go.work` that points at `dev/github/silo-plugin-sdk`. Do not commit machine-local filesystem replaces as the supported release path.

## Development

```sh
go test ./...
go build .
```

## Attribution

Metadata provided by [Sportarr](https://sportarr.net).

## Releases

Releases are signed with GitHub artifact attestations. [CONTRIBUTING.md](CONTRIBUTING.md#release)
describes how maintainers cut a release and how to verify a downloaded binary.

## License

`silo-plugin-metadata-sportarr` is licensed under `AGPL-3.0-or-later`. See [LICENSE](LICENSE).
