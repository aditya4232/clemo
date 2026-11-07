# Clemo Roadmap

## Phase 0 — contracts and static shell
- Tool manifest and event schema with versioning.
- Accurate catalog seeded with Agent Workspace, Agent Labs and Clemo tools.
- Status labels and health/deployment distinction.
- Responsive workspace shell and accessible tool cards.
- No cross-product SSO or deployment mutation.

## Phase 1 — identity and workspace
- Auth, workspace membership, invitations, sessions and audit log.
- Host-only cookies and CSRF/origin protection.
- User data export and account deletion semantics.

## Phase 2 — GitHub integration
- GitHub App installation and explicit repository selection.
- Read-only metadata sync, webhooks, ETags and review diffs.
- Disconnect/revoke and retention controls.
- No Actions write or deployment permission.

## Phase 3 — first-party tool integrations
- Launch Agent Workspace and Agent Labs through independent auth.
- Add signed, minimal activity events.
- Add opt-in sanitized trace export from Agent Workspace to Agent Labs.
- Add one-time launch code or OIDC only after security review.

## Phase 4 — Clemo document tools
- Browser-first OCR/extraction/redaction/FormFit flows.
- Evidence checks, test corpus and quality reports.
- Privacy lifecycle tests and deletion verification.

## Phase 5 — deployment control plane (separate approval)
- Approved deployment registry, preview metadata and health checks.
- Build/deploy approvals, isolated build runner, provenance and rollback.
- Never execute arbitrary code in the main API Worker.

## Exit criteria
Each phase requires tests, accessibility review, security threat review, operational runbook, cost forecast and owner approval. Do not label planned work as live.
