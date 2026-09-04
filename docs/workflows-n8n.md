# 🔀 Workflows de n8n

Todos los workflows llevan el tag `Tecnobichos` en la instancia
`n8n.axchisan.com`. IDs para referencia rápida.

| Workflow | ID | Trigger | Rol |
|---|---|---|---|
| **Fase 1 — Hello Video** (pipeline de render) | `2xACDbjcP0TQ6iTF` | Webhook `cc-run-pipeline` | Produce 1 short: tema → guion → imágenes → voz → render → `en_revision`. Ver [pipeline-produccion.md](pipeline-produccion.md). |
| **Productor — Auto-render nocturno** | `CDoBAHOu1gLGZzO0` | cron `*/10 * * * *` | Si el `media-worker` responde `/health` **y** hay `pendiente`, dispara el pipeline. Si el EC2 está apagado, no hace nada (sin error). |
| **Enviar a aprobación** | `FYJ9eS3ce2MedS2M` | cron `0 12,17,21 * * *` | Manda 1 video `en_revision` por WhatsApp con botones aprobar/rechazar → `en_aprobacion`. |
| **Aprobación (webhook)** | `W9bJlOSSovAoLQJE` | `GET /webhook/cc-approve?id&a` | `a=1`→`aprobado`, `a=0`→`rechazado`. Responde HTML. |
| **Publicar YouTube** | `sKCKn6aiYFYWj74J` | cron 5 min | Sube 1 `aprobado` a YouTube → `publicado`. Ver [publicacion-youtube.md](publicacion-youtube.md). |
| **Publicar Instagram** | `9xcLWJ13DmChF3oI` | cron 10 min | Sube 1 `publicado` sin `instagram_url` a IG. Ver [publicacion-instagram.md](publicacion-instagram.md). |
| **Set miniatura YouTube** | `v2TrbfL8MzLXuB8F` | webhook | Sube thumbnail 16:9 vía `thumbnails.set`. |
| **Recordatorio — Generar guiones** | `j6ATFH9cOftgkoKW` | cron `0 20 * * 0,3` | Dom/Mié 8pm: WhatsApp + email con inventario; avisa si quedan pocos videos. |
| **cc-alert — Log + WhatsApp** | `wrWqbkELdoprL5AC` | Webhook `cc-alert` | Alerta central: escribe en `cc_logs` + WhatsApp si es error/crítico. |
| **Alertas — Error Handler** | `ZK0RPwJdXyatukam` | Error Trigger | `errorWorkflow` de los demás → manda a `cc-alert`. |

## Convenciones importantes

- **PostgREST con `Accept: application/vnd.pgrst.object+json`**: n8n a veces entrega
  la respuesta como **string bajo `.data`** → hay que `JSON.parse` en el Code node.
- **Credenciales por env / referencia**: los workflows referencian credenciales de
  n8n por id/nombre; los secretos **nunca** están en el JSON del workflow.
- **Un ítem a la vez**: tanto el render (`Claim topic` con guarda SQL) como los
  publicadores procesan de a uno por tick, para respetar RAM y rate limits.
- **cron de n8n = 5 campos** (no el formato AWS de 6 con `?`).

## Reimportar

Los JSON exportados (cuando se incluyan en `/workflows`) se importan en
n8n → *Import from File*. Después hay que **reconectar las credenciales** (YouTube
OAuth, Supabase headers, Evolution, Cloudflare) porque no viajan en el export.
