---
grok_wiki: true
page_id: tests-and-ci-verification
title: Tests and CI verification
repository: error-pages
branch: docs/generate-wiki
ref: 6d3ced4
generated_at: 2026-09-04T07:07:12Z
source_files:
  - .github/workflows/tests.yml
  - Makefile
  - test/hurl/404.hurl
  - test/hurl/index.hurl
  - test/hurl/x_code.hurl
  - test/hurl/code_502_json.hurl
  - test/hurl/code_502_xml.hurl
  - internal/config/config_test.go
  - internal/http/server_test.go
---

# Tests and CI verification

Verification is split between Go unit tests, static/config linting, generated-artifact checks, container scanning, and Hurl HTTP contracts.

## CI stages

The tests workflow runs:

1. Gitleaks, golangci-lint, configuration schema validation, and ESLint for localization JavaScript.
2. Go tests with race detection and coverage.
3. Linux/Darwin amd64 builds, followed by a version/help smoke test on Linux.
4. The page generator with `--index`, then checks representative files for every configured template.
5. A Docker build, vulnerability scan, container startup, health wait, and Hurl suite.

The workflow ignores Markdown-only changes for ordinary CI triggers, but the wiki is still evidence-backed documentation generated from the source tree.

## HTTP contract coverage

Hurl verifies unmatched routes, default and `X-Code` status behavior, HTML page content, JSON/XML negotiation, request details, proxy-header allowlisting, liveness, metrics, and version output. The JSON and XML tests also exercise `X-Format` as the alternate negotiation header.

## Unit-test shape

The repository has focused table-driven tests around configuration, environment handling, CLI commands, format parsing, rendering, picker behavior, metrics, versioning, and server behavior. Some handler packages contain explicit skipped placeholder tests, so the strongest end-to-end guarantees for those routes currently come from the Hurl suite and server-level tests.

## Reproduction commands

The Makefile exposes `make lint`, `make gotest`, `make int-test`, and `make test`. The integration suite expects the compose `web` service; CI instead builds and runs the image directly before invoking Hurl.

Read [[04-response-formats-and-error-contracts]] to map assertions to implementation and [[09-container-and-release-delivery]] for image checks.

<details>
<summary>Relevant sources</summary>

- [`tests workflow`](../../.github/workflows/tests.yml)
- [`Makefile`](../../Makefile)
- [`test/hurl/404.hurl`](../../test/hurl/404.hurl)
- [`test/hurl/index.hurl`](../../test/hurl/index.hurl)
- [`test/hurl/x_code.hurl`](../../test/hurl/x_code.hurl)
- [`test/hurl/code_502_json.hurl`](../../test/hurl/code_502_json.hurl)
- [`test/hurl/code_502_xml.hurl`](../../test/hurl/code_502_xml.hurl)
- [`internal/config/config_test.go`](../../internal/config/config_test.go)
- [`internal/http/server_test.go`](../../internal/http/server_test.go)
</details>
