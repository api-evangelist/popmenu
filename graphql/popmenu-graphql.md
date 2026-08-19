---
generated: '2026-08-13'
method: probed
source: https://api.popmenu.com/graphql
---

# Popmenu GraphQL API

Popmenu's public-facing machine interface is **GraphQL**, not REST. No OpenAPI, Swagger,
AsyncAPI or `.proto` document was found on any Popmenu host (see the probe table below).
The endpoint below was verified live on 2026-08-13; its **schema is authentication-gated**,
so no SDL is captured here and none has been invented.

## Endpoints (verified live)

| Endpoint | Method | HTTP | Content-Type | Notes |
|---|---|---|---|---|
| `https://api.popmenu.com/graphql` | POST | 200 | `application/json; charset=utf-8` | Answers every POST; introspection rejected as `unauthorized` |
| `https://api.popmenu.com/graphql` | GET | 301 | — | Redirects to `https://api.popmenu.com/` (404) |
| `https://my.popmenu.com/graphql` | POST | 200 | `application/json; charset=utf-8` | Same service, same response envelope and `popmenu-version` build SHA |

Both hosts resolve into the same Cloudflare-fronted Popmenu origin, emit the vendor
`popmenu-version` build header, and set `__cf_bm` with `Domain=popmenu.com`.
`https://my.popmenu.com/robots.txt` (HTTP 200) explicitly names `/graphql` in its
`Allow:` list for `facebookexternalhit`, confirming the path is Popmenu's own.

## Introspection is gated

```
POST https://api.popmenu.com/graphql
Content-Type: application/json

{"query":"{__schema{queryType{name} mutationType{name}}}"}
```

```json
{"errors":[{"friendlyMessage":"Sorry, you are not authorized to perform that action- try signing back in","message":"unauthorized"}]}
```

The same `unauthorized` envelope is returned for `{__typename}` and for a query naming a
field that does not exist — i.e. **the service authenticates before it validates**, so an
unauthenticated caller cannot enumerate types, fields, or even distinguish a malformed
query from an unauthorized one. Schema capture requires credentials issued through
Popmenu's partner channel.

## How access is obtained

`https://get.popmenu.com/developer-api` is a **lead-capture form**, not documentation. Its
copy reads: *"By integrating with our API, you'll gain access to a wealth of data on menus,
guests, orders, and much more"* and submission lands on `/success-api` — *"Thank you for
your interest in working with Popmenu! We'll be reaching out to you soon."* There is no
published schema, base URL, authentication reference, scope list, or rate-limit table on
any Popmenu property.

## Observed runtime characteristics

- **Rate limiting** — vendor-prefixed headers on every response:
  `popmenu-ratelimit-limit`, `popmenu-ratelimit-remaining`, `popmenu-ratelimit-reset`
  (see `rate-limits/popmenu-rate-limits.yml`).
- **Tracing** — `x-request-id` (UUID) and `x-runtime` on every response.
- **Build identity** — `popmenu-version` carries a git SHA; there is no API version segment
  in the path and no `Accept`-header version negotiation.
- **Stack** — Rails + React SSR (`RAILS_ENV`, `APP: core-admin-web`, `GQL_OPS: TRUE`,
  `PREFETCHED_QUERIES_NUMBER` in the `my.popmenu.com` page footer), consistent with
  Popmenu's public engineering description.

## Discovery probes that missed (2026-08-13)

| URL | HTTP |
|---|---|
| `https://api.popmenu.com/openapi.json` | 404 |
| `https://api.popmenu.com/openapi.yaml` | 404 |
| `https://api.popmenu.com/swagger.json` | 404 |
| `https://api.popmenu.com/v1/openapi.json` | 404 |
| `https://api.popmenu.com/api-docs` | 404 |
| `https://api.popmenu.com/docs` | 404 |
| `https://api.popmenu.com/redoc` | 404 |
| `https://api.popmenu.com/graphiql` | 404 |
| `https://api.popmenu.com/schema.graphql` | 404 |
| `https://api.popmenu.com/mcp` | 404 |
| `https://my.popmenu.com/openapi.json` | 404 |
| `https://get.popmenu.com/*` (all paths) | 403 — Cloudflare bot challenge |

Marketing hosts (`get.popmenu.com`, `popmenu.com`, `www.popmenu.com`,
`support.popmenu.com`, `developer.popmenu.com`, `docs.popmenu.com`, `trust.popmenu.com`)
return **403 to every non-browser client**, including `robots.txt`. Their content was read
where possible via the Internet Archive rather than asserted from absence.
