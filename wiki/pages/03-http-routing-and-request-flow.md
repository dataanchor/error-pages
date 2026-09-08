---
grok_wiki: true
page_id: http-routing-and-request-flow
title: HTTP routing and request flow
repository: error-pages
branch: docs/generate-wiki
ref: 6d3ced4
generated_at: 2026-09-04T07:07:12Z
source_files:
  - internal/http/server.go
  - internal/http/common/middlewares.go
  - internal/http/handlers/index/handler.go
  - internal/http/handlers/errorpage/handler.go
  - internal/http/handlers/notfound/handler.go
  - internal/http/core/errorpage.go
  - internal/http/core/headers.go
---

# HTTP routing and request flow

[`Server.Register`](../../internal/http/server.go) creates a private Prometheus registry, wraps the router with request logging and duration metrics, and registers the complete route set:

| Route | Method | Behavior |
| --- | --- | --- |
| `/` | GET | Default configured page, or valid `X-Code` override |
| `/{code}.html` | GET | Requested page code; response status is normally 200 |
| `/version` | GET | Cached JSON version |
| `/healthz` | ANY | Liveness check |
| `/health/live` | ANY | Deprecated alias for liveness |
| `/metrics` | GET | Prometheus exposition |
| other paths | router fallback | Plain-text 404 guidance |

## Shared error response path

The index handler starts with configured page/status defaults and accepts `X-Code` only when it is 1–3 characters, parses as an integer, and falls between 1 and 599. The `/{code}.html` handler extracts the router's `code` user value and deliberately passes HTTP 200 to the shared responder; the page code is content, not the transport status for that endpoint.

Both handlers call [`core.RespondWithErrorPage`](../../internal/http/core/errorpage.go), which sets `X-Robots-Tag: noindex`, builds render properties, copies configured proxy headers, negotiates a format, renders, and writes the response. Unknown paths never enter this renderer.

## Middleware and shutdown

The outer duration middleware records every request in the registered counter and histogram. The logging middleware skips user agents containing `healthcheck` (case-insensitive), otherwise logs request/response metadata after the handler returns. `serve` listens for OS signals, cancels the context, closes the template picker when supported, closes the renderer, and shuts down fasthttp.

## Edge cases

- A configured or fallback page code with no message returns 404 text saying the code is unavailable.
- A missing selected template or render failure returns 500 text from the shared responder.
- `/not-found` returns a plain-text 404 explaining the `/{code}.html` URL shape.
- `X-Code` affects `/` only; the error-page route ignores it, as covered by [`x_code.hurl`](../../test/hurl/x_code.hurl).

Read [[04-response-formats-and-error-contracts]] for negotiation and [[08-operational-endpoints-and-observability]] for non-error routes.

<details>
<summary>Relevant sources</summary>

- [`internal/http/server.go`](../../internal/http/server.go)
- [`internal/http/common/middlewares.go`](../../internal/http/common/middlewares.go)
- [`internal/http/handlers/index/handler.go`](../../internal/http/handlers/index/handler.go)
- [`internal/http/handlers/errorpage/handler.go`](../../internal/http/handlers/errorpage/handler.go)
- [`internal/http/handlers/notfound/handler.go`](../../internal/http/handlers/notfound/handler.go)
- [`internal/http/core/errorpage.go`](../../internal/http/core/errorpage.go)
- [`internal/http/core/headers.go`](../../internal/http/core/headers.go)
</details>
