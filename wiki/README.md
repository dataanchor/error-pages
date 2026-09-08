# error-pages repository wiki

This technical wiki explains the Go error-page server and generator at ref `6d3ced4` on branch `docs/generate-wiki`.

## Pages

- [01 — Repository overview](pages/01-repository-overview.md)
- [02 — CLI and configuration](pages/02-cli-and-configuration.md)
- [03 — HTTP routing and request flow](pages/03-http-routing-and-request-flow.md)
- [04 — Response formats and error contracts](pages/04-response-formats-and-error-contracts.md)
- [05 — Template rendering and localization](pages/05-template-rendering-and-localization.md)
- [06 — Template selection](pages/06-template-selection.md)
- [07 — Static page generation](pages/07-static-page-generation.md)
- [08 — Operational endpoints and observability](pages/08-operational-endpoints-and-observability.md)
- [09 — Container and release delivery](pages/09-container-and-release-delivery.md)
- [10 — Tests and CI verification](pages/10-tests-and-ci-verification.md)
- [11 — Repository map and extension points](pages/11-repository-map-and-extension-points.md)

## Evidence

- [Inspected sources](sources.md)

The page claims are grounded in source links and the repository state recorded in [manifest.json](manifest.json). Secrets used by CI are intentionally not reproduced.

## Suggested reading paths

- Runtime behavior: [[01-repository-overview]] → [[03-http-routing-and-request-flow]] → [[04-response-formats-and-error-contracts]]
- Customization: [[02-cli-and-configuration]] → [[05-template-rendering-and-localization]] → [[07-static-page-generation]]
- Operations: [[08-operational-endpoints-and-observability]] → [[09-container-and-release-delivery]] → [[10-tests-and-ci-verification]]
