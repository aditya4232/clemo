# Clemo Workspace Architecture

## Bounded contexts
- Identity: account, workspace, membership, session, invitation.
- Catalog: tool manifest, category, status, launch target and health.
- Connections: GitHub App installations, selected repositories and webhooks.
- Launcher: authorization decision, short-lived launch code and redirect.
- Activity: minimal product events and audit history.
- Document tools: browser-first pipelines isolated from the workspace control plane.

The workspace shell owns identity, catalog and connections. Each product owns its own domain data and execution.

## Tool manifest
Required fields: id, slug, display_name, description, owner, repository_url, status (concept/design/preview/live/maintenance/archived), canonical_origin, environment_origins, launch_path, auth_mode, optional health_check_url, required_scopes, supported_events, manifest_version, data_classification, privacy_url, support_url, last_verified_at and owner_approved_at.

Reject javascript:, data:, file:, localhost and private-network URLs in production. Validate hostname against an owner-maintained allowlist. A manifest cannot grant permissions to itself.

## Launch assertion
Use a one-time, audience-bound, short-lived opaque code. Persist only a hash, expiry, intended user, workspace, tool ID, audience and nonce. Redeem server-to-server once; consume atomically. Never put API keys, provider tokens, full chat context or long-lived JWTs in query parameters. The receiving product re-checks its own authorization and creates its own session.

If the receiving product does not implement secure redemption, use an ordinary link and require independent authentication. Do not simulate SSO with shared cookies.

## GitHub integration
Prefer a GitHub App over a user PAT. Request metadata/read-only permissions initially. Add contents/actions/deploy permissions only for a documented feature requiring them and explicit approval. Validate webhook HMAC signatures, timestamp tolerance, delivery ID uniqueness and installation ownership. Queue processing is idempotent. Respect ETag, 304, Retry-After and rate-limit reset; bounded exponential backoff. Limits vary by authentication type and installation.

## Event envelope
event_id, event_type, schema_version, occurred_at, source_tool_id, workspace_id, pseudonymous actor_ref, minimal allowlisted data, signature and delivery_id. Never include raw prompts, document contents, secrets, provider keys or chat transcripts. Reject unknown schema versions and replayed event IDs.

## Data ownership
| Data | System of record |
|---|---|
| Account/workspace/membership | Clemo |
| Tool manifests and connection metadata | Clemo |
| GitHub repository/installation metadata | Clemo cache + GitHub source |
| Chat history, model config, provider key vault | Agent Workspace |
| Evaluation runs and experiment artifacts | Agent Labs |
| Document contents/generated files | Clemo document tool or its explicit storage subsystem |
| Public pages/articles | Portfolio CMS |

Clemo stores references and summaries, not duplicated product databases.

## Deployment control plane is not the initial MVP
Launching a deployed app and deploying arbitrary repository code are different capabilities. Deployment automation introduces code execution, secrets, build isolation, artifact provenance, supply-chain, tenant isolation and cost risks. Initial Clemo only launches owner-approved deployments. Build/deploy orchestration is a later separately approved subsystem.
