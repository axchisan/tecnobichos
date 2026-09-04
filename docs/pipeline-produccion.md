# 🎬 Pipeline de producción (render de un short)

Cómo un tema `pendiente` se convierte en un MP4 9:16 listo para revisar. Todo vive
en el workflow **"Fase 1 — Hello Video"** (`2xACDbjcP0TQ6iTF`), que el **Productor**
dispara vía webhook cuando el EC2 está encendido y hay algo en cola.

## Cadena de nodos

```
Webhook → Config (voces) → Claim topic → [Groq guion] → Parse guion
   → Split escenas → Cloudflare image → Aggregate images
   → TTS → Build render payload → Render → Upload video → Sign URL → Update row
```

1. **Claim topic**: `UPDATE … SET estado='produciendo'` sobre 1 fila `pendiente`
   (con guarda `NOT EXISTS produciendo` → nunca procesa dos a la vez, por la RAM).
   Devuelve el `guion` ya escrito por la skill.
2. **Parse guion**: prefiere el `guion` de la skill; si no hay, cae al de **Groq**
   (fallback LLM). El guion tiene 3 escenas.
3. **Cloudflare image**: genera imágenes flat con Flux (fallback / portada).
4. **TTS** → llama al `media-worker`. Enruta por tipo de guion:
   - `guion.dialogo` presente → **`/tts-dialogue`** (debate, 2 voces por turnos).
   - todas las escenas con `narracion` → **`/tts-scenes`** (1 voz, timing por escena).
   - si no → **`/tts`** (1 voz, narración entera).
5. **Build render payload** (Code node): arma el JSON de render — imágenes (con su
   `gemini_prompt` o `b64`), `duration_sec` por escena (sincronía imagen↔charla),
   subtítulos ASS karaoke, overlays de mascotas, música por mood.
6. **Render** → `media-worker /render`: FFmpeg compone el video 9:16 con Ken Burns,
   transiciones, subtítulos quemados y música con ducking.
7. **Upload / Sign / Update**: sube el MP4 a Storage, firma la URL y marca
   `en_revision`.

## Tres mejoras clave del pipeline

- **Sincronía imagen↔charla**: `/tts-scenes` devuelve `start_sec/end_sec` por escena,
  y cada imagen dura exactamente su segmento hablado (antes se repartía a partes
  iguales y la imagen llegaba tarde).
- **Voces más fluidas**: SSML de Azure con `<mstts:silence>` recorta las pausas de
  coma y fin de frase (sonaban "trabadas, como pensando").
- **Formato debate**: las "peleas/versus" son diálogos por turnos con 2 voces y 2
  mascotas alternando esquinas (la que habla animada, la otra asintiendo).

## Imágenes: Gemini + fallback

- Las infografías se generan con **Gemini web** (vía `cc-browser`, gratis, sin las
  restricciones de pago de la API de imágenes).
- Si Gemini cae, hay **fallback a Cloudflare Flux** (imagen de respaldo `b64`) para
  no romper el render.
- **Gotcha resuelto**: cuando Google lanzó Gemini 3.x, la UI nueva metió la imagen en
  *shadow DOM* y el detector de `cc-browser` dejó de encontrarla. Se endureció para
  perforar shadow DOM y reconocer la imagen por su `alt="generada por IA"`/`blob:`.

## Restricción de RAM (regla de oro)

**Un video a la vez.** El `Claim topic` lo garantiza a nivel SQL. Los renders de
debate usan **speaker-only** (~8 overlays) en vez de hablante+oyente (~16) porque 16
overlays de FFmpeg excedían el `RENDER_TIMEOUT` y reventaban por OOM.

Ver [../services/media-worker/README.md](../services/media-worker/README.md) para los
endpoints del worker.
