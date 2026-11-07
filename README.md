# Clemo — the workspace for Aditya's product ecosystem

**Clemo is the front door and workspace shell for the Aditya CUZ.in product ecosystem.** It helps users discover, connect to, launch and return to independent tools and projects from one coherent workspace.

**Canonical app:** clemo.adityacuz.in (planned)  
**Public portfolio:** adityacuz.in  
**Agent chat:** chat.adityacuz.in (Agent Workspace; separate product)  
**Status:** product architecture/design phase. No executable runtime, authentication, wallet or payment implementation exists yet.

## Product model
1. Workspace shell: identity, workspace home, project catalog, connected repositories, launch/deep links, recent activity, account settings and (after approval) billing/usage.
2. First-party tools: Agent Workspace, Agent Labs and future tools. Each remains its own repository, deployment, release cycle and security boundary.

Clemo is not a monolith and does not absorb every product's code or data. It is an orchestrator and product launcher.

## Experience
Home for recent/pinned tools and workspace health; catalog by capability/status; least-privilege GitHub App connection; approved product launcher; scoped context references; minimal signed activity events; account, permissions and data settings.

Tool catalog states: Concept, Design, Preview, Live, Maintenance or Archived. A repository connection is not a deployment.

## Integration rules
- Portfolio publishes curated public metadata; it does not read Clemo private workspace state.
- Agent Workspace owns chats, provider keys, tool permissions and chat deletion.
- Agent Labs owns experiments and evidence. Only synthetic/public or sanitized opt-in traces may be shared.
- Future tools implement versioned manifests and event contracts.
- No broad shared parent-domain cookies, long-lived tokens in query strings or silent private-data sharing.
- If secure one-time launch assertion/OIDC is not implemented, use ordinary links and independent sign-in.
- GitHub App starts read-only. Repository changes create owner-review diffs; no auto-publish or auto-deploy.

## Security
Tenant membership is checked server-side. Repository connections are explicit. Webhooks use signature validation, replay protection and idempotency. Treat repository text, issues, commits, files and AI outputs as untrusted. No arbitrary Git URL cloning or user-supplied server-side fetch. No production deployment automation until isolation, approvals, audit and rollback are implemented.

## Roadmap
P0 runnable service foundation, CI, typed contracts, health/readiness and security/test harness.  
P1 auth, workspace membership, settings, tool manifest and audit.  
P2 wallet ledger, quotes/reservations and idempotent settlement.  
P3 queue-backed tools and artifacts.  
P4 payment sandbox, webhook verification, reconciliation and legal review.  
P5 secure cross-product launch and signed events.

## Documentation
- [AI builder context](docs/AI-BUILD-CONTEXT.md) — canonical product truth, boundaries, invariants and implementation workflow.
- [Architecture](docs/ARCHITECTURE.md)
- [Workspace contracts](docs/WORKSPACE.md)
- [Integration contracts](docs/INTEGRATION-CONTRACTS.md)
- [Credit, commerce, support and tools](docs/CREDITS-COMMERCE-AND-SUPPORT.md)
- [Platform cost and operations](docs/PLATFORM-COST-AND-OPERATIONS.md)
- [Platform engineering and consumer quality](docs/PLATFORM-ENGINEERING-AND-CONSUMER-QUALITY.md)
- [Development handoff](docs/DEVELOPMENT-HANDOFF.md)
- [Roadmap](docs/ROADMAP.md)
- [Security](SECURITY.md)
