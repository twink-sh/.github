# twink.sh

> EARLY planning & PoC phase. Do NOT use.

Infrastructure for self-serve domains and routing. The goal is to make user-scoped subdomains, DNS, HTTP(S) routing, and small utilities like URL shortening programmable from TypeScript.

This org is early stage and under active development. Expect changes. (and nothing to work - yet)

---

## Repositories

- twink-sh/twink
  TypeScript SDK and control plane for the platform. Manages:
  - DNS record CRUD against Cloudflare
  - Delegation and dynamic DNS via an authoritative server backed by Redis
  - HTTP routes in Caddy for reverse proxy, rewrites, and forwards
  - Multi-tenant, user-scoped operations and reconciliation between data stores and providers  
  Stack highlights: TypeScript, Cloudflare API, Redis, Express, http-proxy-middleware, Supabase client, Bun for tooling.

- twink-sh/twinkui
  The user-facing webapp that puts a pretty face on the APIs :D

- twink-sh/url-shortener-testing
  Experimental URL shortener playground. Monorepo with Next.js apps for API, web, and docs. Uses React 19, Tailwind, Turborepo, Zod, and Bruno collections for HTTP tests.

- twink-sh/ServerConfig
  Will contain various configs etc for deploying TWINK

Primary language: TypeScript. - Though some of our "borrowed" modules use other programming languages - And rewriting the URL Shortener handler in Go is something im considering

---

## Architecture overview

The system separates a control plane (provisioning, policy, reconciliation) from data planes that actually answer DNS and HTTP traffic.

- Identity and RBAC  
  - Supabase Postgres stores users and roles  
  - better-auth handles authentication  
  - Read replica is used for failover protection

- Control plane  
  - The `twink` package is the orchestrator  
  - Validates permissions, writes desired state to stores, and reconciles providers like Cloudflare and Caddy

- DNS plane  
  - Cloudflare owns the apex zone and delegates subdomains to an authoritative server  
  - An authoritative DNS server (dinodns) reads records from Redis using a Redis store plugin  
  - Persistent Redis holds DNS records, with replication for durability  
  - Inbound DNS queries are answered by dinodns

- HTTP routing plane  
  - Caddy terminates and routes traffic  
  - `twink` programs Caddy routes for reverse proxy, rewrites, and forwards after passing auth checks

- URL shortener plane  
  - Next.js middleware handles request rewriting and metadata injection  
  - Redis JSON holds short link definitions  
  - Supabase Postgres stores analytics  
  - Auth and RBAC gate creation and reads of analytics

- State sync and reconciliation  
  - `twink` acts as the single source of truth and sync layer  
  - Periodic sidecars and on-demand operations reconcile Supabase, Redis, Cloudflare, and Caddy

---

## Using the SDK

Coming soon i promise xd

---

## Development notes

- TypeScript (almost) everywhere (wether you like it or not), strict mode
- ESLint and Prettier for linting & consistent code-style.
- Next.js 15 and React 19 in the web repos
- Multiple Redis instances are required. Some with JSON functionality too.
- Postgres (with or without) Supabase for identity and analytics
- DinoDNS handles DNS lookup requests.
- Caddy runs the HTTP data plane. Cloudflare runs the apex DNS zone

---

## Status and scope

- VERY VERY early planning & PoC development. do NOT use.
- Mainly a hobby project of mine, but we'll see where it goes lmao

---

## Contributing

Issues and pull requests are welcome. Keep changes focused, covered by linting and type checks, and provide minimal repro steps. For new modules or providers, include a reconciliation plan and failure modes.

---

## Security

If you find a security issue, please contact the maintainers privately first. Do not open a public issue until coordinated.

---

## License

See individual repositories for license terms. **but** we strive to keep everything as opensource as we can.

```
