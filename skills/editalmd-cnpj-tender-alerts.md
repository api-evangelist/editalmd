---
name: editalmd-cnpj-tender-alerts
description: Stand up no-signup new-tender alerts for a Brazilian company — by CNPJ (each CNAE activity becomes an alert family) or by explicit terms and UF — delivered by pull, signed webhook or email.
api: EditalMD API
generated: '2026-09-05'
method: generated
source: openapi/editalmd-openapi.json + https://editalmd.com/llms.txt
operations:
  - post_api_dono
  - post_api_alertas
  - get_api_cnaes
  - get_api_alertas
  - get_api_alertas_by_id_compras
  - patch_api_alertas_by_id
  - post_api_dono_confirmar_email
---

# Alert a company to new tenders

1. **Create an owner.** `POST /api/dono` (`post_api_dono`) — no signup. Save the `edm_…` token (shown exactly once) and the `whsec_…` webhook secret. 429 means the network's daily owner cap was hit.
2. **Create the alert.** `POST /api/alertas` (`post_api_alertas`) with `Authorization: Bearer edm_…`:
   - **By CNPJ:** send `cnpj` (+ optional `uf`, `max_familias`). Each CNAE activity of the company becomes a term family — one alert per family, main activity first. A CNAE outside the dictionary is listed with `familia: null` (422 when none maps: fall back to terms). Preview the dictionary free with `GET /api/cnaes` (`get_api_cnaes`).
   - **By terms:** send `termos` (space = AND, `|` = OR group) and optional `uf`.
   - The first active alert is free; each further alert costs $0.10 per 30 days — `402` with x402 `accepts[]`, or debit a `cred_…` token. CNPJ mode charges all families in one payment.
3. **Choose the channel.** `canal`: `pull` (default), `webhook` (`destino` = your URL), or `email` (`destino` must be confirmed via `POST /api/dono/confirmar-email`, `post_api_dono_confirmar_email` — 5 wrong codes returns 429).
4. **Consume matches.** Pull: `GET /api/alertas/{id}/compras` (`get_api_alertas_by_id_compras`). Webhook: verify Standard Webhooks headers (`webhook-id`, `webhook-timestamp`, `webhook-signature: v1,…`, HMAC-SHA256 of `id.timestamp.body` with the decoded `whsec_` key; reject timestamps older than 5 minutes; dedupe on `webhook-id`). The body carries `alerta_url` to cross-check by pull.
5. **Manage.** `GET /api/alertas` lists (`get_api_alertas`); `PATCH /api/alertas/{id}` (`patch_api_alertas_by_id`) pauses, reactivates or edits. DELETE is final — it also erases the match history.
