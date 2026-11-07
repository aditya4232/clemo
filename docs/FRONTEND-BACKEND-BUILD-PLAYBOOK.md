# Clemo Frontend and Backend Build Playbook

**Status:** implementation contract, not shipped application code. Clemo P0 runtime design/plan remains the first bounded implementation slice.

## Product experience

Clemo is a calm, trustworthy workspace shell and launcher for independent products—not a monolithic SaaS app and not an AI chat clone. Users need to understand what a tool does, what data it accesses, what it costs, what permissions they are granting, and whether a run is connected, launching, complete or failed. Mobile-first, fast, accessible, clear recovery and transparent credits are more important than visual spectacle.

Core surfaces: public landing/catalog; sign-in and onboarding; workspace switcher; tool catalog/detail; connected accounts; launcher; activity/status; credit balance/ledger/top-up; subscription/billing; support; profile/security; and a separately permissioned operations admin. Keep advanced technical details progressive.

## Frontend direction

- Next.js App Router + React + strict TypeScript + Tailwind CSS v4 tokens on Vercel, independent of the API Worker.
- Server Components by default; client islands for workspace switching, catalog filters, launch state, ledger interactions and support forms.
- Accessible primitives; use shadcn/ui patterns/Radix selectively rather than a wholesale component kit.
- Motion for React for small interaction feedback only. No Three.js/GSAP on the core workspace shell unless a specific user-facing workflow benefits and performance remains within budget.
- Zod for client-safe schemas; generated/validated API types from OpenAPI. No Cloudflare bindings in browser bundles.
- Vitest + React Testing Library, Playwright, axe-core and visual screenshot review.

Use a restrained product UI: strong type hierarchy, quiet surfaces, compact but legible data, clear status semantics, meaningful empty states, visible permission scopes, explicit pricing units and predictable navigation. Avoid nested cards, excessive gradients, decorative charts and fake activity.

## Taste Skill and Impeccable

In the frontend repository root:

```bash
npx skills add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend"
npx impeccable install
```

Reload Claude Code after installation. Run `/impeccable init` and create PRODUCT.md / DESIGN.md. Enable hooks only after reviewing permissions: `/impeccable hooks on`. Use Taste Skill for initial visual direction; use Impeccable for critique, audit, polish, responsive behavior, accessibility and anti-pattern detection. Do not let both independently redefine the same product UI. This playbook and the Clemo product contract override generic skill defaults.

Impeccable commands to use selectively: `shape`, `craft`, `critique`, `audit`, `polish`, `layout`, `typeset`, `harden`. Check the installed version's command reference. Taste Skill browser checks require Node 22+ and Chrome/Edge/Chromium. Sources: https://github.com/Leonxlnx/taste-skill, https://www.tasteskill.dev/docs/, https://impeccable.style/docs/.

## Figma recommendation

Use Figma for design tokens and a small set of representative screens: catalog, tool detail/permission consent, workspace home/activity and credit ledger. Establish desktop/mobile layouts and component variants, then implement the real responsive UI. Do not spend time designing every admin edge case upfront. Keep components editable and reusable; do not use flattened screenshot UI.

## Backend foundation

- TypeScript + Hono on Cloudflare Workers; Node 22 local tooling; Zod environment and request validation; OpenAPI contract; Vitest.
- Pure app factory separated from Worker entrypoint; typed environment bindings; readiness/liveness endpoints with sanitized output.
- D1 + Drizzle only when P0 foundation is verified and migrations are designed; KV only for eventually consistent read-mostly config; R2 for user artifacts; Queues/Workflows for async; Durable Objects only for measured serialization requirements.
- No Redis, Postgres, payments, auth provider, production integrations or extra paid services in P0.
- Keep API under versioned `/v1`; stable error envelope; request IDs; explicit timeouts and payload caps.

## Ownership and contracts

Clemo owns identity/workspaces/memberships, catalog/manifest registry, connection metadata, wallet ledger, orders/subscriptions, support, admin and audit. Agent Workspace owns chats, provider keys, MCP grants, run state and artifacts. Agent Labs owns datasets, experiments and evidence. Portfolio owns public content. No shared database, broad parent-domain cookies or implicit context sharing.

Tool catalog distinguishes repository existence, implementation readiness, deployment existence, health status and user authorization. A manifest is untrusted input: validate schema, canonical origin, environment origins, owner proof, capabilities, scopes, data class, health endpoint and lifecycle state. A tool being listed is not proof it is deployed or authorized.

## Security and commerce constraints

- Auth and tenant authorization are server-side; enforce object-level membership on every resource lookup/write.
- Host-only secure cookies, CSRF protection for cookie-authenticated writes, exact origin allowlists, no wildcard credentialed CORS.
- GitHub App read-only initially; explicit repository selection; webhook HMAC, timestamp/replay window, event idempotency and bounded retries.
- Wallet is append-only ledger; quote → reserve → execute → capture/release. No mutable balance as source of truth. Every external side effect is idempotent.
- Payment webhook raw-body signature validation, provider-event uniqueness, reconciliation and compensating refund entries. Browser redirects do not prove payment. Never store PAN/CVV/UPI PIN.
- No production payment activation or live payment credentials until legal, tax, RBI/PPI classification, GST, recurring consent, cancellation/refund and grievance review plus explicit owner approval.
- Audit admin actions; default deny; redact secrets and personal data from logs.

## Verification and delivery

CI must execute real deterministic checks; do not report a green docs-only workflow as application validation. P0: scaffold, env validation, health routes, OpenAPI, local Wrangler and CI. Then auth/tenant boundary; manifest registry; wallet ledger; signed usage settlement; catalog and workspace UI; payment sandbox only after review.

For each UI slice: test behavior first, run focused tests, implement, run full suite, inspect browser at mobile and desktop, check keyboard/reduced-motion/empty/loading/error states, run Impeccable detector and Playwright. For each API slice: test auth, tenant isolation, schema failures, idempotency, replay, timeout and error redaction. Report exact commands and outcomes.

Performance must be measured at p50/p95/p99 with workload and sample count. Do not claim sub-second performance from local intuition. Respect ₹600/month target and ₹1,000/month hard ceiling inclusive tax/FX. No paid tier, production deploy, DNS, secrets or data mutation without approval.
