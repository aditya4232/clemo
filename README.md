# Clemo — the workspace for Aditya's product ecosystem

**Clemo is the front door and workspace shell for the Aditya CUZ.in product ecosystem.** It helps users discover, connect to, launch and return to independent tools and projects from one coherent workspace.

**Canonical app:** clemo.adityacuz.in (planned)  
**Public portfolio:** adityacuz.in  
**Agent chat:** chat.adityacuz.in (Agent Workspace; separate product)  
**Status:** product architecture/design phase. Do not infer that deployments, integrations or features are live.

## Product model

Clemo has two layers:
1. Workspace shell: identity, workspace home, project catalog, connected repositories, launch/deep links, recent activity, account settings and (after approval) billing/usage.
2. First-party tools: Agent Workspace, Agent Labs and future tools. Each remains its own repository, deployment, release cycle and security boundary.

Clemo is not a monolith and does not absorb every product's code or data. It is an orchestrator and product launcher.

## Experience

Home for recent/pinned tools and workspace health; catalog by capability/status; least-privilege GitHub App connection; approved product launcher; scoped context references; minimal signed activity events; account, permissions and data settings.

Tool catalog states must be accurate: Concept, Design, Preview, Live, Maintenance or Archived. A repository connection is not a deployment.

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

P0 manifest/schema, shell, navigation and static catalog.
P1 auth, workspace membership, settings and audit.
P2 GitHub App read-only sync and independent product launch links.
P3 scoped launch assertions and minimal signed events.
P4 separately approved deployment control plane.

## Documentation
- [Architecture](docs/ARCHITECTURE.md)
- [Workspace contracts](docs/WORKSPACE.md)
- [Credit, commerce, support and tools](docs/CREDITS-COMMERCE-AND-SUPPORT.md)
- [Development handoff](docs/DEVELOPMENT-HANDOFF.md)
- [Roadmap](docs/ROADMAP.md)
- [Security](SECURITY.md)
- [Decision: browser-first tools](docs/DECISIONS/0001-browser-first.md)
