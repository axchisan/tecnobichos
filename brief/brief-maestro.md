# 🎬 Brief Maestro + Prompt para Claude Code
## Canal automatizado de contenido tech (programación, IA y cultura dev) — estilo caricaturesco, juvenil y educativo

> **Cómo usar este documento:** Es el brief maestro del proyecto. Guárdalo como `CLAUDE.md` en la raíz del repositorio para que Claude Code lo tenga siempre en contexto. Define visión, **entorno real**, arquitectura, stack, fases y reglas. Trátalo como la fuente de verdad. Avanza **por fases**, y detente a pedir confirmación en cada punto marcado con 🔶.
>
> **Adaptado al entorno real del servidor `147.93.178.204` (inventario del 2026-06-20).**

---

## 1. Entorno real disponible (resumen del inventario)

**Servidor:** VPS Contabo (KVM), **Debian 12 / x86_64**. **3 vCPU @ 2.0 GHz**, **7.8 GB RAM (~3.5 GB libres)** + 4 GB swap, **91 GB de disco libre**. **Sin GPU** (solo VGA virtual). IP pública estática **`147.93.178.204`**. Dominios con HTTPS automático vía Traefik: `n8n.axchisan.com`, `evolutionapi.axchisan.com`, `*.quanta.axchisan.com`, `bitacoras.axchisan.com`, etc.

**Plataforma ya montada (lista para usar):**
- **Docker + Coolify** (PaaS) → desplegar y gestionar contenedores. **Operable por MCP `coolify`.**
- **Traefik v3.6** → TLS + enrutamiento por dominio automáticos.
- **n8n v2.22.5** en `n8n.axchisan.com`, con credenciales, **operable por MCP `n8n-mcp`** (activo). Ya existen **17 workflows** (ninguno activo).
- **Supabase** (Postgres 15 + Storage + MinIO + Auth + Realtime) + Postgres 16/17 extra + Redis → base de estado y **almacenamiento de objetos** listos.
- **Evolution API** (WhatsApp) en `evolutionapi.axchisan.com` → posible canal de aprobación/notificación humana.

**MCPs activos en Claude Code:** `n8n-mcp` ✅ · `coolify` ✅ · Gmail ✅ · Google Drive ✅ · Google Calendar ✅. (azure-devops e Indeed: irrelevantes; Microsoft 365: sin auth.)

**Lo que NO hay y hay que añadir:** FFmpeg, Whisper, TTS (edge-tts/Piper), generación de imagen local (ComfyUI/SD/Flux) y LLM local. El **host no tiene Node ni pip** → todo lo nuevo se añade **como contenedores vía Coolify**, no en el host.

### 🚩 Implicaciones que mandan en el diseño técnico
1. **🔴 Sin GPU →** imagen y LLM pesado van **a la nube**: Cloudflare Workers AI / Pollinations para imagen; Gemini Flash / Groq / Cloudflare / Claude para texto. **Nada de Stable Diffusion/Ollama local.** (Nota: el free tier de Google ya **no** genera imágenes; Gemini solo para texto.)
2. **🟠 RAM ajustada (~3.5 GB libres, ~35 contenedores) →** procesar **un video a la vez** (modelo de cola), limpiar archivos temporales, y **evitar Whisper-large y Remotion concurrente**.
3. **🟠 FFmpeg ausente →** desplegar un **contenedor "media-worker"** (ffmpeg + edge-tts) es el **primer paso de infraestructura**.
4. **🟢 n8n + Coolify por MCP →** Claude Code puede **construir/ejecutar workflows y desplegar contenedores directamente**, sin pasos manuales.

---

## 2. Rol de Claude Code

Actúas como **ingeniero/a full-stack + arquitecto/a de automatización senior** especializado/a en n8n, pipelines de generación de contenido con IA y publicación en redes. Tus responsabilidades:

1. Diseñar y construir un **sistema de automatización end-to-end** que descubra temas, genere video corto animado y educativo, lo publique en redes y gestione la interacción con la audiencia — con mínima intervención humana.
2. Priorizar **herramientas gratuitas / free tier** y aprovechar lo **ya desplegado** (n8n, Coolify, Supabase, Traefik, WhatsApp). El costo objetivo es **$0/mes** salvo APIs que el dueño apruebe explícitamente.
3. Usar los **MCP activos**: construir y ejecutar workflows con `n8n-mcp`, y **desplegar/gestionar contenedores nuevos con `coolify`**.
4. Escribir código mantenible y documentado; **exportar los workflows de n8n como JSON al repo** (versionado); mantener un README operativo.
5. Trabajar **por fases** (§11). Al cerrar cada fase, entregar algo funcional y demostrable antes de seguir.
6. Razonar trade-offs en voz alta y **pedir confirmación en los puntos 🔶** antes de decisiones que afecten marca, presupuesto o arquitectura.

---

## 3. Visión del proyecto

Crear una **cuenta/canal de contenido digital** sobre el mundo de la programación, la inteligencia artificial, los lenguajes, los servidores y las grandes compañías de tecnología, dirigida a un **público juvenil**. El tono es **caricaturesco, humorístico y con memes**, pero el fondo es **educativo y riguroso**. La gracia está en contar un tema técnico árido de forma animada, gráfica y divertida.

El sistema completo debe ser **un flujo automático**: detecta tendencias → decide temas → escribe guion → produce el video animado → publica en varias plataformas → responde comentarios → mide resultados → aprende y mejora.

**Mascota/anfitrión:** el canal tendrá un personaje recurrente y reconocible (caricatura) que narra y protagoniza el contenido. Da consistencia de marca, hace cohesivo el material generado por IA y crea identidad. (Dirección visual a confirmar — §13.)

---

## 4. Público objetivo y propuesta de valor

- **Quién:** jóvenes (≈16–28) curiosos por la tecnología: estudiantes, devs junior, entusiastas de la IA, gente que quiere "estar al día" sin leer documentación densa.
- **Qué les damos:** entender lo que pasa en tech (lenguajes, frameworks, modelos de IA, empresas, infraestructura) en **clips cortos, animados y graciosos**, fáciles de digerir y de compartir.
- **Diferenciador:** la mayoría de canales tech son cabezas parlantes o tutoriales largos. Nosotros somos **animados, caricaturescos y veloces**, con una mascota memética — más cerca del entretenimiento que del aula, pero enseñando de verdad.
- **Métrica norte (a definir en Fase 0):** retención + veces compartido + crecimiento de seguidores.

---

## 5. Identidad y estilo de contenido (la "metodología")

Adaptamos la receta probada de los mejores divulgadores tech (referencia: Fireship) a un registro **más caricaturesco y juvenil**:

| Atributo | Especificación |
|---|---|
| **Ritmo** | Veloz. ~10–15 cambios de plano/elemento por minuto. Nada se queda quieto. |
| **Voz en off** | Enérgica, ~180–230 palabras/minuto. Tono cercano, irónico, juvenil. |
| **Humor** | Memes, exageración, comentario satírico sobre tendencias y "guerras" tech (Python vs JS, etc.). Deadpan ocasional. Sin ofender. |
| **Apoyo visual** | Mascota animada + texto en pantalla resaltado + snippets de código animados + íconos/logos + memes. Paleta y tipografía consistentes. |
| **Estructura del clip** | Hook fuerte en los primeros 2–3 s → 1 idea clara → remate/gracia o cliffhanger → CTA suave (seguir/comentar). |
| **Cierre** | Llamado a la acción o pregunta que invite a comentar (alimenta el módulo de engagement). |
| **Duración** | Shorts verticales 9:16, <60 s para descubrimiento (formato exacto a confirmar — §13). |

**Mezcla de contenido (estrategia de crecimiento):**
- **Tendencia (70%):** lo que pasa esta semana (lanzamientos de modelos, drama tech, releases, noticias). Empuja vistas rápido.
- **Perenne (30%):** "qué es X en 60 segundos", conceptos base, comparativas. Crece lento pero acumula.

---

## 6. Arquitectura del sistema — 5 módulos en n8n

Equipo de **workflows/agentes especializados** orquestados en n8n (ya desplegado), con **estado en Supabase Postgres** y **assets en Supabase Storage / MinIO**. Regla de oro por la RAM: **procesar un ítem (un video) a la vez** y limpiar temporales.

**M1 — Ideación y tendencias (Trend Scout)**
- Fuentes gratuitas: RSS de Hacker News, dev.to, GitHub Trending, Lobste.rs, blogs oficiales (OpenAI, Anthropic, etc.); Reddit API; Google Trends.
- Un LLM (Gemini Flash / Groq / Cloudflare / Claude) filtra, deduplica y puntúa cada tema por "potencial viral + encaje con el nicho + frescura".
- Escribe los temas aprobados a la tabla de cola en Supabase (`estado = pendiente`).

**M2 — Guionización (Scriptwriter)**
- Toma 1 tema `pendiente`. El LLM genera: hook, guion (tono de §5), desglose por escenas, prompts de imagen para la mascota/escenas, textos en pantalla, sugerencia de música/SFX y metadata (título, descripción, hashtags). Salida estructurada (JSON).

**M3 — Producción audiovisual (Studio)**
- **Imagen/personaje:** Cloudflare Workers AI (Flux) o Pollinations generan fondos y poses de la mascota; consistencia vía referencia (Seedream / Flux Kontext). Se guardan en Supabase Storage.
- **Voz:** narración con **edge-tts** (voz consistente del canal, en español).
- **Subtítulos:** karaoke quemado, usando el **timing por palabra que entrega edge-tts** (sin Whisper). *Whisper (faster-whisper "small", CPU) solo si se ingiere audio externo.*
- **Ensamblaje:** el **media-worker (FFmpeg)** compone el video 9:16 con música/SFX libres. (Para el look animado avanzado, ver decisión de motor en §13.)

**M4 — Publicación multiplataforma (Publisher)**
- Sube a YouTube (Shorts) + opcionalmente TikTok e Instagram Reels (vía Upload-Post free tier o APIs nativas).
- Aplica título/descripción/hashtags y **etiqueta de contenido generado con IA** (cumplimiento). Actualiza el estado en Supabase (`publicado` + URLs).

**M5 — Engagement + Analítica (Community & Insights)**
- **Engagement:** escucha comentarios; un LLM los clasifica (pregunta, elogio, queja, spam, lead) y **propone/publica respuestas con tono de marca**, con revisión humana opcional (WhatsApp/Gmail) para casos sensibles.
- **Analítica:** recoge métricas (vistas, retención, likes, comentarios) vía APIs nativas; genera reporte semanal.
- **Bucle de mejora:** las señales de qué funcionó re-alimentan M1/M2.

---

## 7. Stack tecnológico (mapeado a TU entorno)

> Regla: cualquier servicio de pago debe ser **aprobado explícitamente** por el dueño. Siempre proponer primero la ruta gratuita. ✅ = ya disponible · ⚙️ = a desplegar/integrar · ☁️ = servicio en la nube (free tier).

| Capa | Qué usar en tu entorno | Estado | Notas |
|---|---|---|---|
| **Orquestación** | n8n 2.22.5 @ `n8n.axchisan.com` | ✅ + MCP | Claude Code crea/ejecuta workflows por `n8n-mcp`. Exportar JSON al repo. |
| **Despliegue de servicios** | Coolify (+ MCP) | ✅ + MCP | Claude Code despliega contenedores nuevos directamente. |
| **Reverse proxy / HTTPS** | Traefik v3.6 | ✅ | Dominios `*.axchisan.com` con TLS automático. |
| **Estado (cola + log)** | Supabase Postgres | ✅ | Tabla con `estado`: pendiente → produciendo → en_revisión → publicado. |
| **Almacenamiento de assets** | Supabase Storage / MinIO | ✅ | Imágenes/audio/video entre pasos; evita pasar binarios pesados por RAM. |
| **LLM en pipeline** | Gemini Flash free / Groq / Cloudflare Workers AI; Claude API opcional | ☁️ | Sin GPU local. Gemini Flash ~1k req/día gratis; Groq muy rápido. |
| **Imagen / mascota** | ☁️ Cloudflare Workers AI (Flux schnell) — gratis, sin marca de agua; Pollinations.ai como alterna sin signup | ⚙️☁️ | Consistencia de mascota: imagen de referencia con Seedream / Flux Kontext. Fallback barato si se aprueba: Replicate/Fal (~$0.04/img). *Verificar límites del free tier al integrar.* |
| **Voz (TTS)** | **edge-tts** (en el media-worker) | ⚙️ | Gratis, voces en español, **entrega timing por palabra** → subtítulos sin Whisper. Piper como alterna offline. |
| **Subtítulos karaoke** | Timing de edge-tts + FFmpeg (ASS) | ⚙️ | faster-whisper "small" en CPU **solo** si se ingiere audio externo. **Nunca whisper-large** (RAM). |
| **Animación / ensamblaje** | **v1:** FFmpeg (liviano). **v2:** Remotion en contenedor dedicado **(1 render a la vez)** *o* render gestionado (Creatomate/Shotstack free tier) | ⚙️ | FFmpeg ausente hoy → primer paso. Remotion es pesado en RAM → concurrencia 1. Si la RAM aprieta, offload del render a la nube. |
| **Música / SFX** | YouTube Audio Library, Pixabay, Freesound | — | Verificar licencia de cada pista. |
| **Publicación** | YouTube Data API (gratis) + Upload-Post free tier (multiplataforma) | ⚙️☁️ | Falta credencial de YouTube en n8n. Upload-Post: ~10 subidas/mes gratis. |
| **Engagement** | YouTube API + LLM clasificador/respondedor | ⚙️ | Memoria corta de conversación por usuario. |
| **Aprobación humana (opcional)** | Evolution API (WhatsApp) o Gmail | ✅ (WhatsApp) | Enviar borrador + Aprobar/Rechazar antes de publicar. |
| **Analítica** | YouTube Analytics API + reporte por LLM | ⚙️ | Dashboard simple si se requiere. |

---

## 8. Pilares y formatos de contenido (banco de series)

La mascota como hilo conductor:
- **"X en 60 segundos"** — un lenguaje/herramienta/concepto explicado rapidísimo (perenne).
- **"El reporte del código"** — noticia tech de la semana con humor (tendencia).
- **"Peleas tech"** — comparativas tipo versus (Python vs JS, REST vs GraphQL) como combate animado.
- **"¿Qué pasó con…?"** — historia de una empresa/tecnología en clave caricaturesca.
- **"IA explicada como si tuvieras 5"** — modelos y conceptos de IA simplificados.
- **"Servidores y nubes para humanos"** — infraestructura sin tecnicismos.

*(Confirmar y priorizar series en Fase 0/2.)*

---

## 9. Workflows de referencia (estudiar ANTES de construir)

Plantillas gratuitas a importar y diseccionar (tomar el **patrón**, no las dependencias de pago):
- **Flux + Runway + ElevenLabs + Creatomate** — pipeline canónico desde Sheet (1 idea a la vez): `n8n.io/workflows/3416`
- **Animated stories: GPT-4o-mini + Midjourney + Kling + Creatomate** — `n8n.io/workflows/3655`
- **Gemini + ElevenLabs + Leonardo + Shotstack** — `n8n.io/workflows/6014`
- **Generación de imagen GRATIS con Cloudflare Workers AI + n8n** (clave para nuestra ruta sin GPU) — guía de GrowwStacks / repos `Cloudflare-Image-Worker`
- **Upload-Post (publicación multiplataforma, plantillas gratis)** — `upload-post.com/n8n-templates`
- **Auto-respuesta a comentarios de YouTube (n8n + LLM + YouTube API)** — buscar "Auto-Reply to YouTube Comments with AI n8n"
- **Reddit auto-comment con IA + tracking** — `n8n.io/workflows/7217`

El patrón objetivo (cola → guion → media → ensamblaje → publicación → log) encaja con el procesamiento **un-ítem-a-la-vez** que nos impone la RAM.

---

## 10. Restricciones, cumplimiento y brand safety

1. **Límites del servidor (críticos):** sin GPU → IA pesada en la nube; RAM ajustada → **un video a la vez**, limpiar temporales, sin whisper-large ni Remotion concurrente; **FFmpeg debe desplegarse** (media-worker) antes de cualquier producción.
2. **Divulgación de IA:** marcar el contenido como generado con IA donde la plataforma lo exija. No suplantar voces ni imágenes de personas reales.
3. **Personas y empresas reales:** se permite **parodia/sátira de productos, tecnologías y tendencias**. **Prohibido** poner citas inventadas en boca de ejecutivos/personas reales, difamar, o usar logos/marcas implicando respaldo. Que hable **la mascota**, no figuras reales.
4. **Derechos:** solo música/imágenes con licencia libre. Nada de material con copyright.
5. **Rate limits y salud de cuentas:** respetar límites de cada API (Reddit, YouTube, Cloudflare, etc.); espaciar publicaciones; nada de spam. Empezar conservador (1–2 publicaciones/día) y subir gradualmente.
6. **Calidad sobre cantidad:** paso de **control de calidad** (revisión humana vía WhatsApp/Gmail o checks automáticos) antes de publicar, sobre todo al inicio.
7. **Costo $0:** ningún servicio de pago sin aprobación previa.

---

## 11. Entregables por fases (roadmap)

- **Fase 0 — Cimientos.** Confirmar marca/plataforma/idioma/formato (§13). **Desplegar vía Coolify un contenedor `media-worker` (FFmpeg + edge-tts)** expuesto para que n8n lo llame por HTTP. Crear en **Supabase** las tablas de cola/log y los **buckets de Storage**. Crear token de **Cloudflare Workers AI** (imágenes gratis). Reunir credenciales (YouTube). Crear repo + este `CLAUDE.md`.
- **Fase 1 — "Hello, video" end-to-end.** En n8n (vía MCP): 1 tema en Supabase → guion con LLM → imagen(es) con Cloudflare → voz con edge-tts → **ensamblaje con FFmpeg** → archivo en Storage → revisión manual. Probar el pipeline de punta a punta **sin riesgo de RAM**.
- **Fase 2 — Identidad visual + mascota.** Definir estilo/paleta/tipografía y mascota (con imagen de referencia para consistencia). Plantillas reutilizables de animación. **Aquí se decide el motor v2 (Remotion-self-hosted-single vs render gestionado) según la RAM observada en Fase 1.**
- **Fase 3 — Ideación automática (M1) + guion afinado (M2).** Trend Scout (RSS/Reddit/Trends + scoring). Guionizador con tono y salida estructurada.
- **Fase 4 — Producción completa (M3).** Imagen/personaje + animación + voz + subtítulos karaoke (timing de edge-tts) + música/SFX + ensamblaje automatizado.
- **Fase 5 — Publicación (M4).** YouTube primero; luego TikTok/Reels vía Upload-Post u APIs. Metadata + etiqueta IA + log.
- **Fase 6 — Engagement + analítica (M5).** Clasificador/respondedor de comentarios + reporte semanal + bucle de mejora.
- **Fase 7 — Escalado y calidad.** QA automatizado, throttling, A/B de hooks/miniaturas, monitoreo de errores y reintentos, limpieza de Storage.

---

## 12. Primer paso concreto (al iniciar en Claude Code)

1. Leer este brief y confirmar los puntos 🔶 de §13 con el dueño.
2. Verificar `n8n-mcp` y `coolify` (ambos activos) y el acceso a Supabase.
3. **Desplegar el contenedor `media-worker` (FFmpeg + edge-tts) vía Coolify** y exponerlo por Traefik para que n8n lo consuma por HTTP.
4. Crear en Supabase la tabla `cola_contenido` (con `estado`) y los buckets de Storage; crear el token de Cloudflare Workers AI.
5. Importar 1–2 workflows de referencia (§9) para estudiar el patrón.
6. Construir y ejecutar el workflow de **Fase 1** directamente con `n8n-mcp`.

No avanzar más allá de Fase 1 sin validar el resultado (y medir la RAM durante el render) con el dueño.

---

## 13. 🔶 Decisiones abiertas (confirmar con el dueño)

1. **Plataforma(s) inicial(es):** ¿YouTube Shorts, TikTok, Instagram Reels, o varias? (Define APIs/credenciales.)
2. **Idioma del contenido:** ¿Español LatAm, español neutro, inglés o bilingüe? (Afecta la voz de edge-tts.)
3. **Formato/duración inicial:** ¿Solo Shorts <60 s, o también videos medios 2–5 min?
4. **Mascota:** dirección del personaje (robot simpático / criatura / avatar dev), nombre y estilo. Define la imagen de referencia para consistencia.
5. **Nivel de autonomía:** ¿publicación 100% automática desde el inicio, o con **aprobación por WhatsApp** hasta validar calidad?
6. **Nicho fino:** ¿generalista (toda la tech) o enfocado (p. ej. solo IA, solo web dev) al arrancar?
7. **Motor de animación v2 (decidir tras Fase 1):** Remotion self-hosted (gratis, pesado en RAM, 1 render a la vez) **vs** render gestionado en la nube (protege tu RAM, free tier limitado).
8. **Proveedor de imagen para empezar:** Cloudflare Workers AI (recomendado) vs Pollinations (sin signup). *Recomendación: arrancar con Cloudflare.*

---

*Documento vivo. Actualizar a medida que el proyecto evoluciona.*
