# ⚙️ Setup / despliegue

Guía resumida para levantar las piezas. Los **secretos van por variables de
entorno** — este repo no incluye ninguno.

## 1. Supabase (estado + storage)

Tabla principal `cc_cola_contenido` (estados: `pendiente → produciendo → en_revision
→ en_aprobacion → aprobado → publicado / rechazado`), más `cc_config` (voces, tokens
de IG, sesión de Gemini) y `cc_logs`. Buckets: `cc-videos`, `cc-assets`, `cc-mascota`.

Acceso solo por la API de Kong (no Postgres directo):
- SQL/DDL: `POST /pg/query` (pg-meta) con header `apikey` + `Authorization: Bearer <service_role>`.
- CRUD: PostgREST `/rest/v1/<tabla>`.
- Storage: `/storage/v1/...`.

## 2. Servicios en AWS (Coolify)

Ambos se despliegan como contenedores Docker vía Coolify, en un EC2 que se enciende
bajo demanda.

### media-worker (`services/media-worker`)
Variables de entorno:
```
MEDIA_WORKER_API_KEY   # auth del propio worker (header X-API-Key)
AZURE_SPEECH_KEY, AZURE_SPEECH_REGION   # TTS Azure (voces multilingües)
ELEVENLABS_KEY         # (opcional) TTS alternativo
PEXELS_KEY             # (opcional) fotos stock
BROWSER_URL, BROWSER_API_KEY   # apuntar a cc-browser
SUPABASE_URL, SUPABASE_SERVICE_KEY   # backup de imágenes
ALERT_URL              # webhook cc-alert
RENDER_TIMEOUT=480     # tope duro de FFmpeg (s)
```

### cc-browser (`services/cc-browser`)
```
BROWSER_API_KEY        # auth
SUPABASE_URL, SUPABASE_SERVICE_KEY   # lee la sesión de Gemini de cc_config
STORAGE_STATE_JSON_B64 # (o cc_config gemini_session) sesión de Gemini logueada
ALERT_URL
```
La sesión de Gemini se captura logueado en `gemini.google.com` y se mantiene viva
con una extensión de Chrome; `cc-browser` la recarga de la DB en cada llamada.

## 3. n8n

Importar los workflows y reconectar credenciales:
- **YouTube** (`youTubeOAuth2Api`): client_id/secret de Google Cloud + "Sign in with
  Google" en la UI (una vez). Ver [publicacion-youtube.md](publicacion-youtube.md) §4.
- **Supabase service headers** (httpCustomAuth): `apikey` + `Authorization: Bearer`.
- **Evolution API** (WhatsApp), **Cloudflare Workers AI**, **Groq**.

## 4. AWS EC2 encendido/apagado

EventBridge Scheduler:
```
tecnobichos-start = cron(0 22 ? * SUN,WED *)   # Dom/Mié 10pm (America/Bogota)
tecnobichos-stop  = cron(0 2 ? * MON,THU *)    # Lun/Jue 2am
```
El EC2 tiene Elastic IP (apagar/encender no cambia la IP). Tras reiniciar, Coolify
puede necesitar **revalidar el servidor** antes de desplegar.

## 5. Operación diaria (para el dueño)

1. **Noche (Dom/Mié):** correr la skill `tecnobichos-temas` → inserta ~10-12
   guiones `pendiente`. El Productor los renderiza cuando el EC2 arranca a las 10pm.
2. **Día:** aprobar por WhatsApp los videos que llegan (franjas 12/5/9pm). Cada
   aprobado se publica solo en YouTube + Instagram.
