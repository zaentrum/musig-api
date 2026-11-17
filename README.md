# musig-api

Backend REST API (Go BFF) for the **musig** music product of the stube
platform. Serves the `musig` web/app clients: OIDC-guarded product endpoints
plus a stub catalog surface pending the catalog data model.

## Status

**Early scaffold.** The chi router, slog logging, OIDC bearer-JWT verifier
middleware, and an embedded OpenAPI 3 spec are in place. The HTTP server
compiles and serves `/api/healthz` and `/api/openapi.yaml`; the `/api/v1/*`
handlers return stub data until the catalog service is online.

## Stack

- Go 1.23 + [chi](https://github.com/go-chi/chi) router + `log/slog`
- OIDC JWT validation via [`go-oidc`](https://github.com/coreos/go-oidc)
- Distroless container image (multi-stage build)

## Endpoints (scaffold)

| Path | Auth | Notes |
|---|---|---|
| `GET /api/healthz` | none | liveness / readiness |
| `GET /api/openapi.yaml` | none | OpenAPI 3 spec for codegen |
| `GET /api/v1/me` | bearer JWT | echoes the caller's `sub` |
| `GET /api/v1/items` | bearer JWT | stub list (replaced once the catalog is online) |

## Local development

```bash
go run ./cmd/server
# in another shell
curl -sS http://localhost:8080/api/healthz
```

Disable OIDC for local poking:

```bash
OIDC_ENABLED=false go run ./cmd/server
curl -sS http://localhost:8080/api/v1/items
```

### Configuration

All wiring is via environment variables (see `internal/config`):

| Variable | Default | Purpose |
|---|---|---|
| `ADDR` | `:8080` | listen address |
| `OIDC_ISSUER` | — | OIDC issuer URL |
| `OIDC_AUDIENCE` | `musig-web` | the audience the tokens must carry: set it to the client id your identity provider issues them for |
| `OIDC_ENABLED` | `true` | set `false` to bypass auth locally |

## Container image

```bash
docker build -t musig-api .
# use a private base-image mirror instead of Docker Hub:
docker build --build-arg BASE=registry.example/library/ -t musig-api .
```

CI publishes `ghcr.io/zaentrum/musig-api` on push to `main`.

## Layout

```
cmd/server/main.go                  process entry
internal/config/                    env wiring
internal/http/router.go             chi router + middleware
internal/http/openapi.{go,yaml}     embedded OpenAPI spec
internal/http/sample.go             stub data
internal/auth/oidc.go               bearer JWT verifier middleware
k8s/                                deployment manifests (example)
Dockerfile                          distroless multi-stage build
```

## License

[MPL-2.0](./LICENSE)
