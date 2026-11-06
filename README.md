# Clemo

**Small free tools for everyday paperwork and privacy.** One job per tool, no paywalls, works on a phone, and your files stay on your device whenever the browser can do the job.

Live (planned): **clemo.adityacuz.in** · Part of [adityacuz.in](https://adityacuz.in) · MIT licensed

> **Status: design phase.** The launch set below is a hypothesis to be validated with real usage. Nothing is built yet.

## Launch set
| Tool | For | What it does | Runs |
|---|---|---|---|
| **Redact** | anyone sharing ID copies or statements | Masks Aadhaar/PAN/phone/account numbers and faces before you share a file | fully in your browser |
| **FormFit** | applicants uploading to government or exam portals | Paste the portal's instructions; it extracts the size and format rules and fits your photo, signature and PDFs, with a pass/fail check | in browser; AI only reads the instruction text |
| **Paperwork Pilot** | rent agreements, bank letters, policies, notices | Plain-language summary, key dates and amounts, unusual clauses and questions to ask, each linked to the exact line it came from | redaction in browser, AI via gateway |
| **Scam Check** | fake job offers, payment requests, phishing SMS | Explains the risk signals in a message or link and what to do; signals, not a verdict | rules + small model, link checks server-side |
| **Split the Bill** | friends, roommates, trips | Receipt photo to itemised table to who owes whom, UPI-ready | OCR in browser, AI fallback |

Build order: Redact and FormFit first (client-side, no AI cost), then Paperwork Pilot and Scam Check once usage shows demand. Developer-oriented tools (MCP Playground, GitHub Roast & Fix) come later on a separate shelf.

## Principles
- **Plain language, one job per tool, stated limits.**
- **Browser-first.** If it can run locally it does; files never leave your device.
- **No retention.** Anything processed server-side is deleted after the job; each tool page says what is kept.
- **Show the evidence.** AI answers point at the part of your document they came from.
- **Honest limits.** Not legal advice; scam results are signals; accuracy is measured and published.
- **Licence-clean.** Every model and library is checked before use.

## Architecture
Monorepo, one package per tool, shared UI and session packages from `@adityacuz/*`. See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) and [docs/ROADMAP.md](docs/ROADMAP.md).

[SECURITY.md](SECURITY.md) · [CONTRIBUTING.md](CONTRIBUTING.md) (design phase: open an issue first)