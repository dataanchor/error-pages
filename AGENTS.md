<!--
Harness knowledge file.
Update when the corresponding architecture or workflow changes.
Do not add speculative information.
-->

# AGENTS.md

Guidance for AI coding agents working in this repository. Human-written notes in
this file take precedence; extend, do not silently remove them.

## What this is

`error-pages` (`github.com/tarampampam/error-pages`) is a Go 1.18 project that
turns a YAML config plus HTML templates into either:

- a fast HTTP server (`serve`) that renders error pages on demand, or
- a static HTML asset tree (`build`) written to disk.

Both paths share the renderer in `internal/tpl`. The Docker image runs `build` at
image-build time and ships the static output alongside the server binary.

## Where things live

| Area | Path | Notes |
| --- | --- | --- |
| Entry point | `cmd/error-pages/main.go` | thin; delegates to `internal/cli` |
| CLI commands | `internal/cli/{root,serve,build,healthcheck,version}` | Cobra tree |
| Config load | `internal/config/config.go` | YAML + envsubst → `Config` |
| Flags/env | `internal/cli/serve/flags.go`, `internal/env/env.go` | precedence: CLI flag > env var > default |
| HTTP wiring | `internal/http/server.go` (`Server.Register`) | fasthttp + `fasthttp/router` |
| Response core | `internal/http/core/errorpage.go`, `formats.go`, `headers.go` | negotiation, status, proxied headers |
| Handlers | `internal/http/handlers/*` | one package per route |
| Rendering | `internal/tpl/render.go`, `properties.go` | Go `text/template` + 2s in-memory cache |
| Template selection | `internal/pick/`, `internal/cli/serve/command.go` | first / random / random-per-request / daily / hourly |
| Assets | `templates/*.html`, `error-pages.yml`, `l10n/l10n.js` | page set + browser-side localization |
| Public contract | `schemas/config/1.0.schema.json`, `test/hurl/*.hurl` | keep in sync with runtime |
| Delivery | `Dockerfile`, `.github/workflows/`, `Makefile` | |

## Deeper context

A full generated wiki lives in `wiki/pages/` (11 pages). Route by task:

- Architecture / request trace → `wiki/pages/03-http-routing-and-request-flow.md`, `ARCHITECTURE.md`
- CLI & config inputs → `wiki/pages/02-cli-and-configuration.md`
- Response formats & error contract → `wiki/pages/04-response-formats-and-error-contracts.md`
- Rendering & localization → `wiki/pages/05-template-rendering-and-localization.md`
- Template selection → `wiki/pages/06-template-selection.md`
- Static generation → `wiki/pages/07-static-page-generation.md`
- Ops endpoints (`/healthz`, `/metrics`, `/version`) → `wiki/pages/08-operational-endpoints-and-observability.md`
- Container & release → `wiki/pages/09-container-and-release-delivery.md`
- Tests & CI → `wiki/pages/10-tests-and-ci-verification.md`
- Extension points → `wiki/pages/11-repository-map-and-extension-points.md`

Commands and local dev: `docs/development.md`. Machine-readable summary:
`.harness/service.yaml`, routing table: `.harness/context.yaml`.

## Commands (host toolchain)

```
go build -trimpath -o ./error-pages ./cmd/error-pages/   # build
go test -race ./...                                        # unit tests
golangci-lint run                                          # lint (CI pins v1.44)
```

`make` targets (`build`, `fmt`, `lint`, `gotest`, `int-test`, `test`, `up`) run
through Docker Compose; use them when you don't have the toolchain locally. The
Hurl integration suite (`make int-test`) needs the compose `web` service.

## Invariants (verified in code)

- `internal/tpl` is the only renderer for both `serve` and `build`; keep template
  tokens/functions changes there so both paths stay consistent.
- All HTTP error responses go through `core.RespondWithErrorPage`. It sets
  `X-Robots-Tag: noindex`, negotiates format from `Content-Type` / `X-Format`
  (only `json` and `xml` are special-cased; everything else is HTML), and proxies
  only the headers named in `--proxy-headers`.
- `/` honors the `X-Code` request header (1–599) to override the default page and
  status; `/{code}.html` always responds `200` with the page body for that code.
- Config template `path` values are resolved relative to the config file's
  directory (the loader `os.Chdir`s there and back). `envsubst` runs on the YAML,
  so `$VAR` in `error-pages.yml` is expanded from the environment.
- Page codes must not contain whitespace (`config.Validate`); the JSON schema
  pattern is stricter than the Go check — update both together.
- `TemplateRenderer` starts a cleanup goroutine; callers must `Close()` it
  (server does this in `Server.Stop`).
- Docker runtime image is `scratch`, runs as UID `10001`, healthcheck shells
  `/bin/error-pages healthcheck`.

## Guardrails

- Do not change response contracts, status codes, header names, or the config
  schema without updating `schemas/` and `test/hurl/` together.
- CI ignores Markdown-only changes for build/test jobs but still lints config and
  l10n JS. `error-pages.yml` must validate against `schemas/config/1.0.schema.json`
  (`ajv`), and `l10n/*.js` must pass ESLint.
- Keep `go.mod` at Go 1.18 and avoid adding dependencies unless necessary.
