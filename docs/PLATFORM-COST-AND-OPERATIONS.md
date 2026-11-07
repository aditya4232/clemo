# Platform Cost and Operations Guardrails

Reviewed 2026-11-07. Pricing/limits change; recheck official docs before provisioning. This is a cost model, not a bill estimate.

## Current platform direction

Keep the first release modular and serverless:
- Cloudflare Workers + Hono for Clemo APIs and webhooks.
- D1 for relational metadata, ledger, orders, reservations and support cases.
- Queues/Workflows for asynchronous tool jobs, payment fulfillment, email and reconciliation.
- R2 for private artifacts; signed short-lived download URLs.
- Durable Objects only for serialized coordination or live state where needed.
- Separate sandbox service only after explicit threat model and budget approval.

## Verified pricing/limit observations

Cloudflare's current Workers pricing documentation lists D1 Free allowances of 5 million rows read/day, 100,000 rows written/day and 5 GB total storage; Paid lists 25 billion rows read/month and 50 million rows written/month included, with additional row and storage rates. The pricing page also describes Queue operation accounting and Workflows request/CPU/storage/step billing.

D1 limits documentation currently lists a 500 MB maximum database size on Free and 10 GB on Paid, with query/subrequest and SQL limits. These are platform ceilings, not a design target. Index every high-selectivity ledger query and avoid unbounded scans.

Official references:
- https://developers.cloudflare.com/workers/platform/pricing/
- https://developers.cloudflare.com/d1/platform/pricing/
- https://developers.cloudflare.com/d1/platform/limits/
- https://developers.cloudflare.com/workflows/reference/pricing/

## Budget discipline

Previously approved target is ₹600/month, hard ceiling ₹1,000/month including tax/FX. Do not add a paid service or upgrade plan without owner approval. Before production estimate:
1. Workers/API requests and CPU per tool.
2. D1 rows read/written, indexes and storage.
3. Queue messages, retries, dead-letter and retention.
4. Workflow step count and wait duration.
5. R2 stored GB, Class A/B operations and egress.
6. Email volume and transactional provider price.
7. Model/OCR inference, fallback and retry cost.
8. Payment gateway fees, GST on fees and refund costs.
9. Monitoring, backups, domain and tax/FX.
10. Cost per successful job at p50/p95/p99.

Credit prices must be derived from measured unit costs, not intuition. Maintain a per-tool cost sheet with input classes, model/provider, tokens/pages/CPU/storage, p50/p95/p99, max budget and gross margin. Put per-user and global provider spend caps in place before enabling paid usage.

## Reliability controls

- Idempotent queue consumers and payment event handlers.
- Dead-letter queues with audited replay.
- Scheduled reconciliation for wallet, payments and job states.
- Database migrations are backward-compatible and reviewed.
- Backup/restore drills and ledger integrity checks.
- Provider circuit breakers and concurrency quotas.
- Structured logs, correlation IDs, metrics and alerts with secrets/PII redacted.
- Health endpoints are read-only and bounded.
- Runbooks for provider outage, webhook delay, payment dispute, credit mismatch, data deletion and key compromise.

## Operational SLO proposals

Initial targets are proposals to validate against load and budget:
- API availability: 99.5% monthly.
- Payment webhook: verify and durably persist before acknowledge.
- Credit mutation: atomic and idempotent.
- Job status: user-visible state updates and bounded terminal timeout.
- Critical billing mismatch: alert and stop further settlement for affected reservation.
- RPO/RTO: set after backup/restore exercise; do not claim until tested.

## Cost control kill switches

Global and per-provider monthly spend ceilings; per-account daily quotas; max job runtime; max upload pages/bytes; concurrency caps; model token limits; fallback disable switch; expensive tool feature flags; emergency tool disable. Reaching a cap should fail safely with a clear message, not silently use another paid provider.
