---
grok_wiki: true
page_id: container-and-release-delivery
title: Container and release delivery
repository: error-pages
branch: docs/generate-wiki
ref: 6d3ced4
generated_at: 2026-09-04T07:07:12Z
source_files:
  - Dockerfile
  - docker-compose.yml
  - .github/workflows/release.yml
  - Makefile
---

# Container and release delivery

The project ships both compiled binaries and a minimal Docker image. The image is built in two stages: a Go/Alpine builder compiles the binary and generates static pages, then a `scratch` runtime receives only the prepared rootfs.

## Image layout and defaults

The builder places the binary at `/bin/error-pages`, the config at `/opt/error-pages.yml`, templates at `/opt/templates`, and generated pages at `/opt/html`. The runtime uses UID/GID `10001`, sets `/opt` as its working directory, binds port 8080 by default, selects `ghost`, defaults to page/status 404, and starts `serve --log-json`.

The image defines a healthcheck that runs `/bin/error-pages healthcheck --log-json`. Version metadata is injected at link time through the `internal/version.version` variable. The build uses `CGO_ENABLED=0`, `-trimpath`, and stripped linker flags.

## Release pipeline

The release workflow builds Linux and Darwin amd64 binaries, publishes multi-architecture Docker images for Linux amd64/arm64/armv6/armv7, and deploys the generated `/opt/html` tree to GitHub Pages as the demonstration. It also purges the CDN cache for the localization asset.

CI secrets are referenced by the workflow but are not part of the application configuration and are intentionally omitted from this wiki.

## Local development

[`docker-compose.yml`](../../docker-compose.yml) is explicitly a development example. Its `web` service runs `go run`, exposes port 8080, enables details, and proxies three example headers. The `hurl` service waits for the web healthcheck and runs HTTP integration tests. The [`Makefile`](../../Makefile) wraps image, binary build, formatting, linting, unit tests, integration tests, and compose lifecycle commands.

See [[07-static-page-generation]] for why `/opt/html` exists and [[08-operational-endpoints-and-observability]] for health behavior.

<details>
<summary>Relevant sources</summary>

- [`Dockerfile`](../../Dockerfile)
- [`docker-compose.yml`](../../docker-compose.yml)
- [`release workflow`](../../.github/workflows/release.yml)
- [`Makefile`](../../Makefile)
</details>
