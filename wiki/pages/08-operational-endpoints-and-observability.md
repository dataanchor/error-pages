---
grok_wiki: true
page_id: operational-endpoints-and-observability
title: Operational endpoints and observability
repository: error-pages
branch: docs/generate-wiki
ref: 6d3ced4
generated_at: 2026-09-04T07:07:12Z
source_files:
  - internal/http/server.go
  - internal/http/common/middlewares.go
  - internal/http/handlers/healthz/handler.go
  - internal/http/handlers/metrics/handler.go
  - internal/http/handlers/version/handler.go
  - internal/checkers/live.go
  - internal/checkers/health.go
  - internal/metrics/metrics.go
  - internal/metrics/registry.go
  - test/hurl/healthz.hurl
  - test/hurl/metrics.hurl
  - test/hurl/version.hurl
---

# Operational endpoints and observability

The service provides liveness, self-description, and Prometheus endpoints, and wraps all routed requests in logging and request-duration instrumentation.

## Endpoints

| Endpoint | Contract |
| --- | --- |
| `/healthz` | `200 OK` with body `OK` while the live checker succeeds; `503` with the checker error otherwise |
| `/health/live` | Same handler and behavior; marked deprecated in route registration |
| `/version` | `200`, `application/json`, body `{"version":"..."}`; bytes are cached after first request |
| `/metrics` | Prometheus exposition from the server's private registry |

The current live checker always returns success. The CLI `healthcheck` is a separate active probe: it performs a loopback GET to `/healthz` with a three-second HTTP client timeout and fails unless the status is 200. The Docker `HEALTHCHECK` invokes that CLI command.

## Metrics

`DurationMetrics` measures the entire wrapped handler execution, increments `http_requests_total_count`, and observes request duration in `http_requests_duration_milliseconds` using seconds as the Prometheus observation unit despite the metric name. The registry adds a process collector and is created per server registration rather than using the global default registry.

## Logs and health traffic

`LogRequest` captures user agent, method, URL, referer, status, content type, connection-close state, duration, and request/response headers. It skips logging when the user agent contains `healthcheck`, keeping recurring probe traffic out of normal request logs. The root command supports plain, verbose, debug, and JSON logger modes.

The endpoint contracts are covered by [`healthz.hurl`](../../test/hurl/healthz.hurl), [`metrics.hurl`](../../test/hurl/metrics.hurl), and [`version.hurl`](../../test/hurl/version.hurl).

Read [[09-container-and-release-delivery]] for the container healthcheck and [[10-tests-and-ci-verification]] for the verification pipeline.

<details>
<summary>Relevant sources</summary>

- [`internal/http/server.go`](../../internal/http/server.go)
- [`internal/http/common/middlewares.go`](../../internal/http/common/middlewares.go)
- [`internal/http/handlers/healthz/handler.go`](../../internal/http/handlers/healthz/handler.go)
- [`internal/http/handlers/metrics/handler.go`](../../internal/http/handlers/metrics/handler.go)
- [`internal/http/handlers/version/handler.go`](../../internal/http/handlers/version/handler.go)
- [`internal/checkers/health.go`](../../internal/checkers/health.go)
- [`internal/metrics/metrics.go`](../../internal/metrics/metrics.go)
- [`internal/metrics/registry.go`](../../internal/metrics/registry.go)
- [`test/hurl/healthz.hurl`](../../test/hurl/healthz.hurl)
- [`test/hurl/metrics.hurl`](../../test/hurl/metrics.hurl)
- [`test/hurl/version.hurl`](../../test/hurl/version.hurl)
</details>
