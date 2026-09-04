<!--
Harness knowledge file.
Update when the corresponding architecture or workflow changes.
Do not add speculative information.
-->

# Development

All commands below are confirmed against `Makefile`, `.github/workflows/tests.yml`,
`Dockerfile`, and `docker-compose.yml`.

## Toolchain

- Go 1.18 (`go.mod`). CI builds/tests with Go 1.18; the `golangci-lint` job uses
  Go 1.17 with `golangci-lint v1.44`.
- Optional: Node 16 for `ajv-cli` (config schema) and `eslint@8` (l10n JS).
- Docker + Docker Compose for the containerized workflow (`make` targets).

## With a local Go toolchain

```bash
go build -trimpath -o ./error-pages ./cmd/error-pages/   # build binary
go run ./cmd/error-pages serve --verbose --show-details   # run server on :8080
go test -race ./...                                       # unit tests
go test -race -covermode=atomic -coverprofile cover.txt ./...   # as CI runs it
golangci-lint run                                         # lint (config: .golangci.yml)
gofmt -s -w . && go mod tidy                              # format
```

Version stamping (matches CI/Docker):

```bash
LDFLAGS="-s -w -X github.com/tarampampam/error-pages/internal/version.version=$(git rev-parse HEAD)"
CGO_ENABLED=0 go build -trimpath -ldflags "$LDFLAGS" -o ./error-pages ./cmd/error-pages/
```

## Via Docker Compose (no local Go needed)

```bash
make build      # compile binary inside golang container
make gotest     # go test -v -race -timeout 10s ./...
make lint       # golangci-lint (golangci/golangci-lint:v1.44-alpine)
make fmt        # goimports + gofmt + go mod tidy
make up         # start server -> http://127.0.0.1:8080  (make down to stop)
make int-test   # Hurl integration suite against the compose 'web' service
make test       # lint + gotest + int-test
make image      # docker build -> error-pages:local, prints run hint
```

## CLI subcommands

| Command | Purpose |
| --- | --- |
| `serve` (`s`, `server`) | start the HTTP server |
| `build <out-dir>` (`b`) | render the static HTML tree; `--index` adds `index.html`, `--disable-l10n` |
| `healthcheck` | probe a running instance (used by the Docker `HEALTHCHECK`) |
| `version` | print build version |

Global flags: `-c/--config-file` (`$CONFIG_FILE`, default `./error-pages.yml`),
`-v/--verbose`, `--debug`, `--log-json`.

`serve` flags (each has an env fallback; **CLI flag > env var > default**):
`-l/--listen` (`$LISTEN_ADDR`), `-p/--port` (`$LISTEN_PORT`), `-t/--template-name`
(`$TEMPLATE_NAME`; `random`, `i-said-random`, `random-daily`, `random-hourly`, or
an explicit name), `--default-error-page` (`$DEFAULT_ERROR_PAGE`),
`--default-http-code` (`$DEFAULT_HTTP_CODE`), `--show-details` (`$SHOW_DETAILS`),
`--proxy-headers` (`$PROXY_HTTP_HEADERS`, comma-separated),
`--disable-l10n` (`$DISABLE_L10N`).

## Config & localization checks

```bash
ajv validate --all-errors --verbose -s ./schemas/config/1.0.schema.json -d ./error-pages.y*ml
cd l10n && eslint ./*.js
```

## Integration tests (Hurl)

`test/hurl/*.hurl` assert the external HTTP contract (routing, negotiation,
`X-Code`, proxied headers, `/healthz`, `/metrics`, `/version`). `make int-test`
runs them against the compose `web` service; CI instead builds the Docker image,
starts it with `SHOW_DETAILS=true` and `PROXY_HTTP_HEADERS=X-Foo,Bar,Baz_blah`,
waits for `healthy`, then runs the same suite.

## CI gates (`tests.yml`)

gitleaks · golangci-lint · `ajv` config validation · l10n ESLint ·
`go test -race` + Codecov · linux/darwin amd64 build + `version`/`-h` smoke ·
generator run with `--index` and per-template file existence checks ·
Docker build + Trivy scan (fails on MEDIUM+) + container start + Hurl suite.
Markdown-only changes skip the build/test jobs.
