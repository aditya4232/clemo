# Clemo P0 Runtime Foundation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox syntax for tracking.

**Goal:** Establish a small, secure, testable Hono + TypeScript API foundation for Clemo with real CI checks and explicit health contracts.

**Architecture:** A pure Hono app factory is shared by a Cloudflare Worker entry point and Vitest. Typed environment parsing is isolated from handlers. No database, auth provider, payment, queue or production resource is added in P0.

**Tech Stack:** Node.js 22 LTS, npm lockfile, TypeScript, Hono, Zod, Vitest, Wrangler, GitHub Actions.

**Spec:** docs/superpowers/specs/2026-11-07-clemo-p0-runtime-design.md

## Global Constraints
- Keep Clemo independent and preserve product/data boundaries.
- Maintain ₹600/month target and ₹1,000/month hard ceiling; no paid services or production resource provisioning.
- No production deploy, DNS, live credentials, payment or external integration.
- Worker authorization is authoritative; CORS is disabled until exact trusted origins and auth exist.
- Health output must not reveal secrets, binding names, account identifiers, topology or stack traces.
- Node 22 LTS and npm; commit a lockfile and use npm ci.
- No D1/wallet/payment/queue/R2/KV/Durable Object implementation in P0.
- Use test-first behavior changes and run full CI suite before PR merge.

## Review Focus
- Missing or malformed Worker bindings must fail readiness safely without breaking liveness.
- Query strings, unusual HTTP methods and unexpected content types must not produce unhandled errors or leak stack traces.
- Environment values containing whitespace, empty strings or secret-like values must not appear in errors.
- Direct Worker entrypoint and test app factory must use the same route implementation.
- Dependency or lockfile drift must fail CI rather than silently skipping checks.

---

### Task 1: Establish executable project and deterministic toolchain

**Files:** Create package.json, package-lock.json, .nvmrc, tsconfig.json, .gitignore, eslint.config.js, src/app.ts, src/index.ts and test/app.test.ts. Modify .github/workflows/ci.yml and README.md.

**Interfaces:** createApp(env: RuntimeEnv): Hono is exported from src/app.ts. Worker default export in src/index.ts implements fetch(request, env, ctx) and delegates to createApp(env).fetch(...). Scripts: dev, typecheck, lint, test, build and check.

- [ ] Step 1: Write a failing app contract test in test/app.test.ts asserting unknown route returns sanitized JSON 404 and app factory exports a Hono app.
- [ ] Step 2: Run npm test -- --run test/app.test.ts. Expected: FAIL because src/app.ts and package scripts do not exist.
- [ ] Step 3: Add minimal project/toolchain and pure app factory; select current stable compatible dependency versions and commit npm lockfile. No middleware or features beyond the contract.
- [ ] Step 4: Run targeted test and full check. Expected: app contract test PASS; npm run check exits 0.
- [ ] Step 5: Commit chore: bootstrap clemo worker api.

### Task 2: Typed environment configuration

**Files:** Create src/config/env.ts, test/env.test.ts and .dev.vars.example. Modify src/index.ts.

**Interfaces:** RuntimeEnv contains only declared Worker bindings/config. parseRuntimeEnv(input: unknown): RuntimeEnv validates required values and returns no secret-bearing error text.

- [ ] Step 1: Write tests for valid config, missing required public config, empty/whitespace value, unknown field handling and secret redaction.
- [ ] Step 2: Run npm test -- --run test/env.test.ts. Expected: FAIL because parser is absent.
- [ ] Step 3: Implement Zod schema and sanitized parse result. Do not add auth/database secrets in P0.
- [ ] Step 4: Run targeted tests and npm run check. Expected: PASS.
- [ ] Step 5: Commit feat: validate worker runtime configuration.

### Task 3: Health contracts and route behavior

**Files:** Create src/contracts/health.ts, src/routes/health.ts and test/health.test.ts. Modify src/app.ts and src/index.ts.

**Interfaces:** GET /health/live returns 200 {status: ok, service: clemo-api, apiVersion: v1}. GET /health/ready returns 200 {status: ready, service: clemo-api, apiVersion: v1} only when required P0 runtime config is valid; otherwise returns 503 {status: not_ready, service: clemo-api, apiVersion: v1, error: service_not_ready}. Neither route exposes secret values, binding names or internal exceptions.

- [ ] Step 1: Write route tests for liveness, ready success, ready missing config, method mismatch and secret redaction.
- [ ] Step 2: Run npm test -- --run test/health.test.ts. Expected: FAIL because routes are absent.
- [ ] Step 3: Implement route module and register in app factory.
- [ ] Step 4: Run targeted tests and npm run check. Expected: PASS.
- [ ] Step 5: Commit feat: add worker health endpoints.

### Task 4: Cloudflare local configuration and CI gates

**Files:** Create wrangler.toml (or supported JSONC format). Modify .github/workflows/ci.yml, README.md and CONTRIBUTING.md.

**Interfaces:** npm run dev runs Wrangler local development. npm run build validates the Worker bundle. npm run check runs typecheck, lint, test and build. CI runs Node 22, npm ci, then all scripts; permissions remain read-only; no deployment job.

- [ ] Step 1: Add workflow contract validation to assert CI includes lockfile install, typecheck, lint, tests and build, with no deploy permissions or deployment step.
- [ ] Step 2: Run targeted workflow/config test. Expected: FAIL until CI is replaced with explicit steps.
- [ ] Step 3: Configure Wrangler with a local-only Worker name/entrypoint and no production IDs/secrets.
- [ ] Step 4: Replace reusable skip-on-missing-lockfile workflow with explicit CI steps. Use least privilege and pinned action major versions; no deploy credentials.
- [ ] Step 5: Run npm run check, inspect workflow and run GitHub Actions. Expected: all CI checks green; no deploy job exists.
- [ ] Step 6: Commit ci: enforce clemo worker quality gates.

### Task 5: Documentation and handoff

**Files:** Modify README.md, CONTRIBUTING.md, docs/ROADMAP.md, CHANGELOG.md and docs/AI-BUILD-CONTEXT.md.

- [ ] Step 1: Add README smoke-check list for install, local run, health routes and tests.
- [ ] Step 2: Document actual env variables and local-only setup; no placeholders that look like real secrets.
- [ ] Step 3: Update status only for verified pieces; keep auth, wallet and payments marked planned.
- [ ] Step 4: Run full check and inspect diff for secrets, unsupported claims, dependency/license issues and product-boundary drift.
- [ ] Step 5: Commit docs: document clemo p0 runtime foundation.

## Verification
Run from repository root: npm ci; npm run check; npm test -- --run; npm run build. GitHub Actions must run the same checks on pull requests and pushes to main. Report exact runtime versions and results. Green CI validates only the scaffold; it does not certify auth, wallet, payment or production security.

## Rollback
Revert the P0 PR. No persistent production data or external resources are created.