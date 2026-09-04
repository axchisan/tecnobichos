# 🎬 Tecnobichos

**Canal de contenido tech (programación, IA y cultura dev) 100% automatizado.**
Shorts 9:16 animados con mascotas, estilo caricaturesco y juvenil, pero educativos y
rigurosos. El sistema descubre temas, escribe guiones, produce el video, lo publica
en **YouTube** e **Instagram** y gestiona todo el estado — con un único paso manual:
aprobar por WhatsApp.

Elenco: **Bit** 🔵 (anfitrión), **Chip** 🟠 (veterano gruñón), **Sonda** 🟣 (dron
reportera), **Tera** ⬜ (IA sabia). Canal: [@tecnobichos94](https://www.instagram.com/tecnobichos94/).

---

## 📚 Documentación

| Doc | De qué trata |
|---|---|
| **[docs/publicacion-youtube.md](docs/publicacion-youtube.md)** | ⭐ **Publicación automática en YouTube** — flujo, nodo n8n, y fundamentos de la YouTube Data API v3 (OAuth2, `videos.insert`, cuotas). |
| [docs/publicacion-instagram.md](docs/publicacion-instagram.md) | Publicación de Reels con la Instagram Graph API. |
| [docs/arquitectura.md](docs/arquitectura.md) | Visión general, stack y componentes. |
| [docs/pipeline-produccion.md](docs/pipeline-produccion.md) | Cómo se renderiza un short (guion → imágenes → voz → FFmpeg). |
| [docs/workflows-n8n.md](docs/workflows-n8n.md) | Los 10 workflows de n8n y qué hace cada uno. |
| [docs/setup.md](docs/setup.md) | Despliegue y variables de entorno. |

> 👉 Si vienes a entender **cómo publicamos en YouTube por API**, empieza por
> [docs/publicacion-youtube.md](docs/publicacion-youtube.md).

---

## 🗺️ Estructura del repo

```
tecnobichos/
├── docs/                    # Toda la documentación (empieza aquí)
├── services/
│   ├── media-worker/        # Microservicio FFmpeg + TTS (render de video)
│   └── cc-browser/          # Microservicio Playwright (imágenes con Gemini web)
├── skills/
│   └── tecnobichos-temas.md # Skill que genera temas + guiones
├── brief/
│   ├── brief-maestro.md     # Brief maestro del proyecto (visión, reglas)
│   ├── CANAL.md             # Biblia del canal (marca, elenco, lore)
│   └── ROADMAP.md
└── workflows/               # (exports JSON de n8n, cuando aplique)
```

---

## 🔄 Cómo funciona, en una frase

```
skill genera guiones → Productor (n8n) renderiza en un EC2 bajo demanda
→ WhatsApp para aprobar → YouTube Data API + Instagram Graph API publican solos
```

Máquina de estados en Supabase:
```
pendiente → produciendo → en_revision → en_aprobacion → aprobado → publicado
                                                      ↘ rechazado
```

---

## 🧩 Los dos microservicios

- **media-worker** ([README](services/media-worker/README.md)) — FastAPI + FFmpeg +
  edge-tts/Azure. Endpoints `/tts`, `/tts-scenes`, `/tts-dialogue`, `/render`,
  `/health`. Compone el video 9:16 con subtítulos karaoke, mascotas y música.
- **cc-browser** ([README](services/cc-browser/README.md)) — FastAPI + Playwright.
  Genera infografías con Gemini web reutilizando la sesión logueada del dueño
  (evita el costo de la API de imágenes). Endpoint `/gen-image`.

---

## 🔐 Seguridad

Este repo **no contiene secretos**. Todas las claves (Supabase service_role, tokens
de YouTube/Instagram, Azure, etc.) van por **variables de entorno** o en la tabla
`cc_config` de Supabase. Ver [docs/setup.md](docs/setup.md).

---

## 🛠️ Stack

n8n · Supabase (Postgres + Storage) · FFmpeg · edge-tts / Azure Speech · Gemini web ·
Cloudflare Workers AI (Flux) · YouTube Data API v3 · Instagram Graph API ·
Evolution API (WhatsApp) · AWS EC2 + Coolify · Docker.
