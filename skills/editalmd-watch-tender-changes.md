---
name: editalmd-watch-tender-changes
description: Watch a specific PNCP tender for rectifications, new documents, postponed deadlines and value changes, with a snapshot now and a verifiable event trail.
api: EditalMD API
generated: '2026-09-05'
method: generated
source: openapi/editalmd-openapi.json + https://editalmd.com/llms.txt
operations:
  - post_api_dono
  - post_api_vigias
  - get_api_vigias
  - get_api_vigias_by_id
  - get_api_vigias_by_id_eventos
---

# Watch a tender for changes

1. **Have an owner token.** `POST /api/dono` (`post_api_dono`) if you don't already hold an `edm_…` token.
2. **Start the watch.** `POST /api/vigias` (`post_api_vigias`) with `compra_id` and a channel (`pull`, `webhook` or confirmed `email`). It photographs the tender now (status, closing date, value, document list) and then records events. First watcher free; each further one $0.05 (`402` x402 / `cred_…` credit). 404 = tender not in the corpus.
3. **Read the events.** `GET /api/vigias/{id}/eventos` (`get_api_vigias_by_id_eventos`) — each event has `tipo` (`situacao`, `prazo_adiado`, `valor`, `documento_novo`, `documento_removido`, `prazo_impugnacao`, `prazo_proposta`), `antes`/`depois`, `visto_em`, `entregue_em`. The cron checks every 30 minutes. `GET /api/vigias/{id}` (`get_api_vigias_by_id`) returns the current snapshot.
4. **Webhook channel.** Same Standard Webhooks verification as alerts; the body carries `vigia_url` to cross-check by pull.
5. **Housekeeping.** `GET /api/vigias` lists watchers (`get_api_vigias`). DELETE is explicitly final ("Sem volta") — it erases the watcher and its event history, so export events first if you need the trail.
