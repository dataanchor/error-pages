# Inspected sources

All paths below are repository-relative. Links point from this file in `wiki/` to the repository root.

## Application entrypoints and configuration

- [cmd/error-pages/main.go](../cmd/error-pages/main.go) — process entrypoint and exit behavior.
- [internal/cli/root.go](../internal/cli/root.go) — Cobra root command, persistent flags, logger setup, and config-file environment override.
- [internal/cli/serve/command.go](../internal/cli/serve/command.go) — server command lifecycle, template picker selection, startup, and graceful shutdown.
- [internal/cli/serve/flags.go](../internal/cli/serve/flags.go) — serve flags, environment precedence, validation, and option conversion.
- [internal/cli/build/command.go](../internal/cli/build/command.go) — static output generation.
- [internal/cli/build/history.go](../internal/cli/build/history.go) — generated index data and HTML index rendering.
- [internal/cli/healthcheck/command.go](../internal/cli/healthcheck/command.go) — CLI health probe.
- [internal/cli/version/command.go](../internal/cli/version/command.go) — version command.
- [internal/config/config.go](../internal/config/config.go) — YAML parsing, environment substitution, validation, path resolution, and exported config model.
- [internal/env/env.go](../internal/env/env.go) — supported environment variable names.
- [internal/options/errorpage.go](../internal/options/errorpage.go) — runtime error-page options.
- [error-pages.yml](../error-pages.yml) — default templates, JSON/XML formats, and configured status pages.
- [schemas/config/1.0.schema.json](../schemas/config/1.0.schema.json) — public configuration schema.

## HTTP runtime

- [internal/http/server.go](../internal/http/server.go) — fasthttp server construction, routes, middleware, and shutdown.
- [internal/http/common/middlewares.go](../internal/http/common/middlewares.go) — request logging and duration metrics middleware.
- [internal/http/core/errorpage.go](../internal/http/core/errorpage.go) — shared response construction and rendering path.
- [internal/http/core/formats.go](../internal/http/core/formats.go) — `Content-Type`/`X-Format` parsing and response content types.
- [internal/http/core/headers.go](../internal/http/core/headers.go) — request header constants.
- [internal/http/handlers/index/handler.go](../internal/http/handlers/index/handler.go) — default and `X-Code`-driven responses.
- [internal/http/handlers/errorpage/handler.go](../internal/http/handlers/errorpage/handler.go) — `/{code}.html` handler.
- [internal/http/handlers/notfound/handler.go](../internal/http/handlers/notfound/handler.go) — unmatched-route response.
- [internal/http/handlers/healthz/handler.go](../internal/http/handlers/healthz/handler.go) — liveness endpoint.
- [internal/http/handlers/metrics/handler.go](../internal/http/handlers/metrics/handler.go) — Prometheus adapter.
- [internal/http/handlers/version/handler.go](../internal/http/handlers/version/handler.go) — JSON version endpoint.
- [internal/checkers/live.go](../internal/checkers/live.go) — always-success liveness checker.
- [internal/checkers/health.go](../internal/checkers/health.go) — loopback health checker used by the CLI and container.
- [internal/metrics/metrics.go](../internal/metrics/metrics.go) — custom request counter and duration histogram.
- [internal/metrics/registry.go](../internal/metrics/registry.go) — isolated Prometheus registry and process collector.

## Rendering and assets

- [internal/tpl/render.go](../internal/tpl/render.go) — Go-template functions, short-lived render cache, and renderer shutdown.
- [internal/tpl/properties.go](../internal/tpl/properties.go) — render tokens and request-detail fields.
- [internal/tpl/hasher.go](../internal/tpl/hasher.go) — render cache hashes.
- [internal/pick/picker.go](../internal/pick/picker.go) — first, random-once, and random-every-time selection modes.
- [templates/readme.md](../templates/readme.md) — template asset guidance.
- [templates/ghost.html](../templates/ghost.html) — default template referenced by the container.
- [templates/l7-light.html](../templates/l7-light.html) — light/dark responsive template.
- [templates/l7-dark.html](../templates/l7-dark.html) — dark template.
- [templates/shuffle.html](../templates/shuffle.html) — shuffle design.
- [templates/noise.html](../templates/noise.html) — canvas/noise design.
- [templates/hacker-terminal.html](../templates/hacker-terminal.html) — terminal design.
- [templates/cats.html](../templates/cats.html) — cats design.
- [templates/lost-in-space.html](../templates/lost-in-space.html) — space design.
- [templates/app-down.html](../templates/app-down.html) — app-down design.
- [templates/connection.html](../templates/connection.html) — connection design.
- [templates/matrix.html](../templates/matrix.html) — matrix design.
- [l10n/readme.md](../l10n/readme.md) — localization workflow.
- [l10n/l10n.js](../l10n/l10n.js) — browser-side locale table and translation behavior.

## Delivery and verification

- [Dockerfile](../Dockerfile) — multi-stage image build, static generation, scratch runtime, user, defaults, and healthcheck.
- [docker-compose.yml](../docker-compose.yml) — local development services.
- [Makefile](../Makefile) — build, lint, test, integration-test, and container commands.
- [.github/workflows/tests.yml](../.github/workflows/tests.yml) — CI lint, unit, schema, build, generation, image, scan, and HTTP checks.
- [.github/workflows/release.yml](../.github/workflows/release.yml) — release binaries, multi-platform images, and demo deployment.
- [test/hurl/404.hurl](../test/hurl/404.hurl) — unmatched route behavior.
- [test/hurl/index.hurl](../test/hurl/index.hurl) — index HTML/JSON/XML behavior.
- [test/hurl/x_code.hurl](../test/hurl/x_code.hurl) — ingress status-code override behavior.
- [test/hurl/code_502_default.hurl](../test/hurl/code_502_default.hurl) — HTML error-page contract.
- [test/hurl/code_502_json.hurl](../test/hurl/code_502_json.hurl) — JSON negotiation, details, and proxy inputs.
- [test/hurl/code_502_xml.hurl](../test/hurl/code_502_xml.hurl) — XML negotiation and details.
- [test/hurl/proxy_headers.hurl](../test/hurl/proxy_headers.hurl) — selected request-header forwarding.
- [test/hurl/healthz.hurl](../test/hurl/healthz.hurl) — current and deprecated health routes.
- [test/hurl/metrics.hurl](../test/hurl/metrics.hurl) — metrics endpoint contract.
- [test/hurl/version.hurl](../test/hurl/version.hurl) — version endpoint contract.
- [CHANGELOG.md](../CHANGELOG.md) — release history.
