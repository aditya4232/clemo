# Architecture (planned)

- `packages/<tool>`: one package per tool with its own tests and a README stating inputs, outputs, limits and privacy.
- Shared: `@adityacuz/ui` (tokens/components), `@adityacuz/session` (verify login via public keys), `@adityacuz/credits-client`.
- Browser-first: WASM/WebGPU for OCR and redaction where licences allow; server paths only for AI steps, behind the credits gateway.
- Uploads: size and type limits, scanned content treated as untrusted, deleted after processing.
- Each tool ships with sample inputs and a reproducible test set so quality claims can be checked.