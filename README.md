# Clemo

**Small free tools that do one thing well.** No sign-up walls, no paywalls, and your files stay with you whenever the browser can do the job.

Live (planned): **clemo.adityacuz.in** · Part of [adityacuz.in](https://adityacuz.in) · MIT licensed

> **Status: design phase.** The tool specs below are decided; none is built yet.

## The launch set
| Tool | What it does | Runs | Credits |
|---|---|---|---|
| **Redact** | Find and black out personal data (names, numbers, emails) in text, PDFs and screenshots before you share them | in your browser | 0 |
| **FormFit** | Fit your details into messy PDF forms and flatten the result | in your browser | 0 |
| **GitHub Roast & Fix** | Review a public repo and propose concrete fixes (README, CI, security basics) with diffs | server + AI | metered |
| **Invoice to Sheet** | Turn invoice PDFs/photos into a clean table (CSV/XLSX) | browser OCR first, AI fallback | metered |
| **MCP Playground** | Connect to an MCP server and try its tools safely, with a log of every call | server + policy | metered |

Why these: each solves a task people currently pay for or hand to sketchy sites, and each is a small enough scope to ship, test and explain.

## Principles
- **One job per tool**, a plain description, a stated limit.
- **Local by default.** If it can run in your browser, it does, so files never leave your device.
- **No retention.** Server-processed files are deleted after the job; each tool page says exactly what is kept.
- **Licence-clean.** Every model and library is checked before use (e.g. non-commercial models are excluded).
- **Honest AI.** AI-assisted outputs are labelled; you review before using.

## Architecture
Monorepo, one package per tool, shared UI and session packages from `@adityacuz/*`. See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) and [docs/ROADMAP.md](docs/ROADMAP.md).

## Security and contributing
[SECURITY.md](SECURITY.md) · [CONTRIBUTING.md](CONTRIBUTING.md) (design phase: open an issue first).