# Clemo

**Upload anything, get the job done.** Free, AI-assisted flows for everyday paperwork and privacy. Your files are not stored.

Live (planned): **clemo.adityacuz.in** · Part of [adityacuz.in](https://adityacuz.in) · MIT licensed

> **Status: design phase.** Everything below is the plan; nothing is built yet.

## How it works
Drop a file, paste text, or paste a screenshot into **Smart Drop**. It recognises what you gave it and picks the right flow (you can override). Each flow is several steps, not one prompt:

`read in your browser -> plan -> extract -> verify against your document -> assess -> explain -> export`

Every extracted fact must point to the exact place in your document it came from, or it is thrown away.

## Flows
| Flow | You give | You get |
|---|---|---|
| **Paperwork Pilot** | rent agreement, bank letter, policy, notice, job offer | plain-language summary, key dates and amounts, unusual clauses, questions to ask, calendar reminders, a draft reply letter, answers with citations |
| **Scam Check** | suspicious message, link, screenshot | why it looks risky, safe link analysis, what to do and where to report |
| **FormFit** | a portal's size/format instructions + your photo, signature, PDFs | files resized and checked against each rule, with a pass/fail report |
| **Redact** | ID copies, statements | masked numbers and faces, metadata stripped, verified before download |
| **Split the Bill** | receipt photo | itemised table, who owes whom, UPI-ready message |

## Privacy
- Files are read and processed in your browser first; only extracted text or a downsized image is sent for AI steps.
- **Nothing is saved**: no file storage, no document text in logs. Server work happens in memory and is discarded.
- AI answers are labelled; they are not legal, financial or medical advice. Scam results are signals, not verdicts.

## Quality
Each flow has a labelled test set and an automated evaluation; published accuracy comes only from those runs, along with the known failure cases.

## Docs
[Architecture](docs/ARCHITECTURE.md) · [Roadmap](docs/ROADMAP.md) · [Security](SECURITY.md) · [Contributing](CONTRIBUTING.md) (design phase: open an issue first)