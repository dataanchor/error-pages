<!--
Harness knowledge file.
Update when the corresponding architecture or workflow changes.
Do not add speculative information.
-->

# Architecture

Concise map for agents. The full, citation-backed deep dive is the generated
wiki in [`wiki/pages/`](wiki/pages/); this file is the index and the invariants.

## Shape

Two products, one rendering core:

```
error-pages.yml + templates/*.html
        │
        ├── serve  ─► internal/cli/serve ─► internal/http (fasthttp) ─► core.RespondWithErrorPage ─┐
        │                                                                                          ├─► internal/tpl.Render
        └── build  ─► internal/cli/build ─► iterate templates × pages ─► write *.html to disk ──────┘
```

- `serve`: loads config, picks a template-selection strategy, registers fasthttp
  routes, renders per request.
- `build`: loads the same config, writes one HTML file per template/page pair
  (optionally an `index.html`). Invoked during `docker build` to bake the static
  tree into the image (`/opt/html`).
- `internal/tpl` is shared, so template tokens, functions, and rendering errors
  behave identically on both paths.

## Components

| Component | Package | Responsibility |
| --- | --- | --- |
| Entry | `cmd/error-pages` | build `os.Args[0]`-named Cobra command, exit code |
| CLI | `internal/cli` (+ `serve`, `build`, `healthcheck`, `version`) | flag/env parsing, logging setup, lifecycle |
| Config | `internal/config` | YAML + `envsubst` → `Config{Templates, Pages, Formats}`; validation |
| Options | `internal/options` | resolved runtime options passed to handlers |
| Env | `internal/env` | typed env-var names + `Lookup` |
| HTTP server | `internal/http` | fasthttp server, route registration, graceful stop |
| Response core | `internal/http/core` | format negotiation, status codes, header proxying, `RespondWithErrorPage` |
| Handlers | `internal/http/handlers/*` | `index`, `errorpage`, `healthz`, `metrics`, `version`, `notfound` |
| Middleware | `internal/http/common` | request logging, per-request duration metrics |
| Renderer | `internal/tpl` | `text/template` execution, extra funcs, short-lived cache |
| Picker | `internal/pick` | first / random-once / random-every-time, with optional daily/hourly interval |
| Checkers | `internal/checkers` | liveness (`/healthz`) and self healthcheck subcommand |
| Metrics | `internal/metrics` | Prometheus registry + counters/histograms |
| Breaker | `internal/breaker` | OS signal → context cancel |

## Request flow (serve)

1. `common.DurationMetrics(common.LogRequest(router.Handler))` wraps every request.
2. Router dispatches:
   - `GET /` → `index` handler: default page/code, or `X-Code` header (1–599) override.
   - `GET /{code}.html` → `errorpage` handler: that code, HTTP `200`.
   - `GET /version`, `ANY /healthz` (+ `/health/live`), `GET /metrics`.
   - unmatched → `notfound` handler.
3. `core.RespondWithErrorPage`: sets `X-Robots-Tag: noindex`; chooses JSON/XML
   (from `Content-Type` or `X-Format`) if that format is configured, else HTML via
   the picker; fills `tpl.Properties` (adds Ingress detail headers when
   `--show-details`); proxies allow-listed request headers to the response;
   renders and writes.

## Key invariants

See `AGENTS.md` → "Invariants". Most load-bearing:

- Single rendering path (`internal/tpl`) for `serve` and `build`.
- Single response path (`core.RespondWithErrorPage`) for all error output.
- Config `path` entries resolve relative to the config file's directory.
- Public HTTP contract is pinned by `test/hurl/*.hurl` and
  `schemas/config/1.0.schema.json` — change them alongside code.

## Non-goals / absent concerns

Stateless service: no database, no message queue, no external service calls at
runtime. The only cache is a 2-second in-process template-output cache.
