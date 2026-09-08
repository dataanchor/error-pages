---
grok_wiki: true
page_id: template-rendering-and-localization
title: Template rendering and localization
repository: error-pages
branch: docs/generate-wiki
ref: 6d3ced4
generated_at: 2026-09-04T07:07:12Z
source_files:
  - internal/tpl/render.go
  - internal/tpl/properties.go
  - internal/tpl/hasher.go
  - templates/l7-light.html
  - templates/noise.html
  - l10n/readme.md
  - l10n/l10n.js
---

# Template rendering and localization

[`TemplateRenderer`](../../internal/tpl/render.go) parses and executes Go `text/template` content. It supports built-in functions `now`, `hostname`, `json`, `version`, and `int`; renderer-specific functions expose `show_details`, `hide_details`, `l10n_enabled`, and `l10n_disabled`.

## Render properties

[`tpl.Properties`](../../internal/tpl/properties.go) defines tokens for the page code, message, description, and ingress/request metadata. Only fields with a `token` tag become direct template functions; empty values are represented as empty strings. Boolean switches control detail blocks and localization state.

The HTTP responder fills page text from configured pages, and fills request-detail fields from headers only when `ShowDetails` is true. The static builder supplies page text but explicitly disables request details because no request exists.

## Short-lived cache

The renderer hashes the properties and template bytes, combines the two MD5 hashes, and caches rendered bytes for two seconds. A cleanup goroutine scans the cache every second. `Render` returns `ErrClosed` after `Close`; shutdown signals the cleanup goroutine and clears the cache.

This cache is an optimization, not a durable artifact: changing any token value or template content changes the cache key, and entries expire quickly. The hash implementation is in [`hasher.go`](../../internal/tpl/hasher.go).

## Browser localization

The HTML templates keep English strings in markup and mark translatable nodes with `data-l10n`; examples appear in [`l7-light.html`](../../templates/l7-light.html) and [`noise.html`](../../templates/noise.html). [`l10n.js`](../../l10n/l10n.js) exposes an English-keyed table with French, Russian, Ukrainian, Portuguese, and Dutch translations. The page loads this logic from a versioned CDN path described in [`l10n/readme.md`](../../l10n/readme.md).

Localization is client-side and applies to HTML only. JSON/XML bodies are rendered from the configured server-side strings. `DISABLE_L10N` controls the template's localization conditional, allowing a deployment to suppress the browser behavior.

## Extension constraints

Add a new token by extending `Properties` with a tagged string field, then populate it at the boundary that owns the data. Add a template function only when the value is not request/page data. Keep the HTML marker convention aligned with `l10n.js`.

Continue with [[06-template-selection]] for choosing an HTML template and [[07-static-page-generation]] for the non-request path.

<details>
<summary>Relevant sources</summary>

- [`internal/tpl/render.go`](../../internal/tpl/render.go)
- [`internal/tpl/properties.go`](../../internal/tpl/properties.go)
- [`internal/tpl/hasher.go`](../../internal/tpl/hasher.go)
- [`templates/l7-light.html`](../../templates/l7-light.html)
- [`templates/noise.html`](../../templates/noise.html)
- [`l10n/readme.md`](../../l10n/readme.md)
- [`l10n/l10n.js`](../../l10n/l10n.js)
</details>
