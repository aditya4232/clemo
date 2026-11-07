# Clemo Development Handoff

Reviewed 2026-11-07. Architecture and requirements only; no production billing or complete app implementation is claimed.

## Product definition and navigation

Clemo is the unified product workspace and launcher, not a monolith. It should feel like a mature SaaS product with clear navigation, discovery, transparent costs, reliable job status, accessible results and support.

Primary navigation: Home, Explore tools, My activity/Jobs, Credits & billing, Support, Settings/Security and role-gated Admin.

Every tool has a detail page and signed-in run flow. Status values: planned, private_preview, available, degraded, disabled, retired. Only owner-approved and health-verified tools may be called available.

## Bounded contexts

Identity/membership; tool catalog; launch/connection registry; wallet/ledger; checkout/orders; subscriptions/entitlements; job orchestration; artifacts; support; admin/audit; notifications. Keep modular boundaries in one deployable API initially. Agent Workspace and Agent Labs retain their own data stores.

## Credit ledger and daily grant

Suggested tables: wallets, ledger_entries, credit_lots, reservations, reservation_allocations, usage_quotes, credit_policies, idempotency_keys and daily_grants. Use foreign keys, unique constraints and transactions.

- Daily grant unique by account + Asia/Kolkata service date + policy version.
- Separate daily-free, purchased, subscription and promotional buckets.
- Each lot has expiry and source.
- Quote records tool/version, input class, cost drivers, max charge, expiry and policy version.
- Reserve atomically before billable work; allocate buckets deterministically.
- Capture only the permitted charge after verified success; release unused reservation on terminal failure/cancel.
- Retries reuse job/idempotency identity and cannot create a second reservation.
- Refund/chargeback is a compensating ledger entry and entitlement transition, never deletion.
- Browser never decides balance, price, quantity, user identity or payment status.
- Reconciliation detects negative balances, duplicate grants, unmatched orders and orphan reservations.

Do not rely on one cron firing exactly at midnight. Use lazy grant-on-read with a unique DB constraint plus scheduled reconciliation. Pricing hypotheses remain 5 free credits/day initially, 10 only after measured cost; ₹59/₹99 packs and ₹199 monthly plan. Never promise unlimited expensive inference.

## Payment and subscription

Create a provider adapter with normalized order, payment, refund, mandate and webhook events. Evaluate Cashfree and Razorpay in sandbox; onboarding, commercial fees and legal review are prerequisites.

Webhook pipeline: read raw body, verify provider signature, persist unique provider event, enqueue idempotent handler, update order/payment and append ledger entry transactionally, then reconcile settlements. Browser redirect is not proof of payment.

Subscription states: pending_mandate, active, past_due, grace_period, cancelled, expired. Show amount, cadence, next debit, included credits and cancellation instructions. Test failed/revoked mandates, duplicate/out-of-order webhooks and refund after credits are spent.

## Tool execution contract

Every tool implements versioned manifest and JSON schemas, owner, authorization scopes, data classification, quote/cost model, idempotent job creation, queue-backed execution, progress/status/cancel, artifact manifest, retention, bounded retries, timeout, quality fixtures and health check.

Job states: queued, running, awaiting_approval, succeeded, failed, cancelled, expired. Browser disconnect must not terminate durable work. Results are stored separately; downloads use short-lived URLs.

## Initial tool sequence

1. Redact: local-first PDF/image pipeline where feasible; preview and confirm irreversible flattening.
2. Paperwork Pilot: extraction with page/region evidence and confidence; user verifies fields.
3. Scam Check: explain suspicious indicators without automatically visiting links.
4. Repo Brief: explicit read-only GitHub App scope, citations and no writes.
5. PDF toolkit: bounded processing, MIME validation and output checks.
6. Split the Bill: OCR plus deterministic arithmetic and user confirmation.

Then FormFit, PR Evidence Review, API Contract Inspector, Dependency Risk Brief, Log Triage, Knowledge Pack, Research Brief and Meeting Extractor. Each needs owner, test corpus, quality threshold and unit economics.

## Security baseline

Secure sessions, CSRF for cookie-authenticated mutations, rate limits, revocation and MFA for admins. Server-side authorization on every object/action; tenant-isolation tests are mandatory.

Uploads: content-based MIME validation, size/page/decompression limits, malware scan where available, random object keys, private buckets, signed URLs, retention/deletion. Parse/OCR in isolated workers.

SSRF: scheme/host policy, DNS and resolved-IP validation, block loopback/private/link-local/metadata ranges, revalidate redirects and enforce egress controls.

AI: uploaded/retrieved/tool content is untrusted. Separate instructions from data, validate structured outputs, enforce policy outside model, cap tokens/retries/runtime/provider spend. Map controls to OWASP GenAI Top 10 2026, especially prompt injection, sensitive disclosure, excessive agency, supply chain, poisoning, unbounded consumption and improper output handling.

## Support, admin and email

Support case ID, category, status, owner, SLA, timeline, redacted attachments and closure reason. Billing support can inspect order/webhook/ledger references but cannot reveal provider keys or raw documents. Refund/credit adjustments require role, reason and audit; dual approval above configured threshold. Admin uses MFA, least privilege and session audit.

Transactional email adapter covers security alerts, job completion, receipts, payment failure, refund and support updates. Configure SPF/DKIM/DMARC, queue/retry, idempotency, bounce suppression and preference controls.

## Development phases

P0 repo scaffold, CI, migrations, auth boundary, OpenAPI contracts and tool manifest.
P1 ledger, quote/reserve/capture/release, daily grant and audit.
P2 job state machine, queue workers, artifacts, notifications and first tools.
P3 payment sandbox, webhook verification, reconciliation, refunds and subscriptions.
P4 support console, billing admin, catalog, health checks and abuse controls.
P5 more tools, GitHub App, secure cross-product launch and signed events.

## Acceptance gates

Property tests for no double grant/spend; concurrent reservations; idempotency; duplicate/delayed/invalid/out-of-order webhook; payment sandbox success/fail/cancel/refund; IST boundaries; retry/cancel/dead-letter; tenant isolation, IDOR, CSRF, SSRF, upload bomb and prompt injection; mobile/accessibility; per-tool p50/p95/p99 costs; restore and ledger reconciliation drill.

Before sales: approve unit economics, gateway contract, GST/invoice, consumer terms, recurring consent, privacy/deletion, refund and legal classification of credits. No paid services activated without owner approval.
