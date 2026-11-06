# Architecture (planned)

- `packages/<tool>`: one package per tool with its own tests and a README stating inputs, outputs, limits and privacy.
- Shared: `@adityacuz/ui` (tokens/components), `@adityacuz/session` (verify login via public keys), `@adityacuz/credits-client`.
- Browser-first: WASM/WebGPU for OCR and redaction where licences allow; server paths only for AI steps, behind the credits gateway.
- Uploads: size and type limits, scanned content treated as untrusted, deleted after processing.
- Each tool ships with sample inputs and a reproducible test set so quality claims can be checked.
## Privacy rules
- Redaction and document parsing happen in the browser first; only redacted text goes to the AI step.
- No document text in logs; uploads are size/type limited, treated as untrusted, deleted after the job.
- Paperwork Pilot answers must cite spans from the input; answers without evidence are rejected.

## Flow engine
Each flow is a fixed pipeline of typed steps (ingest, plan, extract, verify, assess, explain, export). The planner chooses among known steps; it cannot invent tools. Extraction uses strict JSON schemas, and the verify step rejects any value not found in the source text. Runs emit traces without content. Jobs run in Worker memory; any temporary state lives in a Durable Object with an alarm that deletes it within 15 minutes. Upload cap about 10 MB; larger files are rejected before any server call.

## Load assumptions
100-500 visitors/day, 50-80 active. Per-run token caps, a global daily cap, and 10 credits/user/day (Paperwork 3, Scam Check 1, FormFit 1, Split 2, Redact 0).
