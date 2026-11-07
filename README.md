# Clemo — the workspace for Aditya's product ecosystem

**Clemo is the front door and workspace shell for the Aditya CUZ.in product ecosystem.** It helps a user discover, connect to, launch and return to independent tools and projects from one coherent workspace.

**Canonical app:** `clemo.adityacuz.in` (planned)  
**Public portfolio:** `adityacuz.in`  
**Agent chat:** `chat.adityacuz.in` (Agent Workspace; separate product)  
**Status:** product architecture / design phase. Do not infer that deployments, integrations or product features are already live.

## Product model

Clemo has two layers:

1. **Workspace platform (the shell):** identity, workspace home, project catalog, connected repositories, launch/deep links, recent activity, saved workspaces, account settings, billing/usage surfaces if ever approved.
2. **First-party tools (independent products):** Agent Workspace, Agent Labs and future tools. Each remains its own repository, deployment, release cycle and security boundary.

Clemo is not a monolith and does not absorb the code of every connected project. It is an orchestrator and product launcher. The first-party tools can be selected like projects in an IDE/workspace selector, but their runtime remains independently deployed.

## Workspace experience

- **Home:** recently opened tools, pinned tools, workspace health, useful shortcuts and curated announcements.
- **Tool catalog:** browse first-party tools by category, capability, status and access level.
- **Connect repository:** connect an allowlisted GitHub repository through least-privilege GitHub App access; show repository identity, branch, permissions and connection status.
- **Project launcher:** open the product's canonical URL or a scoped deep link. Show an explicit "not deployed" state if no valid deployment exists.
- **Workspace context:** selected project, environment, user preferences and approved context references. Never silently share one product's private conversation/files with another.
- **Activity:** events from connected products through signed, versioned webhooks; only minimal metadata by default.
- **Settings:** connected accounts, permissions, sessions, data controls and optional usage.

## Initial tool catalog

| Tool | Role | Canonical repository | State at last inspection |
|---|---|---|---|
| Agent Workspace | Agentic chat, model/provider selection, inspectable tool execution | https://github.com/aditya4232/agent-workspace | Design phase |
| Agent Labs | Reproducible agent evaluations, trace analysis and safety experiments | https://github.com/aditya4232/agent-labs | Design phase |
| Clemo Paperwork | Document extraction, validation and privacy workflows | This repository | Design phase |

The tool catalog must display accurate availability: Concept, Design, Preview, Live, Maintenance or Archived. A repository connection is not a deployment.

## Product flows

### A. Sign in and open a tool
1. User signs in to Clemo.
2. Clemo resolves membership and workspace entitlements.
3. User chooses a tool.
4. Clemo checks tool availability and the user's permission.
5. Clemo opens the independent tool with a short-lived, scoped launch assertion or normal sign-in redirect.
6. The tool establishes its own session and enforces its own authorization.

Do not pass long-lived bearer tokens in query strings. Do not share broad parent-domain cookies. Prefer OAuth/OIDC authorization-code flow with PKCE or a short-lived, audience-bound, one-time launch code redeemed server-to-server. If SSO is not implemented, use a normal sign-in at each product.

### B. Connect a repository
1. User initiates GitHub App installation/authorization.
2. Show requested repository scopes and exact permissions.
3. Store installation/repository identifiers, not personal access tokens.
4. Verify webhook signatures and installation ownership.
5. Sync repository metadata with rate-limit handling and ETags.
6. Repository changes create a reviewable diff; never auto-publish or auto-deploy.
7. User can disconnect; revoke access and purge derived metadata according to retention policy.

### C. Launch / deployment
Clemo stores a product manifest containing tool ID, owner, repo, canonical origin, environment URLs, supported launch modes, health-check URL, auth mode, minimum client version, status, required scopes and contact. Only an owner-approved URL on an allowlisted domain can be launched. Health checks are read-only, time-bounded and never imply security certification.

## Integration contracts

- **Portfolio:** public catalog and case studies; read-only. Portfolio never reads Clemo private workspace state.
- **Agent Workspace:** launches chat; optionally sends a minimal workspace ID and approved context handles via a short-lived launch assertion. Agent Workspace owns chats, provider keys, tool permissions and chat deletion.
- **Agent Labs:** opens experiments and trace/evaluation reports; only synthetic/public traces can be shared back to Clemo.
- **Future tools:** implement a versioned Tool Manifest and event contract; no product-specific assumptions in the shell.

## Security and tenancy

- Tenant/workspace membership enforced server-side on every request.
- Repository connection is per-user or per-workspace and explicit.
- Separate session and authorization boundary per product.
- No cross-product data sharing by default; require explicit grants and purpose.
- Signed webhooks, replay protection, idempotency keys and timestamp validation.
- Treat README, issue text, commit messages, uploaded files, webhook payloads and AI output as untrusted.
- No arbitrary Git URL cloning or server-side fetch from user-supplied URLs.
- Never store GitHub PATs or provider keys in plaintext.
- No production deployment automation until permission model, audit trail, approval and rollback are implemented.

## Privacy and trust

The earlier browser-first document-processing design remains relevant to Clemo's own document tools, but it does not automatically apply to all connected tools. Each tool must publish its own data-flow, retention, deletion and provider disclosure. "Clemo does not store files" must not be generalized to products that do store data.

## Roadmap

### Phase 0 — contract and shell
- Tool manifest schema, status model, domain allowlist, navigation and design system.
- Read-only static catalog seeded with accurate product states.
- No shared login or deployment controls yet.

### Phase 1 — workspace identity
- Auth, workspace membership, sessions, account settings and audit log.
- No cross-app SSO until security review.

### Phase 2 — integrations
- GitHub App connect/disconnect, repository metadata, webhooks and review queue.
- Agent Workspace and Agent Labs launch links; no cross-product private data.

### Phase 3 — workspace context
- User-approved context handles, scoped launch codes, recent activity and product events.
- Cross-app consent, deletion and audit flows.

### Phase 4 — deployment control plane (separate approval)
- Environment/deployment registry, preview lifecycle, build status, deploy approval, rollback.
- No arbitrary user code execution in the Clemo Worker.
- Prefer provider APIs and isolated build systems; define sandbox threat model and cost caps first.

## Documentation

- [Architecture](docs/ARCHITECTURE.md)
- [Workspace and integration contracts](docs/WORKSPACE.md)
- [Roadmap](docs/ROADMAP.md)
- [Security](SECURITY.md)
- [Decision: browser-first tools](docs/DECISIONS/0001-browser-first.md)
