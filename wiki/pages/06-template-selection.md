---
grok_wiki: true
page_id: template-selection
title: Template selection
repository: error-pages
branch: docs/generate-wiki
ref: 6d3ced4
generated_at: 2026-09-04T07:07:12Z
source_files:
  - internal/cli/serve/command.go
  - internal/cli/serve/flags.go
  - internal/pick/picker.go
  - internal/config/config.go
---

# Template selection

The `serve` command converts `template-name` into a picker over `Config.TemplateNames()`. The empty value selects the first configured template; named values must exist in the config. The default configuration's first template is `ghost`.

## Selection modes

| Value | Picker mode | Selection lifetime |
| --- | --- | --- |
| empty | `First` | First configured name for every request |
| `random` | `RandomOnce` | One random name for the picker lifetime |
| `i-said-random` | `RandomEveryTime` | Random name on each pick |
| `random-daily` | `RandomEveryTime` + 24h interval | Random name changes at interval |
| `random-hourly` | `RandomEveryTime` + 1h interval | Random name changes at interval |
| any configured name | single-element `First` picker | Fixed name |

The interval variants use the picker implementation with an interval wrapper, so the random decision is reused until the interval elapses. A requested name absent from the config fails command startup before the server is registered.

`pick.NewPicker` protects random state with a mutex and avoids returning the same random index consecutively when another choice is available. The picker is closed during graceful serve shutdown when it implements `Close`.

The build command does not select dynamically: it iterates every configured template and page. See [[07-static-page-generation]].

<details>
<summary>Relevant sources</summary>

- [`internal/cli/serve/command.go`](../../internal/cli/serve/command.go)
- [`internal/cli/serve/flags.go`](../../internal/cli/serve/flags.go)
- [`internal/pick/picker.go`](../../internal/pick/picker.go)
- [`internal/config/config.go`](../../internal/config/config.go)
</details>
