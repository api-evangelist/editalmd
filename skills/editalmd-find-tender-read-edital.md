---
name: editalmd-find-tender-read-edital
description: Find a Brazilian public tender on PNCP via EditalMD, check its deadlines, and read the edital as markdown with provenance — paying only when the tender is recent.
api: EditalMD API
generated: '2026-09-05'
method: generated
source: openapi/editalmd-openapi.json + https://editalmd.com/llms.txt
operations:
  - get_api_busca
  - get_api_compra_by_id
  - get_api_documento_by_id_markdown
  - post_api_documento_by_id_habilitacao
  - get_api_recibo_by_id
---

# Find a tender and read its edital

1. **Search free.** `GET /api/busca?q=<terms>&uf=<UF>&abertas=1` (`get_api_busca`). Space between words requires all of them; `|` inside a group accepts any (`uniforme|fardamento escolar`). Terms under 3 letters return 400. Always free.
2. **Open the sheet.** `GET /api/compra/{id}` (`get_api_compra_by_id`) returns the tender, its documents with per-document price and regime, and the deadlines: `prazos.proposta_ate` (from PNCP) and `prazos.impugnacao_ate` (estimated, with `base_legal`). Free.
3. **Read the edital.** `GET /api/documento/{id}/markdown` (`get_api_documento_by_id_markdown`). Tenders published 30+ days ago are free; a recent one answers `402` with an x402 `accepts[]` offer — pay (USDC) and repeat with the `X-PAYMENT` header, or send a prepaid `Authorization: Bearer cred_…` token. The markdown carries provenance front-matter and its SHA-256.
4. **Extract eligibility (optional, paid on recent documents).** `POST /api/documento/{id}/habilitacao` (`post_api_documento_by_id_habilitacao`) returns the habilitação checklist grouped legal / fiscal / economic-financial / technical, each requirement with its literal excerpt. The same extracted text (`sha256_texto`) is never charged twice.
5. **Keep the receipt.** Every delivery returns `x-editalmd-recibo` and `x-editalmd-sha256`; verify at `GET /api/recibo/{id}` (`get_api_recibo_by_id`).

Rules: 404 and 409 are never charged. Errors come as `{"error": "<slug>"}` — see `errors/editalmd-problem-types.yml`.
