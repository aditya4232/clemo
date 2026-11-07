# Ecosystem Integration Contracts

Status: proposed v1 contracts. No public API endpoint or SSO flow is implemented yet.

## Principles

Each product owns its identity/session, data and authorization. Clemo owns the workspace catalog and credits ledger. Agent Workspace owns conversations, provider profiles and MCP grants. Agent Labs owns experiment datasets and evidence. Portfolio owns public editorial content. No shared database and no implicit cross-product data sharing.

## Tool manifest v1

Required fields:
- schema_version, tool_id, tool_version, display_name, summary, category, owner
- repository_url, canonical_origin, allowed_environment_origins, launch_path
- lifecycle_status, auth_mode, minimum_client_version
- health_check_url, last_health_check_at, owner_approved_at
- capabilities, required_scopes, data_classification
- privacy_url, retention_summary, support_url

Lifecycle values: planned, private_preview, available, degraded, disabled, retired. Status changes require an owner and timestamp. Health check is evidence of endpoint reachability only, not security certification.

Validate HTTPS origin against owner allowlist. Reject javascript/data/file schemes, localhost, private/link-local/metadata IPs and unexpected ports. Redirect targets must be revalidated. Manifest content is untrusted data and cannot grant itself capabilities.

## Signed event envelope v1

Required: event_id (UUID), schema_version, event_type, occurred_at, source_tool_id, workspace_id, pseudonymous actor_ref, correlation_id, minimal payload, key_id and signature.

- Sign canonical serialized bytes with a rotated service key.
- Verify signature, timestamp tolerance, key status and schema version.
- Persist unique event_id before side effects; replay returns prior result.
- Use transactional outbox for reliable publication.
- Reject unknown schema versions and oversized payloads.
- No raw prompts, chat transcripts, provider keys, document bodies, payment card data or sensitive personal data.
- Events are notifications, not authorization. Receiver rechecks user/workspace access.

Initial allowlisted events: tool.health_changed, tool.run.started, tool.run.completed, tool.run.failed, artifact.ready, experiment.completed, billing.usage_reported. Billing events are accepted only against an existing Clemo reservation ID and signed service identity.

## Usage settlement

1. Clemo creates quote with quote_id, reservation_id, tool_id, user/workspace, maximum credits, pricing_policy_version and expiry.
2. Tool receives opaque reservation reference, not wallet credentials.
3. Tool starts job with stable job_id and idempotency key.
4. Tool reports completion with reservation_id, job_id, actual billable units, declared tool version, result status and signature.
5. Clemo verifies signer, reservation, quote expiry, tool identity, maximum and idempotency.
6. Clemo appends capture or release ledger entries transactionally.
7. Mismatch, unknown pricing version or expired reservation goes to reconciliation/manual review; never silently overcharge.

Agent Workspace provider inference costs are not included in Clemo credit capture when BYOK is used. Platform-side execution/storage charges must be separately disclosed and quoted.

## Launch assertion

If OIDC is not available, issue a random one-time opaque launch code. Store only its hash, user, workspace, tool, audience, expiry and nonce. Redeem server-to-server exactly once. The receiving product establishes its own session and performs its own authorization. Never place long-lived tokens or private context in URL query parameters.

## Versioning and compatibility

Use additive optional fields within a major version. Breaking schema changes require new major version and a migration window. Producers publish contract tests; consumers validate fixtures in CI. Unknown fields may be ignored only where explicitly allowed; unknown event types are rejected or quarantined. Keep a compatibility matrix by tool version.

## Operational and privacy requirements

Key rotation, event delivery metrics, dead-letter queue, replay tool with audit, event retention and deletion are required. Logs contain event IDs and correlation IDs, not payload secrets. A disconnected product must not block other tools. Disable a compromised signing key and revoke the affected integration without invalidating unrelated products.
