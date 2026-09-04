---
grok_wiki: true
page_id: cli-and-configuration
title: CLI and configuration
repository: error-pages
branch: docs/generate-wiki
ref: 6d3ced4
generated_at: 2026-09-04T07:07:12Z
source_files:
  - internal/cli/root.go
  - internal/cli/serve/command.go
  - internal/cli/serve/flags.go
  - internal/cli/build/command.go
  - internal/cli/healthcheck/command.go
  - internal/cli/version/command.go
  - internal/config/config.go
  - internal/env/env.go
  - internal/options/errorpage.go
  - error-pages.yml
  - schemas/config/1.0.schema.json
---

# CLI and configuration

The root command exposes `serve`, `build <output-directory>`, `healthcheck`, and `version`. Persistent flags configure logging and the config path; command-specific flags are converted into `options.ErrorPage` for the HTTP path.

## Precedence and config loading

For serve flags, an explicitly changed CLI flag wins. Otherwise the matching environment variable is read in [`flags.OverrideUsingEnv`](../../internal/cli/serve/flags.go). The config path follows the same rule in [`root.go`](../../internal/cli/root.go): `--config-file` defaults to `./error-pages.yml`, and `CONFIG_FILE` supplies the value only when the flag was not changed.

[`config.FromYamlFile`](../../internal/config/config.go) reads the file, temporarily changes the working directory to the config file's directory so relative template paths work, expands environment expressions, unmarshals YAML, validates it, and exports the runtime model.

## Config model

The default [`error-pages.yml`](../../error-pages.yml) contains 11 HTML templates, JSON and XML response templates, and status pages from 400 through 505. A template can provide `path` plus optional `name`, or inline `content`; a missing name is derived from the path basename. Pages are keyed by code and carry `message` and `description`.

Validation rejects an empty template list, templates with neither usable path/name nor content, an empty page list, page codes containing spaces, and format names containing spaces. The public schema also limits top-level keys and template/format/page object properties; runtime validation adds the semantic checks.

## Important serve settings

| Setting | Default | Effect |
| --- | --- | --- |
| `listen` / `LISTEN_ADDR` | `0.0.0.0` | Bind address |
| `port` / `LISTEN_PORT` | `8080` | TCP port |
| `template-name` / `TEMPLATE_NAME` | first configured | Fixed or special random selection |
| `default-error-page` / `DEFAULT_ERROR_PAGE` | `404` | Page used at `/` |
| `default-http-code` / `DEFAULT_HTTP_CODE` | `404` | Status returned at `/` |
| `show-details` / `SHOW_DETAILS` | false | Add ingress/request fields to templates and formats |
| `proxy-headers` / `PROXY_HTTP_HEADERS` | empty | Copy selected request headers to responses |
| `disable-l10n` / `DISABLE_L10N` | false | Set the renderer's localization switch |

`serve` validates the listen IP, caps the default status at 599, and rejects spaces in the proxy-header list. `build` has independent `--index` and `--disable-l10n` flags; it does not use serve's runtime picker or request details.

## Read next

[[06-template-selection]] explains `template-name`; [[03-http-routing-and-request-flow]] shows how options reach a request; [[07-static-page-generation]] covers the build command.

<details>
<summary>Relevant sources</summary>

- [`internal/cli/root.go`](../../internal/cli/root.go)
- [`internal/cli/serve/command.go`](../../internal/cli/serve/command.go)
- [`internal/cli/serve/flags.go`](../../internal/cli/serve/flags.go)
- [`internal/cli/build/command.go`](../../internal/cli/build/command.go)
- [`internal/config/config.go`](../../internal/config/config.go)
- [`internal/env/env.go`](../../internal/env/env.go)
- [`error-pages.yml`](../../error-pages.yml)
- [`schemas/config/1.0.schema.json`](../../schemas/config/1.0.schema.json)
</details>
