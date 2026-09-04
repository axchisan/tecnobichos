# 📸 Publicación automática en Instagram (Reels)

> Flujo hermano del de YouTube. Sube Reels a Instagram con la **Instagram Graph API**
> (Instagram Login / cuenta MEDIA_CREATOR). Workflow `9xcLWJ13DmChF3oI`.

---

## 1. Cuándo se dispara

Una vez un video está **`publicado`** (ya subió a YouTube), un cron cada 10 min
busca los que aún no tienen `instagram_url` y los sube a IG. Así YouTube e IG
quedan sincronizados y en el mismo orden.

```
GET /rest/v1/cc_cola_contenido?estado=eq.publicado&instagram_url=is.null&order=publicado_en&limit=1
```

---

## 2. La Graph API en 3 pasos

Instagram no sube el binario directamente: recibe una **URL pública** del video,
lo procesa asíncrono, y luego publicas el "contenedor".

```
# 1. Crear contenedor (le pasas una URL pública del MP4):
POST https://graph.instagram.com/v21.0/<IG_USER_ID>/media
  ?media_type=REELS
  &video_url=<URL_PUBLICA_DEL_MP4>
  &caption=<texto + hashtags>
  &access_token=<TOKEN>
  → { "id": "<CONTAINER_ID>" }

# 2. Sondear hasta que IG termine de procesar el video:
GET https://graph.instagram.com/v21.0/<CONTAINER_ID>?fields=status_code&access_token=<TOKEN>
  → status_code: IN_PROGRESS → … → FINISHED   (o ERROR / EXPIRED)

# 3. Publicar el contenedor ya procesado:
POST https://graph.instagram.com/v21.0/<IG_USER_ID>/media_publish
  ?creation_id=<CONTAINER_ID>
  &access_token=<TOKEN>
  → { "id": "<MEDIA_ID>" }   ← ya está en el perfil
```

---

## 3. El workflow (versión robusta)

Nodos: `Buscar sin IG` → `Cfg IG` (lee token/user_id de `cc_config`) → `Prep` →
**`Re-firmar URL`** → **`IG publicar`** (Code node) → `Marcar IG`.

- **Re-firmar URL**: vuelve a firmar la signed URL de Supabase Storage antes de
  enviarla, porque las URLs firmadas expiran a los 7 días y IG necesita descargar
  el video en el momento.
- **IG publicar** (Code node): crea el contenedor, **sondea `status_code` hasta
  `FINISHED`** (15 intentos × 10 s), y publica. **Lanza error si algo falla** → el
  video NO se marca, se dispara alerta y se reintenta en el próximo tick.
- **Marcar IG**: solo se ejecuta si la publicación tuvo éxito real → guarda
  `instagram_url` con el `MEDIA_ID`.

---

## 4. Bugs que tuvo (y la lección)

La primera versión tenía un **`Wait 45s` fijo** + `onError: continue` en todos los
nodos. Resultado:

1. IG no siempre termina de procesar el Reel en 45 s → `media_publish` fallaba.
2. Con `onError: continue`, el workflow **marcaba `instagram_url` igual** aunque la
   subida fallara → **falsos positivos**. La DB decía "16 subidos" pero en IG solo
   había 9.

**Lecciones aplicadas:**
- **Nunca esperar un tiempo fijo** para un proceso asíncrono: **sondea el estado
  real** (`status_code === 'FINISHED'`).
- **No marcar éxito si no lo hubo:** que el paso de publicación **lance error** en
  vez de tragárselo, para no crear falsos positivos.

---

## 5. Notas y limitaciones

- **Token de larga duración** (~60 días): guardado en `cc_config` clave `instagram`
  = `{ token, user_id }`. Hay que refrescarlo antes de que expire.
- La cuenta es tipo **MEDIA_CREATOR** (`tecnobichos94`), requisito para la Graph API
  de contenido.
- ⚠️ **IG NO permite borrar publicaciones por API** (limitación de Meta) → si
  re-subes contenido viejo, queda arriba en el grid; el orden perfecto solo se logra
  borrando manualmente y re-subiendo.
- `$helpers` **no está definido** en el Code node de esta versión de n8n (2.22.5);
  se usa detección multi-variante (`$helpers` / `this.helpers` / `helpers`).

---

## 6. Referencias

- Instagram Content Publishing API: https://developers.facebook.com/docs/instagram-platform/content-publishing
- Reels specs: https://developers.facebook.com/docs/instagram-platform/content-publishing#reels

Ver [publicacion-youtube.md](publicacion-youtube.md) para el flujo principal.
