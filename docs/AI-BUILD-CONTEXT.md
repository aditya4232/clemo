# Clemo — AI Build Context

**Last reviewed:** 2026-11-07  
**Truth status:** architecture/design phase. No executable product runtime, authentication, database, wallet, payment adapter or deployment is currently present. Verify the repository before every implementation session.

## Mission and user promise
Clemo is the calm, trustworthy workspace and front door for independent tools in the Aditya product ecosystem. Users discover tools, understand capabilities/data/cost, connect only what they choose, launch work and see status, artifacts, credits and support. It is a shell/control plane, not a monolith.

## Audiences and UX principles
Primary users: consumers and developers using a small ecosystem of AI/productivity tools. Prioritize clear language, mobile-first workflows, transparent credit pricing, understandable permissions, progress/cancel/retry, recoverable errors and human support. Keep advanced controls progressive; don't force users to understand queues, manifests or model internals.

## Product boundaries
- Clemo owns workspace identity/membership, catalog, launch/connection registry, wallet/ledger, orders/subscriptions, support/admin/audit and minimal cross-product activity.
- Agent Workspace owns conversations, model/provider selection, BYOK secrets, MCP grants, run state and artifacts.
- Agent Labs owns research, datasets, experiment execution and evidence.
- Portfolio owns public editorial content and curated public product metadata.
- Never share databases or secrets across products. Events are minimal, signed notifications, not authorization.
- No parent-domain shared session cookies. Use independent login until secure one-time launch/OIDC is implemented.

## Current architecture decision
Initial control plane: TypeScript + Hono on Cloudflare Workers. Use Zod at trust boundaries, OpenAPI for public/internal API contracts, Drizzle only where it provides maintainable D1 migrations/querying. D1 is candidate for transactional relational data; use explicit transactions/unique constraints and test concurrent wallet operations. Durable Objects only for narrow serialization needs after proving D1 transaction semantics insufficient. Queues for asynchronous jobs, with idempotent consumers and DLQ. KV only for eventually consistent configuration. R2 for object artifacts. Do not add Postgres/Hyperdrive, Redis, Rust, Go or a separate service unless measured scale/limits require it.

## Core invariants
- Every read/write is authorized server-side and tenant-scoped; cross-tenant IDOR must fail closed.
- Wallet is append-only ledger; no mutable client balance. No negative available credits.
- Quote → reserve → execute → capture or release. Idempotency key is bound to actor, operation, resource and request hash.
- Daily grants unique by account + Asia/Kolkata service date + policy version.
- Payment fulfillment only after raw-body webhook signature verification and durable unique event persistence. Redirects are not proof of payment.
- Queue delivery is at-least-once. Duplicate delivery cannot duplicate payment, credits or external effects.
- Unknown pricing, invalid reservation, exceeded budget or unverified identity fails closed.
- Tool manifest status and health are distinct. A repo existing does not mean a deployed/healthy tool exists.
- All money/credit changes have actor, reason, correlation and audit record.

## Initial scope and non-goals
P0 is a runnable service foundation, CI, typed config, health/readiness, API schema, database migration skeleton and security/test harness. Then implement tenant/auth boundary and tool manifest, then ledger. Payments are not P0. No live gateway, subscription, admin console, autonomous GitHub write or production deploy in this phase.

## Performance goals
Measure p50/p95/p99 by route and workload. Initial candidate targets: cached catalog read p95 ≤300ms; account read p95 ≤500ms; local ledger operations p95 ≤300ms under declared test envelope. These are proposed targets, not measured guarantees. Correctness dominates latency for ledger and payments. Use indexes/EXPLAIN, small payloads, pagination, bounded concurrency and cache only non-sensitive public catalog/config.

## Security requirements
Threat model auth/session fixation, CSRF, CORS, IDOR, tenant isolation, replay, webhook forgery, race conditions, secret leakage, SSRF, malicious manifests, queue poison, abuse/cost exhaustion and admin privilege escalation. Explicit origin allowlists, secure host-only cookies, CSRF/origin protections, rate limits, payload caps, audit logs, key rotation and kill switches. No wildcard credentialed CORS. External tool URL/redirect DNS and IP checks. Secrets in Worker secret bindings only.

## Cost and legal constraints
Maintain ₹600/month target and ₹1,000/month hard ceiling inclusive of tax/FX. No paid activation or budget increase without explicit approval. No production payments before sandbox tests, provider onboarding, legal/tax/consumer terms, GST/invoice, recurring consent, refund policy and RBI/PPI classification review. Never store PAN/CVV/UPI PIN.

## AI builder workflow
1. Read current tree and all relevant docs; report actual runtime status.
2. Pick one bounded task and state success/failure cases.
3. Write tests first and observe RED.
4. Implement smallest change and observe GREEN.
5. Run full suite, typecheck, lint and build.
6. Review auth/tenant/idempotency/abuse/cost impacts.
7. Update API contract, docs, changelog and ADR.
8. PR with exact test evidence; never claim deploy/security certification without evidence.

## Sources to keep current
- Cloudflare Workers languages/limits/storage/Queues/D1 docs.
- Hono, Zod, Drizzle and OpenAPI official docs.
- OWASP ASVS, API Security Top 10, GenAI Security Top 10 and MCP guidance.
- RBI/payment provider official terms, gateway pricing and India legal/tax guidance.
Check official sources before adopting volatile APIs or pricing.
