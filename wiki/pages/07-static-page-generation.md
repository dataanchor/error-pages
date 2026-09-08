---
grok_wiki: true
page_id: static-page-generation
title: Static page generation
repository: error-pages
branch: docs/generate-wiki
ref: 6d3ced4
generated_at: 2026-09-04T07:07:12Z
source_files:
  - internal/cli/build/command.go
  - internal/cli/build/history.go
  - internal/tpl/render.go
  - Dockerfile
  - .github/workflows/tests.yml
---

# Static page generation

`build <output-directory>` materializes the configured page catalog for every configured HTML template. It is the same renderer used by the HTTP service, but it passes `ShowRequestDetails: false` and has no request headers to read.

## Output algorithm

1. Load and validate the YAML config.
2. Reject a config with no templates and create the output directory.
3. For each template and each configured page, render the template with page code, message, description, and the localization switch.
4. Write `<output>/<template-name>/<code>.html` with file mode `0664` and directory mode `0775`.
5. If `--index` is set, sort each template's generated page entries by code and write `<output>/index.html` from the embedded index template.

The builder does not generate JSON/XML files; those formats are runtime response templates. It also does not clean unrelated files already present in the output directory.

## Docker use

The Docker builder copies the binary, templates, and config into a temporary rootfs, then runs `error-pages --config-file ./error-pages.yml build ./html --verbose --index`. The final scratch image therefore contains pre-rendered files under `/opt/html` for extraction, in addition to the runtime server under `/bin/error-pages`.

CI verifies representative generated paths such as `out/index.html`, `out/ghost/404.html`, and the other configured template directories after invoking `build --index`.

## Operational implication

Static generation is deterministic with respect to config, template order/content, and the renderer's runtime values. Templates can still call dynamic functions such as `now`, `hostname`, or `version`; the source supports those functions, so generated output should be treated as a build artifact rather than a timeless fixture.

See [[02-cli-and-configuration]] for input semantics and [[09-container-and-release-delivery]] for how the image packages the output.

<details>
<summary>Relevant sources</summary>

- [`internal/cli/build/command.go`](../../internal/cli/build/command.go)
- [`internal/cli/build/history.go`](../../internal/cli/build/history.go)
- [`internal/tpl/render.go`](../../internal/tpl/render.go)
- [`Dockerfile`](../../Dockerfile)
- [`tests workflow`](../../.github/workflows/tests.yml)
</details>
