# Credits, Commerce, Support and Tool Platform

Status: proposed; implementation and provider onboarding are not complete.
Date: 2026-11-07.

## Product promise

Clemo is a coherent SaaS workspace: discover a task, understand data processing, see price and privacy boundary, run it, inspect results, download/export and get help. Never imply a feature is live when deployment or health is unknown.

## Daily credits and pricing experiment

Hypotheses: 5 free credits per account per India-local calendar day at initial launch; 10/day only as a configurable experiment after cost tests; ₹59 and ₹99 one-time top-ups; ₹199/month membership with defined included credits, never unlimited expensive AI. Tool-specific costs must reflect measured model, OCR, compute, storage, retries, support and abuse costs.

Daily grant algorithm:
1. Derive account from verified session, never request body.
2. Compute service date in Asia/Kolkata.
3. Insert unique account + service date + grant-policy version.
4. Append daily grant ledger entry transactionally.
5. Expire daily bucket at next reset; keep paid balance separate.
6. Concurrent requests cannot claim multiple grants.

Use append-only ledger and reservation records, not a mutable balance as source of truth. Quote → reserve → enqueue/run → capture OR release. Quote is server-calculated from tool version and input size. Reserve before billable calls. Capture on verified completion; release on failure/cancellation. Retries reuse job/idempotency identity. Partial billing requires explicit disclosed rules. Admin adjustment requires role, reason and audit. Bucket priority and expiry must be visible in transaction history.

## Suggested versioned API

- GET /v1/me/wallet
- GET /v1/me/wallet/transactions?cursor=
- POST /v1/credits/quotes
- POST /v1/credits/reservations
- POST /v1/credits/reservations/:id/capture
- POST /v1/credits/reservations/:id/release
- POST /v1/checkout/sessions
- GET /v1/billing/orders/:id
- POST /v1/billing/webhooks/:provider
- POST /v1/billing/orders/:id/refund-requests
- GET /v1/me/subscription
- POST /v1/me/subscription/cancel

Mutations require auth, CSRF protection for cookie-authenticated requests, idempotency key, rate limit, schema validation and correlation ID. Webhooks require provider signature validation against raw body.

## Payment provider adapter

Shortlist Razorpay and Cashfree; run sandbox and onboarding comparison. Compare UPI intent/QR, cards, netbanking, UPI AutoPay/e-mandates, mandate cancellation, failed renewal recovery, refund APIs, partial refunds, settlement reports, support and actual fees.

₹59/₹99 are server-priced SKU records. ₹199 subscription is an explicit recurring mandate; display amount, interval, next charge, benefits and cancellation. UPI recurring eligibility depends on provider/bank and user approval; always offer a non-recurring fallback. Never store UPI PIN, PAN/CVV or payment credentials. Browser redirect is not proof of payment. Verify and deduplicate webhooks, grant credits only after verified paid/captured event, reconcile daily. Refund and consumed-credit treatment must be disclosed and tested.

Obtain legal/accounting review of RBI PPI classification, consumer law, GST, invoices, subscription consent and refunds. Keep credits non-transferable, non-cash-redeemable and usable only for Clemo services unless counsel approves otherwise. This is an engineering constraint, not legal advice.

## UI surfaces

- Workspace home: categories, recent tools, credits, plan, product health and support.
- Tool detail: purpose, supported inputs, privacy/retention, expected time, quote, limits and sample.
- Run panel: progress, cancel, safe retry, job ID and failure reason.
- Results: preview, evidence/source links, download/export, retention notice and delete.
- Billing: bucket balances, ledger, packs, plan, invoices, cancellation and refund request.
- Support: ticket creation/timeline, payment-credit dispute and privacy request.
- Settings: profile, security, provider profiles where BYOK is supported, sessions, export/delete and notifications.

## Tool portfolio

Each tool needs a one-page spec, owner, versioned manifest, data classification, threat model, test corpus, credit-cost model and health endpoint before it is available.

Document and personal productivity:
1. Paperwork Pilot — form classification/extraction with field-level confidence and page/region evidence; never submit forms automatically.
2. FormFit — fill from user-approved profile fields; review every field before export.
3. Redact — PII detection, preview, explicit confirmation before irreversible flattening.
4. Split the Bill — receipt OCR, item assignment and transparent tax/tip arithmetic.
5. Scam Check — explain indicators in messages/URLs/documents; no guarantee and no automatic interaction with suspicious sites.
6. PDF toolkit — merge/split/compress/convert locally where feasible; disclose server processing.

Engineering and developer productivity:
7. Repo Brief — explicit read-only repository scope, architecture map and onboarding guide.
8. PR Evidence Review — summarize diffs/tests with source line references; no auto-merge.
9. API Contract Inspector — OpenAPI diff and breaking-change detection.
10. Log Triage — redact secrets, cluster errors and propose causes; never execute shell commands.
11. Dependency Risk Brief — SBOM/advisory matching and upgrade plan; distinguish advisory from confirmed exploit.
12. Data/SQL Explainer — read-only query explanation; no production execution by default.

Research and knowledge:
13. Research Brief — source-backed synthesis, claim-source mapping, dates and uncertainty.
14. Meeting Extractor — action items and decisions with speaker/time citations.
15. Knowledge Pack — private document Q&A with citations and user-controlled deletion.

Avoid thin single-prompt wrappers. Each tool needs deterministic preprocessing, schema validation, provenance, bounded model calls, test fixtures and graceful failure paths.

## Tool manifest and job contract

Manifest: tool_id, version, display_name, description, category, owner, status, origin, health_url, input/output schema, data classification, retention, credit quote policy, max runtime, capabilities, auth scopes and support URL.

Status: planned, private_preview, available, degraded, disabled, retired. Repository existence never implies availability.

Jobs: queued, running, succeeded, failed, cancelled, expired. Include timestamps, progress, correlation ID and safe retryability. Long jobs use queues/workflows, not long HTTP requests.

## Abuse and safety controls

Per-account/IP rate limits, concurrency caps, max upload bytes/pages, MIME sniffing, archive-bomb protection, malware scanning where available, SSRF denylist plus DNS/IP revalidation, isolated file processing, short-lived signed URLs, output sanitization, prompt-injection isolation, provider spend limits and circuit breakers. Treat uploaded content, retrieved text, tool outputs, filenames and OCR as untrusted data. Model instructions cannot override deterministic authorization.

## Support and admin

Ticket categories: payment, credit, refund, tool result, account, security, privacy/deletion, feedback. Admin console shows ledger trace, order/payment/refund state, webhook delivery, job state, provider cost, support timeline and audit. Refund/credit adjustment requires scoped permission, reason and second approval above a threshold. Support operators cannot view BYOK secrets or raw files by default.

## Launch gates

Credit concurrency/idempotency property tests; duplicate/out-of-order/missing webhook tests; sandbox payment success/failure/cancel/refund; IST midnight grant tests; refund-vs-consumed-credit tests; queue retry/provider timeout tests; upload/SSRF security tests; accessibility/mobile/reduced-motion; tool quality benchmark and p95 cost; approved terms, privacy, retention, refund and support pages.

## References

- RBI PPI Master Directions: https://rbi.org.in/scripts/BS_ViewMasDirections.aspx?id=12156
- Razorpay pricing: https://razorpay.com/pricing/
- Cloudflare Durable Object alarms: https://developers.cloudflare.com/durable-objects/api/alarms/
