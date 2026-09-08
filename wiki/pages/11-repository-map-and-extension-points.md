---
grok_wiki: true
page_id: repository-map-and-extension-points
title: Repository map and extension points
repository: error-pages
branch: docs/generate-wiki
ref: 6d3ced4
generated_at: 2026-09-04T07:07:12Z
source_files:
  - README.md
  - go.mod
  - error-pages.yml
  - schemas/config/1.0.schema.json
  - templates/readme.md
  - l10n/readme.md
  - internal/config/config.go
  - internal/http/server.go
  - internal/tpl/render.go
---

# Repository map and extension points

The repository keeps product assets and Go implementation separate:

| Area | Location | Extension point |
| --- | --- | --- |
| Process entrypoint | `cmd/error-pages` | Keep command assembly in `internal/cli` |
| CLI/config | `internal/cli`, `internal/config`, `internal/env`, `internal/options` | Add validated config or flag/env input at the owning boundary |
| HTTP | `internal/http` | Register a handler in `Server.Register`; reuse core response helpers where applicable |
| Rendering | `internal/tpl` | Add tagged properties or template functions deliberately |
| Selection | `internal/pick` | Add a picker mode only if selection semantics are distinct |
| Assets | `templates`, `error-pages.yml`, `l10n` | Add templates/config entries and localization keys |
| Public contract | `schemas`, `test/hurl` | Update schema and executable HTTP assertions together |
| Delivery | `Dockerfile`, `.github/workflows`, `Makefile` | Preserve static generation and runtime smoke checks |

## Adding an HTML template

Create the HTML file under `templates/`, add its path to the `templates` list in [`error-pages.yml`](../../error-pages.yml), and use the existing tokens/conditional functions. The default Docker build and generator CI will include it automatically because they iterate the config. If the template adds visible English strings, add matching localization keys in [`l10n/l10n.js`](../../l10n/l10n.js).

## Adding a response format

The config schema currently documents only JSON and XML format objects. Runtime selection is centralized in [`formats.go`](../../internal/http/core/formats.go), while rendering is centralized in [`RespondWithErrorPage`](../../internal/http/core/errorpage.go). A new format therefore needs schema/config support, MIME recognition, response content type handling, a responder branch, and contract tests.

## Adding an endpoint

Create a handler under `internal/http/handlers`, register it in [`Server.Register`](../../internal/http/server.go), decide whether middleware should observe it—which it will by default—and add Hurl coverage when it is externally visible. Health-like endpoints should be considered separately from error-page rendering because the router currently gives them dedicated handlers.

## Compatibility notes

The module targets Go 1.18 and depends on fasthttp, Cobra, YAML v3, Prometheus, zap, and a small set of support libraries. The public schema is narrower than the runtime model: runtime validation accepts the configured page-code pattern checked in Go, while the JSON schema documents an alphanumeric/underscore/hyphen pattern.

See [[01-repository-overview]] for the high-level shape and [[02-cli-and-configuration]] for the input contract.

<details>
<summary>Relevant sources</summary>

- [`README.md`](../../README.md)
- [`go.mod`](../../go.mod)
- [`error-pages.yml`](../../error-pages.yml)
- [`schemas/config/1.0.schema.json`](../../schemas/config/1.0.schema.json)
- [`templates/readme.md`](../../templates/readme.md)
- [`l10n/readme.md`](../../l10n/readme.md)
- [`internal/config/config.go`](../../internal/config/config.go)
- [`internal/http/server.go`](../../internal/http/server.go)
- [`internal/tpl/render.go`](../../internal/tpl/render.go)
</details>
