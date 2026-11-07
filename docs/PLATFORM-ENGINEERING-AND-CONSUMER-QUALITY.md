# Clemo platform engineering and consumer quality

**Reviewed:** 2026-11-07  
**Status:** architecture and acceptance contract. Clemo remains design phase; no production service, payment flow or latency result is claimed.

## Product outcome

Clemo is the simple, trustworthy workspace and tool-discovery shell for independent products. A consumer should be able to understand what a tool does, what data it uses, what it costs, what action it will take and how to stop or undo it. Clemo owns identity/workspaces, catalog, launcher, minimal activity, credits/commerce and support—not Agent Workspace conversations, Agent Labs datasets or Portfolio content.

## Market evidence and product implications

- Ipsos' 2026 India AI Monitor reports >80% awareness of AI among surveyed Indian consumers: https://www.ipsos.com/en-in/ipsos-ai-monitor-2026-india-report
- NIQ's Aug 2026 top-eight-metro study reports high usage/openness to AI shopping assistance (92% used AI while shopping; 96% open to tools). Scope is metro and shopping-specific: https://nielseniq.com/global/en/insights/report/2026/agentic-commerce-india-2026/
- Gartner's 2026 US survey found a gap between willingness to use AI for discovery and willingness to delegate purchase decisions; users demand accurate comparison and retain final control: https://www.gartner.com/en/newsroom/press-releases/2026-05-27-gartner-survey-finds-consumers-want-ai-shopping-help-but-not-ai-purchase-decisions
- Capgemini's 2026 consumer survey highlights transparent pricing, clear rules for AI action, data control and privacy: https://www.capgemini.com/insights/research-library/what-matters-to-todays-consumer-2026/

Implications: consumer trust is a product feature. Display clear pricing and credit estimates before execution; make permissions understandable; show progress and explain failures; keep user approval for consequential actions; support mobile-first flows and low-bandwidth operation. AI should reduce effort, not remove user agency.

## Runtime and language decision

Keep the initial control plane TypeScript on Cloudflare Workers with Hono, Zod validation and D1 where relational read-heavy data fits. This minimizes runtime and deployment complexity while matching the chosen platform.

- Use D1 for accounts, workspace metadata, tool catalog, orders and append-only ledger where data size and workload fit its limits. Design every query around measured access patterns, indexes and EXPLAIN QUERY PLAN.
- Use Workers KV only for high-read, eventually consistent configuration/session-like data—not balances, payment state or other strongly consistent financial truth.
- Use Durable Objects for narrow coordination atoms (for example, per-account wallet reservation serialization or per-run coordination) only when strong ordering is required. Never route all accounts through one global object.
- Use Queues for email, notifications, provider callbacks and deferred work. Delivery is at-least-once: every consumer must be idempotent; configure DLQs and replay tooling.
- Use R2 for object/blob storage; avoid storing files unless a product explicitly owns retention and deletion.
- Use Hyperdrive with Postgres only if D1 limits, relational requirements or measured query patterns justify an external Postgres database.
- Rust/WASM is reserved for measured CPU-bound parsing/validation modules; Go for a separately operated service with demonstrated need. Do not rewrite request routing or business logic for benchmark vanity.

Cloudflare's storage guide maps D1 to lightweight relational apps, KV to high-read configuration, Durable Objects to strongly consistent coordination, Queues to deferred jobs and R2 to objects: https://developers.cloudflare.com/workers/platform/storage-options/  
Workers supports JS/TS and Rust directly; Go can run through WASM with different integration costs: https://developers.cloudflare.com/workers/languages/  
D1 indexes reduce scanned rows and billable reads; validate with EXPLAIN QUERY PLAN: https://developers.cloudflare.com/d1/best-practices/use-indexes/

## Performance and reliability contract

Proposed launch targets; measure before claiming:

| Metric | Initial target |
|---|---:|
| Catalog/public metadata read | p95 ≤300ms cached |
| Workspace/account read | p95 ≤500ms |
| Wallet quote/reserve/capture/release API | p95 ≤300ms excluding external provider |
| UI interaction acknowledgement | ≤100ms perceived response; optimistic state must reconcile |
| API 5xx | <0.5% rolling 7 days after launch |
| Queue processing | age p95 <60s for routine jobs; job-specific deadlines documented |
| Monthly availability | ≥99.5% after telemetry exists |

The wallet must never sacrifice correctness for latency. Reserve/capture/release must be transactional, append-only, idempotent and auditable. Use optimistic UI only for reversible presentation, never to claim a payment or ledger mutation succeeded before server confirmation.

## Wallet and commerce invariants

1. Append-only ledger; balance is derived or maintained transactionally from ledger entries.
2. Unique idempotency key per actor + operation + resource; reject key reuse with a different request body.
3. Reservation lifecycle: quote → reserve → execute → capture or release. Expiry is explicit.
4. Never permit negative available balance through concurrent requests; test race conditions with parallel reservations.
5. Separate daily-free, purchased, subscription and promotional buckets; explicit expiry and spend order.
6. Daily grants unique by account + Asia/Kolkata service date + policy version.
7. Payment webhook signature over raw body, durable event record, unique provider event ID, idempotent fulfillment and settlement reconciliation. Redirects never prove payment.
8. Refund/chargeback are compensating ledger entries. Document post-consumption refund policy.
9. No card PAN/CVV/UPI PIN stored. Do not enable live payment processing until legal, tax, consent and sandbox review.
10. Show quote, expected credits, external provider cost (if any), limitations and approval requirement before execution.

## Security boundaries

- Worker authorization is authoritative; deny by default. Enforce tenant/workspace scope in every query and mutation.
- Validate session issuer, audience, expiry and CSRF/origin requirements. No wildcard credentialed CORS; use exact trusted origins.
- Tool manifest is signed/versioned or owner-verified; display capabilities, data class, scopes, status and owner.
- Launch assertions are short-lived, audience-bound, single-use and exchanged server-to-server; never place long-lived tokens in URLs.
- GitHub integration starts read-only, selected repository scope, ETag/backoff, HMAC webhook validation, replay protection and owner-reviewed diff before publish.
- Secrets live in platform secret bindings/KMS; never in code, browser bundles, URLs, logs or analytics.
- Rate limit auth, quote, wallet, upload and webhook endpoints; payload and concurrency caps; global and per-account kill switches.
- Treat tool descriptions, README text, webhook payloads and model-generated content as untrusted.
- Use least privilege for all external integrations; record actor, scope, decision, idempotency key and outcome in audit events.

OWASP's agent/MCP guidance calls for least agency, untrusted external content, reviewed tool metadata and deterministic permission boundaries: https://owasp.github.io/DevSecOpsGuideline/2-Process/2-2-Develop/2-2-5-AI-Agent-and-MCP-Security/  
NSA MCP security design considerations: https://www.nsa.gov/Press-Room/Press-Releases-Statements/Press-Release-View/Article/4496698/nsa-releases-security-design-considerations-for-ai-driven-automation-leveraging/

## Consumer UX contract

- A first-time user can discover a useful tool and understand its input, output, data handling and cost without reading technical documentation.
- No unexplained "credits"; show exact credit amount and estimated monetary equivalent where appropriate.
- Tool status vocabulary is explicit: planned, available, degraded, paused, retired. Distinguish repo, implementation, deployment, health and user authorization.
- Progress is observable; long tasks can be resumed, canceled and retried safely.
- Errors explain what happened, whether any credits were consumed, and the next safe action.
- No forced autonomous execution for external writes, purchases, publishing or destructive actions.
- Accessible mobile forms, clear confirmation, undo where possible, and human support path for billing/security issues.

## Operational and cost gates

Maintain the existing ₹600/month target and ₹1,000/month hard ceiling inclusive of tax/FX. No paid tier, paid gateway, production database or external inference service without explicit approval. Track cost per successful tool task, not just request cost.

- Per-account, per-tool and global budgets; hard stop before billable execution.
- Queue retries bounded; idempotent handlers; DLQ alert and audited replay.
- Backups plus restore drill; migration rollback plan; provider reconciliation.
- Measure p50/p95/p99 latency, D1 rows read/written, cache hit ratio, queue age, retry rate, ledger mismatch, auth failures and cost per success.
- Alerts at 50/75/90% of the approved monthly ceiling; optional AI jobs disabled at 90%.
- No production payment activation, DNS or deploy from this design document.

## Delivery order

P0: executable project scaffold, typed config, CI, migrations, auth/tenant isolation and security test harness.  
P1: versioned tool manifest, catalog, launcher and audit trail.  
P2: ledger invariants and quote/reserve/capture/release with concurrency and idempotency tests.  
P3: queue/outbox, DLQ, notifications, reconciliation and support.  
P4: sandbox payment adapter and contract tests; legal review before production.  
P5: first owned tool integrations, load testing, abuse testing and staged rollout.

## Release gates

Unit, integration and contract tests; cross-tenant IDOR; auth/session expiry; webhook signature/replay; concurrent wallet reservations; duplicate queue delivery; ledger reconciliation; rate limit; payload abuse; backup/restore; migration rollback; cost ceilings; load test at declared traffic envelope. Publish measured results and known limits, never estimates as facts.

## References

- Cloudflare Workers limits: https://developers.cloudflare.com/workers/platform/limits/
- Smart Placement: https://developers.cloudflare.com/workers/configuration/placement/
- Queues delivery guarantees: https://developers.cloudflare.com/queues/reference/delivery-guarantees/
- Queues DLQ: https://developers.cloudflare.com/queues/configuration/dead-letter-queues/
- Durable Objects design rules: https://developers.cloudflare.com/durable-objects/best-practices/rules-of-durable-objects/
- D1 indexes: https://developers.cloudflare.com/d1/best-practices/use-indexes/
