# Clemo P0 Runtime Foundation — Design

**Date:** 2026-11-07  
**Status:** Proposed implementation design. The user delegated technical choices and asked work to proceed.  
**Current repository state:** documentation-only; no application package, API, database schema or runtime exists.

## Intent
Turn Clemo from a documentation-only concept into a small, testable API foundation without prematurely implementing authentication, payments, wallet logic or external integrations. This is the first vertical slice on which the rest of Clemo can safely build.

The first slice establishes an executable project, predictable local development, typed environment/configuration, a minimal Hono API, liveness/readiness semantics, an OpenAPI-visible route contract, unit tests and CI that actually runs checks.

## Users and outcome
The first users are maintainers and future builders, not public consumers. A developer can clone the repo, install dependencies, run tests, start the API and understand its health state. An operator can distinguish process liveness from dependency readiness.

## Goals
- Add a small Hono + TypeScript API on Cloudflare Workers.
- Establish a strict, typed configuration boundary; invalid required config fails clearly at startup.
- Provide /health/live and /health/ready with stable JSON contracts. Liveness confirms the process can serve; readiness reports required P0 config without exposing secrets.
- Provide a versioned /api/v1 namespace and OpenAPI-visible contract.
- Add Vitest tests for route contracts, malformed config and readiness failures.
- Add reproducible npm lockfile, Node version pin, scripts and CI for install, typecheck, lint, test and build.
- Document commands, environment variables, local-only development and the fact that no production resources are provisioned.

## Non-goals
- No user authentication provider decision or production login.
- No D1 schema, wallet, credits, payments, queue, R2, KV or Durable Object implementation.
- No external tool calls, GitHub App, SSO or cross-product events.
- No production deploy, DNS, secrets, paid service or account provisioning.
- No Rust/Go rewrite or microservice split.

## Approaches considered
1. **Hono + TypeScript on Cloudflare Workers (recommended).** Fits the accepted platform, has low operational overhead, uses Web APIs and keeps API and future bindings in the intended runtime.
2. **Next.js Route Handlers.** Convenient for a tightly coupled full-stack app, but Clemo's API and workspace UI need independent release lifecycles.
3. **Go or Rust service now.** Adds deployment, database connectivity and operational complexity before a workload or benchmark exists. No demonstrated bottleneck justifies it.

Chosen: approach 1. TypeScript is the control-plane language. Python is reserved for later offline evaluation/analytics; Rust/Go require measured workload justification.

## Architecture
- src/app.ts: pure Hono application factory, route composition and common error handling; no side effects on import.
- src/index.ts: Cloudflare Worker entry point; injects environment bindings and delegates to app factory.
- src/config/env.ts: Zod schema and safe parser for environment configuration.
- src/routes/health.ts: liveness/readiness route definitions and response schemas.
- src/contracts/health.ts: versioned health response types/schema.
- test/: Vitest tests for HTTP behavior and configuration.
- wrangler.toml (or supported JSONC equivalent): local Worker configuration with no production identifiers/secrets.
- openapi/: contract artifact or generated OpenAPI document.
- .github/workflows/ci.yml: direct deterministic CI steps rather than a reusable workflow that silently skips checks when no lockfile exists.

Keep handlers pure and dependencies injected. Health response must not leak binding names, secret values, stack traces, provider metadata or internal topology. Readiness returns non-2xx when required runtime config is missing; liveness remains independent of optional remote services.

## Security and reliability
- Strict environment allowlist; reject malformed values.
- No secrets in .dev.vars.example, fixtures or health output.
- Do not use wildcard CORS; CORS is not enabled until exact trusted origins and auth behavior are implemented.
- Use request IDs/correlation IDs without logging bodies or secrets.
- Bound request body size in future mutating routes; P0 has no mutation endpoints.
- Error responses are stable and sanitized; internal exceptions are logged only as redacted structured metadata.
- Health endpoints reveal only service name, API version and status.
- No database is required for P0; readiness must not claim a DB is healthy if none is configured.

## Acceptance criteria
1. npm ci succeeds from a committed lockfile.
2. npm run typecheck, npm run lint, npm test and npm run build run and pass in CI.
3. A test proves /health/live returns the exact stable contract.
4. A test proves readiness returns 503 when required configuration is missing and does not disclose binding names or values.
5. A test proves malformed config fails closed with sanitized errors.
6. Worker entrypoint is compatible with documented Wrangler local dev command.
7. README lists prerequisites, commands, env vars and explicitly says no production deploy/credentials are configured.
8. No paid resource, live secret, production deploy or DNS change is introduced.

## Open implementation decisions delegated to builder
- Select current mutually compatible stable versions of Hono, Zod, Vitest, Wrangler, TypeScript and a minimal lint/format tool after checking official release notes and Node 22 compatibility.
- Prefer one lint/format stack; avoid overlapping tools.
- Choose the simplest OpenAPI integration that produces a verifiable spec without runtime overhead. If disproportionate, begin with typed route schemas and a checked-in contract fixture, documenting the tradeoff.
- Use Node 22 LTS and npm unless current repository policy or official runtime support indicates otherwise.

## Rollback
This is additive. Revert the P0 PR to return to documentation-only state. No external state or data migration is created.