---
grok_wiki: true
page_id: response-formats-and-error-contracts
title: Response formats and error contracts
repository: error-pages
branch: docs/generate-wiki
ref: 6d3ced4
generated_at: 2026-09-04T07:07:12Z
source_files:
  - internal/http/core/errorpage.go
  - internal/http/core/formats.go
  - internal/http/core/headers.go
  - error-pages.yml
  - test/hurl/index.hurl
  - test/hurl/x_code.hurl
  - test/hurl/code_502_default.hurl
  - test/hurl/code_502_json.hurl
  - test/hurl/code_502_xml.hurl
  - test/hurl/proxy_headers.hurl
---

# Response formats and error contracts

The server supports HTML by default and optional JSON/XML formats from the `formats` section of [`error-pages.yml`](../../error-pages.yml). Every rendered error response sets `X-Robots-Tag: noindex` and uses the selected transport status supplied by its handler.

## Format selection

[`ClientWantFormat`](../../internal/http/core/formats.go) first examines `Content-Type`, accepting values containing `application/json`/`text/json`, `application/xml`/`text/xml`, `text/html`, or `text/plain`. If no usable content type exists, it parses `X-Format` as a comma-separated preference list, reads `;q=` weights, and chooses the highest-weight item. Unknown or absent input falls back to HTML.

The response content types are normalized to `application/json; charset=utf-8`, `application/xml; charset=utf-8`, or `text/html; charset=utf-8`. Plain-text errors use `text/plain; charset=utf-8`.

## Body contracts

The default JSON template emits `error`, string `code`, `message`, and `description`. With details enabled it adds `host`, `original_uri`, `forwarded_for`, `namespace`, `ingress_name`, `service_name`, `service_port`, `request_id`, and a Unix timestamp. The XML template exposes the same concepts with XML element names such as `originalURI` and `requestID`.

The HTML templates receive the same base tokens and may choose their own markup. They carry `data-l10n` markers for browser-side translation and render request details only when enabled and populated.

## Proxy headers

`PROXY_HTTP_HEADERS`/`--proxy-headers` is parsed into a trimmed, deduplicated, sorted list. For each listed name, the responder copies a non-empty request value to the response. The allowlist is opt-in; arbitrary request headers are not mirrored. [`proxy_headers.hurl`](../../test/hurl/proxy_headers.hurl) verifies that configured headers pass through and an unconfigured header does not.

## Status behavior

`/` can return the configured default HTTP code or a valid `X-Code` value. `/{code}.html` always uses 200 on successful rendering, while the code appears in the body. JSON/XML negotiation and details are exercised by [`code_502_json.hurl`](../../test/hurl/code_502_json.hurl) and [`code_502_xml.hurl`](../../test/hurl/code_502_xml.hurl).

See [[03-http-routing-and-request-flow]] for handler selection and [[05-template-rendering-and-localization]] for token execution.

<details>
<summary>Relevant sources</summary>

- [`internal/http/core/errorpage.go`](../../internal/http/core/errorpage.go)
- [`internal/http/core/formats.go`](../../internal/http/core/formats.go)
- [`internal/http/core/headers.go`](../../internal/http/core/headers.go)
- [`error-pages.yml`](../../error-pages.yml)
- [`test/hurl/index.hurl`](../../test/hurl/index.hurl)
- [`test/hurl/x_code.hurl`](../../test/hurl/x_code.hurl)
- [`test/hurl/code_502_default.hurl`](../../test/hurl/code_502_default.hurl)
- [`test/hurl/code_502_json.hurl`](../../test/hurl/code_502_json.hurl)
- [`test/hurl/code_502_xml.hurl`](../../test/hurl/code_502_xml.hurl)
- [`test/hurl/proxy_headers.hurl`](../../test/hurl/proxy_headers.hurl)
</details>
