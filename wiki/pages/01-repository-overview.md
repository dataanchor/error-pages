---
grok_wiki: true
page_id: repository-overview
title: Repository overview
repository: error-pages
branch: docs/generate-wiki
ref: 6d3ced4
generated_at: 2026-09-04T07:07:12Z
source_files:
  - README.md
  - cmd/error-pages/main.go
  - internal/cli/root.go
  - internal/http/server.go
---

# Repository overview

`error-pages` is a Go application that turns configured error-page definitions and HTML templates into either an HTTP service or a static asset tree. The module is `github.com/tarampampam/error-pages` and the inspected source is tagged `v2.15.0` at ref `6d3ced4`.

## Two products, one rendering core

- `serve` loads configuration, chooses a template picker, registers fasthttp routes, and renders responses in-process.
- `build` loads the same configuration and writes one HTML file per configured template/page combination.
- `internal/tpl` is shared by both paths, so template tokens and rendering errors behave consistently.
- The Docker build invokes `build` during image construction and ships the resulting `/opt/html` alongside the server binary.

The process enters through [`main`](../../cmd/error-pages/main.go), creates the Cobra tree in [`cli.NewCommand`](../../internal/cli/root.go), and dispatches to the command packages. The HTTP server wires routes in [`Server.Register`](../../internal/http/server.go).

## Runtime boundaries

| Boundary | Responsibility | Evidence |
| --- | --- | --- |
| CLI | Flags, environment precedence, logging, command lifecycle | [`internal/cli`](../../internal/cli/root.go) |
| Config | YAML/envsubst input into an internal model | [`config.go`](../../internal/config/config.go) |
| HTTP | Routing, negotiation, status codes, headers, middleware | [`server.go`](../../internal/http/server.go), [`errorpage.go`](../../internal/http/core/errorpage.go) |
| Rendering | Go-template execution, tokens, optional cache | [`render.go`](../../internal/tpl/render.go), [`properties.go`](../../internal/tpl/properties.go) |
| Assets | HTML templates, JSON/XML format templates, browser localization | [`templates`](../../templates/readme.md), [`error-pages.yml`](../../error-pages.yml), [`l10n`](../../l10n/readme.md) |

## Read next

Start with [[02-cli-and-configuration]] for inputs, then [[03-http-routing-and-request-flow]] for a request trace. Response details are specified in [[04-response-formats-and-error-contracts]].

<details>
<summary>Relevant sources</summary>

- [`README.md`](../../README.md)
- [`cmd/error-pages/main.go`](../../cmd/error-pages/main.go)
- [`internal/cli/root.go`](../../internal/cli/root.go)
- [`internal/http/server.go`](../../internal/http/server.go)
</details>
