# ADR 0002: Clemo is a workspace shell, not a monolith

- Status: Accepted as target architecture; implementation pending.
- Date: 2026-11-07.

## Context
Clemo originally described browser-first document tools. The ecosystem now includes separate agent chat and agent research products. Folding those implementations into one repo would couple release cycles, permissions and data lifecycle.

## Decision
Clemo owns workspace identity, tool catalog, repository connections, approved deployment references and launcher. Its own document workflows remain one product module. Agent Workspace owns chat/runtime; Agent Labs owns evaluation/research. Each product owns its data and deployment.

## Consequences
- Versioned tool manifest and event envelope are required.
- Cross-product data is not shared by default.
- Launch links use independent authentication until secure SSO is implemented.
- GitHub integration starts read-only with review queue.
- Arbitrary build/deploy execution is out of initial scope.
