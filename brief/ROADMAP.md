# 🛠️ ROADMAP de mejoras — CreationContent

Documento vivo de mejoras del pipeline de video. Base: Fase 1 funcionando end-to-end. Ver `CLAUDE.md` (brief).

> 👉 **Para retomar el proyecto lee primero `HANDOFF.md`** (estado actual + tarea en curso).

## 🆕 Estado 2026-06-24 — sistema completo en producción
Canal **auto-programado y multiplataforma** funcionando: generación nocturna (skill manual → Productor renderiza con EC2 Dom/Mié) → aprobación por franjas (WhatsApp+email) → **publica en YouTube + Instagram sincronizados**. Logros recientes:
- ✅ **Instagram integrado** (Reels vía Graph API, token en `cc_config`, workflow `9xcLWJ13DmChF3oI`; backlog re-sincronizado en orden de YouTube). ⚠️ IG no permite borrar por API.
- ✅ **4 voces en Azure es-MX nativo** (bit/chip/sonda/tera; chip cambiado a Brian joven). Aprobadas por el dueño.
- ✅ **Sesión Gemini en DB + extensión Chrome** (ya no vence en silencio). **Límite diario de imágenes de Gemini** documentado (uso normal 9/día está OK).
- ✅ **Abort-on-fail** (si Gemini falla, no produce video degradado) + **watchdog** (resetea atascados) + constraint `estado` arreglado (`en_aprobacion`/`fallido`).
- 🔴 **EN CURSO: formato DEBATE por turnos (2 voces).** Worker `/tts-dialogue` + skill listos; **falta cablear nodos TTS + Build del workflow `2xACDbjcP0TQ6iTF`** (diseño exacto en `HANDOFF.md`).
- 🟡 **TikTok:** pendiente (manual + auditoría API nativa).

---

## 🆕 Estado 2026-06-21 — "modelo presentable" (validado por el dueño)

El dueño confirmó que el modelo ya es **presentable y bueno**. Hitos cerrados hoy:
- ✅ **Gráficos con IA (el gran salto):** servicio **cc-browser** (`browser.axchisan.com`, Coolify) genera **infografías caricaturescas con Gemini web** usando la sesión Google AI Plus del dueño — **gratis**, sin API de pago. Integrado: escenas de datos = infografía a pantalla completa con info real; escenas de imagen = Pexels. Ver memoria `voz-y-graficos-avanzado`.
- ✅ **Voz:** ElevenLabs (Liam) con karaoke + **fallback automático a edge-tts** + **parametrizable sin redeploy** (tabla `cc_config`).
- ✅ **Bit a COLOR (bug B&N resuelto):** causa raíz = la rama de la mascota se escalaba en yuv420p (croma 4:2:0) y el submuestreo promediaba las líneas finas azules → gris. Fix: procesar la mascota en **yuv444p antes de escalar** (`format=yuv444p` + `scale ...:flags=lanczos`).
- ✅ **Animaciones variadas:** saludar solo al inicio, despedida solo al final, y poses de contenido (hablar/señalar/asentir/pensar/celebrar) **divididas en 2 sub-segmentos por escena** (ya no un bucle de "saludar").
- ✅ **Música:** volumen bajado 0.18 → **0.08** (era muy alto).
- ✅ **Resiliencia:** nodo Cloudflare con retry + `continueRegularOutput` (un rate-limit ya no rompe el pipeline); Build cae a degradado si no hay imagen.
- 🔧 **Infra de edición:** los nodos n8n se editan sin escapado vía REST (`~/.claude.json` → N8N_API_URL/KEY; PUT `/api/v1/workflows/{id}`).
- ✅ **Skill `/tecnobichos-temas` construida + probada:** genera temas+guiones; el pipeline usa el guion pre-escrito (Groq = fallback). Biblia del canal en `CANAL.md`.
- ✅ **Migración del render a AWS (m7i-flex.large 8GB):** media-worker + cc-browser corren en AWS (Coolify multi-servidor); el VPS Contabo ya NO se congela (render ~5 min). Auto start/stop del EC2 (EventBridge). Ver [[aws-render-migracion]] y `AWS-SETUP.md`. Calidad full restaurada.
- ✅ **Fix ñ/tildes:** los guiones de la skill se escribían sin ñ ni acentos (mi error); el pipeline SÍ los soporta (ASS en UTF-8, TTS devuelve "niños"/"español"). Corregidos los 6 + regla de ortografía añadida a la skill.

## 🆕 Estado 2026-06-21 (tarde) — voz por mascota, backup y multi-mascota

Segunda tanda de mejoras tras revisar los 6 videos de Fase 0 (el dueño los validó: "me encanta, es hermoso"):
- ✅ **Aperturas con Gemini (no Pexels):** la 1ª escena (y todas las de tipo imagen) ahora es una **ilustración Gemini coherente al tema** (portada), ya no una foto Pexels aleatoria. `gprompt_intro()` en el Build node; se eliminó el uso de Pexels en el pipeline.
- ✅ **Timing de animaciones por TIEMPO (no por escena):** saludo solo en el `INTRO` (~primeros 6s), despedida solo en el `OUTRO` (~últimos 4.5s), y en el medio poses de contenido alternadas cada ~9s. Antes las animaciones extremas duraban demasiado.
- ✅ **Subtítulos más pequeños y sin tapar la mascota:** fuente 58, `MarginV` 470 (suben sobre Bit); se quitó el título superpuesto (cada infografía Gemini ya trae el suyo).
- ✅ **Base multi-mascota (hasta 2 alternando):** `MASC` map por mascota (bit/chip/sonda/tera con prefijo+poses); `lay(M,'br')` + co-protagonista `lay(CO,'bl')` en esquina opuesta. Listo para cuando existan los clips de las secundarias.
- ✅ **cc-browser borra la conversación de Gemini** tras generar cada infografía (selectores reales verificados: `gem-nav-list-item[data-test-id=conversation]`, `[data-test-id=delete-button/confirm-button]`) → el historial del dueño ya no se llena.
- ✅ **Backup + reuso de gráficos por video:** el worker guarda cada infografía Gemini en `cc-assets/video-<topic_id>/img_NNN.png` y en re-renders la **reusa** (no vuelve a llamar a Gemini). Cambiar la voz / re-render = **gratis, rápido e idéntico**. Requiere `SUPABASE_SERVICE_KEY` en el worker + `topic_id` en el Render. Verificado: 6/6 con 3 imgs c/u.
- ✅ **Voz PARAMETRIZABLE POR MASCOTA (sin redeploy):** `cc_config` clave `tts` `valor` pasó a **mapa** `{_default,bit,chip,sonda,tera}` con `provider/voice/fallback_voice/rate/pitch` (+ `pron`); el nodo TTS elige por `guion.mascota`. Voces: **Bit=Giuseppe** (it-IT ML), **Chip=Florian** (de-DE ML, grave), **Sonda=Emma** (en-US ML, ágil), **Tera=Ava** (en-US ML, calmada). ⚠️ Las voces `...MultilingualNeural` de **zh-CN (Yunyi)** y **ja-JP (Masaru)** NO están en edge-tts gratis (Azure de pago) → probar siempre vía `/tts` antes de fijar.
- ✅ **Acento nativo con Azure Speech (F0 gratis):** Giuseppe (it-IT) en edge-tts sonaba italiano; el endpoint gratis NO manda `<lang>`. Solución: **`provider=azure`** en el worker (SDK + SSML `<lang xml:lang=es-MX>` → acento mexicano nativo + word-boundary para karaoke). Bit ahora en Azure/Giuseppe/es-MX (ya no hace falta `pron`). Desbloquea Yunyi/Masaru. Recurso Speech F0 del dueño (region eastus). Ver memoria `voz-y-graficos-avanzado`.

## 🆕 Estado 2026-06-21 (noche) — PUBLICACIÓN + SISTEMA AUTO-PROGRAMADO

- ✅ **Primer video publicado en YouTube automáticamente:** `https://youtu.be/xJ2_zp4TeI8`. Pipeline M4 probado end-to-end.
- ✅ **Sistema central de alertas (cc-alert):** webhook n8n → tabla `cc_logs` + WhatsApp (573183038190, Evolution AxIAPersonal). Reportan: n8n (errorWorkflow), media-worker (render/ffmpeg/Gemini degradado) y cc-browser (**sesión Gemini caída = crítico**). Ver `sistema-alertas`.
- ✅ **Publicación híbrida + aprobación WhatsApp:** YouTube nativo (cred OAuth `youTubeOAuth2Api`) + aprobación por links WhatsApp. Descripción rica (hook+valor+atribución CC-BY+CTA+hashtags), confirmación al publicar. ⚠️ nodo YouTube devuelve `uploadId`.
- ✅ **Sistema AUTO-PROGRAMADO (3 videos/día 12/5/9pm, generación lotes Dom+Mié):** workflows Recordatorio (Dom+Mié 8pm WhatsApp+Email), Productor (auto-render cuando EC2 encendido), Enviar-a-aprobación (3 franjas), Publicador. **Único manual:** correr la skill de noche + aprobar de día. AWS EventBridge reconfigurado (EC2 Dom+Mié 10pm-2am). Ver `publicacion-m4` + `WORKFLOWS.md`.
- ✅ **Miniatura 16:9 automática:** worker genera `thumb.jpg` (portada sobre fondo desenfocado) por video → webhook `cc-set-thumb` la sube vía YouTube `thumbnails.set`. ⚠️ **Requiere verificar el canal (teléfono)** para que YouTube acepte miniaturas personalizadas.
- ✅ **Clips secundarias 6/8 poses** (chip/sonda/tera: saludar, hablar, señalar, pensar, asentir, despedida) subidos a `cc-mascota` + Build node los usa. Faltan idle/celebrar (cuota LTX).
- ✅ **Organización:** Downloads consolidado en `~/Downloads/Tecnobichos/` (videos+voces); workflows n8n con tags (Tecnobichos + función); `WORKFLOWS.md` documenta todos.

### 🎯 Próximas prioridades (actualizado 2026-06-21 noche)
0. **🟡 PENDIENTE DEL DUEÑO:** (a) **verificar el canal de YouTube** (youtube.com/verify) para miniaturas; (b) **API key de Upload-Post** (TikTok+IG); (c) confirmar email de recordatorios.
1. **✅ D — Skill de generación — CONSTRUIDA + actualizada** (lotes ~10-12, cadencia 3/día). PENDIENTE: primera corrida real grande.
2. **✅ E — Notificación — HECHA** (cc-alert + cc_logs + WhatsApp). Falta opcional: agente de auto-reparación que lea `cc_logs` e intente arreglar; health-checks proactivos (cuota Azure/Gemini).
3. **🟠 Identidad/historia del canal.** Definir nombre(s) de redes, narrativa de lanzamiento, integración paso a paso.
4. **✅ Música adaptada — RESUELTO.**
5. **🟠 Completar elenco:** clips idle/celebrar de secundarias (cuota LTX); a futuro diálogos entre mascotas (formato debate §C).
6. **🟢 Pulido infografías:** texto chico de Gemini a veces distorsionado.
7. **🟠 Upload-Post (TikTok+IG):** integrar al publicador cuando llegue la API key.
8. **🟢 Verificación de canal + miniaturas** (depende del dueño).

### 🎯 Prioridades históricas (2026-06-21 tarde)
1. **✅ D — Skill de generación de temas/guiones — CONSTRUIDA (2026-06-21).** `CreationContent/.claude/skills/tecnobichos-temas/SKILL.md` (`/tecnobichos-temas`). Lee `CANAL.md` + cola (no repetir, detectar fase) → investiga tendencias (HN/dev.to/GitHub, noticias IA, Reddit, Google Trends, redes) → propone 5–8 temas con score (espera aprobación) → escribe guion completo → inserta `pendiente` en `cc_cola_contenido`. Pipeline ya usa el **guion pre-escrito** (Claim devuelve `guion`; Parse lo prefiere; Groq = fallback tolerante). Base: `CANAL.md` (biblia: elenco Bit/Chip/Sonda/Tera, lore, fases). PENDIENTE: primera corrida real + clips de Chip/Sonda/Tera.
2. **🔴 E — Notificación + auto-reparación por nodo.** Si falla la sesión de Gemini, se agota ElevenLabs, no se puede publicar, etc. → **notificar al dueño** (WhatsApp Evolution / Gmail) con el detalle, y un **agente** que procese la solicitud de vuelta y corrija lo que esté a su alcance.
3. **🟠 Identidad/historia del canal.** Definir nombre(s) de redes, narrativa de lanzamiento ("darle vida"), e ir integrándose en redes paso a paso (construir historia). Previo a Fase 5 (publicación).
4. **✅ Música adaptada al contenido — RESUELTO (2026-06-21).** Causa raíz REAL del "no se oye": el `.gitignore` del worker excluía `*.mp3` → **el archivo de música NUNCA estuvo en el contenedor** → `music_on` siempre `False` → la música jamás se mezcló (sin importar el volumen). Fix: `git add -f` del mp3 + **música dinámica por mood**: 4 pistas CC-BY (Kevin MacLeod, normalizadas a -18 LUFS) en `cc-mascota/music/` (chill/tech/groovy/inspiring), el Build node elige por temática (`pickMusicUrl` analiza titulo+tema+pilar), el worker descarga `music_url` y la mezcla con **ducking sidechain** (suena clara en los silencios, se agacha bajo la voz). Verificado: 0 silencios, gaps a -23dB. ⚠️ CC-BY → atribuir Kevin MacLeod en la descripción (ver `assets/music/CREDITS.md`).
5. **🟠 MÁS MASCOTAS (identidad del canal).** Bit es la principal, pero el canal tendrá un **elenco**: secundaria #2 retro naranja (para debates/diálogos), y personajes extra que den vida e identidad. Cada una con su set de clips (como Bit). Construir el elenco progresivamente; reusa el mismo pipeline de clips (LTX/HF) y compositing (alfa empacado yuv444p). A futuro: diálogos entre mascotas (formato debate, §C).
6. **🟢 Pulido infografías:** texto descriptivo pequeño y denso de Gemini a veces sale con palabras distorsionadas (límite del modelo en texto chico); titulares/visuales salen nítidos.

---

## Estado actual (MVP Fase 1)
- ✅ Pipeline end-to-end: tema → guion (Groq) → **infografías (Gemini/cc-browser)** + Pexels → voz (ElevenLabs/edge-tts) → render (FFmpeg 9:16, Bit a color + subtítulos karaoke + música) → Storage → `en_revision`.
- ✅ Imágenes con movimiento (Ken Burns), transiciones, subtítulos, mascota consistente y a color.

## En progreso (pulido de Fase 1)
- [x] **Subtítulos** sincronizados quemados (ASS, chunks de 3 palabras, timing del TTS). ✅ 2026-06-20
- [x] **Transiciones** crossfade (xfade) entre escenas. ✅ 2026-06-20
- [x] **Movimiento** Ken Burns (zoom alternado in/out) en cada imagen. ✅ 2026-06-20
- [x] **Voz estándar del canal**: **es-MX-JorgeNeural +10%/+8Hz** (elegida por el dueño 2026-06-20).
- [x] **Karaoke real** (resaltado palabra por palabra en amarillo, \k de ASS). ✅ 2026-06-20
- [x] **Textos de escena** en pantalla (título arriba) con auto-wrap. ✅ 2026-06-20
- [x] **Música de fondo** mezclada bajo la narración (Carefree, Kevin MacLeod, CC-BY; volumen 0.18, amix). ✅ 2026-06-20
      ⚠️ CC-BY exige atribución en la descripción del video (ver assets/music/CREDITS.md). Sustituir por CC0 si se quiere evitar el crédito.

---

# 🧭 PLAN ESTRATÉGICO (visión del dueño, 2026-06-20)

Tres grandes líneas de evolución, analizadas por **relevancia/impacto** y ordenadas para construir paso a paso. Cada una reusa lo ya construido (media-worker, n8n, Supabase, Cloudflare).

## A. Sistema de MASCOTA (🔴 prioridad máxima — es la columna vertebral de la marca)
Objetivo: una mascota **fija y reconocible** que **convive con los elementos del video** (no una imagen suelta), con una **línea de sprites/animaciones** características (pensando, mirando los subtítulos, saludando, cierre con el nombre del canal…) y capacidad de **generar poses por contexto**. A futuro: 2+ mascotas para diálogos.

**Análisis de herramientas (investigado):**
- **Consistencia del personaje (misma mascota, distintas poses):** **Flux.1 Kontext** (img2img con imagen de referencia) da la **mejor consistencia sin entrenar LoRA**, superior a IP-Adapter. NO está en Cloudflare free → vía **fal.ai / Replicate (~$0.03–0.04/img, requiere aprobación del dueño)**. Alternativa $0 pero menos consistente: Pollinations / reusar seed.
- **Animación 2D del personaje:**
  - **Live2D Cubism** (editor free): ideal para **avatar parlante expresivo** (boca/ojos) desde UNA ilustración → perfecto para el formato debate.
  - **Rive** (editor free, rive.app): **máquina de estados** (idle, pensando, saludando, hablando) que el pipeline dispara por "estado". Runtime ~200KB. Ideal para mascota interactiva.
  - **Lottie** (free, After Effects/alternativas): animaciones loop simples (saludo, burbuja de pensar). Runtime ~60KB, solo playback.
  - **Spine**: skeletal de calidad pero de pago.
- **Compositing en el video:** overlay de sprite PNG transparente con FFmpeg (slide-in, bob, talk-bob) — ya factible en el media-worker.

**Decisiones del dueño (2026-06-20):** mascota principal = concept **#1 redondo azul** (referencia maestra = `mascota-concepts/01_redondo_azul.jpg`); secundaria = **#2 retro naranja** (para debates); #3/#4 = personajes extra. Quiere **clips/animaciones** (no sprites estáticos) generados desde la imagen base. La ruta gratis de Flux schnell da "misma familia" pero NO idéntico → se descarta para la insignia.

**Herramientas de animación investigadas (gratis):**
- **Imagen→video (movimiento):** Hailuo/MiniMax (free generoso, uso comercial) ⭐, Kling (~6/día, sin marca), Haiper (uso comercial), Luma. ⚠️ sin API gratis → uso manual web.
- **Expresiones/hablar (open-source):** **LivePortrait** (HF Spaces, anima expresiones desde driving video, sirve para cartoon), **SadTalker** (talking head lip-sync desde audio) — para el formato debate.

**Enfoque recomendado (construir 1 vez, reusar, $0/video):**
- **2A — Identidad** (🔴): fijar referencia maestra del #1 (hecho) + nombre/personalidad/paleta/tipografía. Preparar versión **fondo verde** de la base para chroma-key.
- [x] **2B — Librería de CLIPS animados** ✅ 2026-06-20: generados vía LTX (API gratis + token HF) 8 clips de **Bit** (idle, saludar, pensar, señalar, hablar, celebrar, asentir, despedida), 768x768 ~3s, mascota idéntica. En `Downloads/mascota-principal-poses/lib_*.mp4` y en **Supabase `cc-assets/mascota-bit/`**. Tienen un leve borde "sticker" (se recorta al hacer overlay). Pendiente: más acciones/variantes según haga falta.
- [x] **2C — Compositing en media-worker** ✅ 2026-06-20: el /render hace overlay de Bit por escena con **alfa empacado** (h264 color-arriba/alfa-abajo + split+alphamerge; recorte por flood-fill, no chromakey). El guion (Groq) declara `accion` por escena; el Build node arma la lista `mascots` (URL pública en bucket `cc-mascota`, ventana temporal por escena, esquina br, scale 0.32). Probado: video v5 con Bit saludar→señalar→celebrar. $0/video (clips fijos reusados). Clips empacados: `Downloads/mascota-principal-poses/packed/` y `cc-mascota` (público, 2.7M total).
- **2D — Hablar sincronizado** (🟡, después): SadTalker (audio→lip-sync) por línea para el formato debate. MVP previo = clip "hablar-loop" + los subtítulos karaoke que ya tenemos.
- ✅ **HALLAZGO (2026-06-20):** el HF Space **`Lightricks/ltx-video-distilled`** expone endpoint **`/image_to_video` por API gratis** (gradio_client). Probado con la imagen base #1 → genera clip 768x768 ~3s que **mantiene la mascota idéntica** y la anima (saludo/bob). ¡Es automatizable y $0! Clips de prueba en `mascota-principal-poses/clip_saludo_v2.mp4`. Esto habilita construir la librería de clips por API.
- Pendiente para overlay con transparencia: generar sobre **fondo verde/magenta** (preprocesar la base con ese fondo) para chroma-key, o usar rembg por frame. El robot es blanco → NO usar white-key.

## B-bis. MOTOR DE RELEVANCIA VISUAL (🔴 alto impacto — conexión imagen-tema)
Problema detectado por el dueño (2026-06-20): nuestras imágenes son ilustraciones IA **genéricas**, no aluden a lo que se dice. Referencia: **@david_bz56** (TikTok). **ANÁLISIS REAL** (videos descargados con yt-dlp + frames analizados, ver [[analisis-referente-tiktok]]): su "plus" NO es b-roll stock — son **infografías/diagramas técnicos DISEÑADOS y muy específicos** (tema oscuro navy+cian, tarjetas-concepto con **íconos**, **bloques de código real**, flechas de flujo, arquitecturas precisas) animados con **pan/zoom** (Runway). El diferenciador real = *diagramas técnicos ricos + código + íconos, con movimiento de cámara*, deeply on-topic. Replicable GRATIS con Mermaid(dark)+Pygments+Iconify+QuickChart+Ken Burns. (Submagic/Crayo hacen b-roll stock automático pero son de pago; Pexels es la via gratis para b-roll general.)

**Se replica GRATIS** con estas fuentes (validadas):
- **Pexels API** (free, requiere key gratis): fotos/videos stock reales por keyword (b-roll de conceptos: "programador", "data center", "robot IA"). Sin atribución.
- **Iconify API** (sin key): **logos/íconos reales a color** (sets `logos`, `devicon`, `skill-icons`) — el logo exacto de Python/Rust/empresas.
- **Wikipedia/Wikimedia REST** (sin key): **info real** (extract) + **imágenes reales** de cosas específicas → para precisión y visuales reales. `…/api/rest_v1/page/summary/<tema>`.
- **QuickChart/Kroki** (sin key): charts/diagramas (ya hecho).
- **Cloudflare Flux** (ya): ilustraciones estilizadas (fallback estético).

**Plan (motor de relevancia):**
- [x] **Código resaltado** ✅ 2026-06-20: panel `code` (Pygments monokai) en tarjeta de marca. Groq emite `codigo={lang,code}`. Probado (ej. scikit-learn).
- [x] **Diagramas** ✅ 2026-06-20: Mermaid **limpio/rounded** (tema base de marca, nodos `([...])` azul/blanco) en tarjeta. (El sketch/handDrawn se DESCARTÓ — al dueño no le gustó. NotebookLM NO tiene API, no automatizable.) Groq prefiere diagramas; min 2 escenas con apoyo visual.
- [x] **B-roll real Pexels** ✅ 2026-06-20: key del dueño en env `PEXELS_KEY` del worker; `ImageItem.pexels` busca foto vertical relevante. Groq da `query_foto` por escena; el Build usa Pexels como fondo en escenas tipo `imagen` (las de panel mantienen fondo abstracto Cloudflare).
- [x] **Fix corte de video** ✅ 2026-06-20: `/tts` ahora devuelve `audio_duration_ms` (ffprobe del mp3 real); el Build dimensiona el video con esa duración → `-shortest` ya no corta el final (antes el timing por palabra quedaba ~0.8s corto).
- [x] **Guion redes sociales** ✅ 2026-06-20: empieza "Hola tecnobichos!" + hook, 160-200 palabras, cierra con CTA (síguenos). Probado (v10, 170 palabras, 67s).
- [ ] **Íconos/logos reales** (Iconify): panel `iconos` con tarjetas ícono+label. Pendiente: necesita rasterizar SVG (cairosvg/resvg) en el worker. FontAwesome dentro de Mermaid NO lo soporta Kroki.
- [ ] **B-roll real** (Pexels): foto/video relevante por keyword. Pendiente: **key de Pexels** del dueño.
- [ ] **Info/datos reales** (Wikipedia): grounding del guion + datos reales para charts (hoy el LLM los estima). 
- Nota de aprendizaje: el referente usa **Runway** (diagramas técnicos oscuros + cámara) y **NotebookLM** (gratis, pizarra/sketch). Emulamos el sketch con Mermaid handDrawn; el movimiento con Ken Burns.

## B. MOTOR GRÁFICO de apoyo (🟠 alto impacto — el gran diferenciador educativo)
Objetivo: pasar de "imágenes decorativas IA" a **gráficos de apoyo con información real**: charts/datos, **mapas mentales/diagramas**, snippets de código resaltado, tablas comparativas. El LLM decide por escena qué elemento usar y con qué datos.

**Herramientas (todas con free tier / sin GPU):**
- **Charts** (barras/líneas/pastel): **QuickChart.io** (API free, config → PNG) o matplotlib en un microservicio.
- **Diagramas / mapas mentales / flujos**: **Mermaid** vía **Kroki.io** (API free) o mermaid-cli → PNG.
- **Código resaltado**: render con Pygments (en el media-worker) o estilo Carbon vía API.
- **Tablas / lower-thirds**: HTML+CSS → imagen (renderer headless) o `drawtext`/ASS.

**Pasos:**
- [x] **B1** ✅ 2026-06-20: el guion (Groq) declara `tipo_apoyo` (imagen|chart|diagrama) + datos (chart {tipo,etiquetas,valores,titulo} / mermaid) por escena. Al menos una escena con apoyo grafico.
- [x] **B2+B3** ✅ 2026-06-20: media-worker /render acepta `panels:[{chart|mermaid|url|b64, start_sec, end_sec, width_frac, y_frac}]` → renderiza chart con **QuickChart** y diagrama con **Kroki/Mermaid** (ambos free, sin auth) y los superpone como **tarjeta centrada** por escena, sobre el fondo Cloudflare, bajo subtítulos y junto a Bit. Probado: video v6 (chart "Velocidad y Eficiencia" + diagrama "empresas que adoptan Rust"). $0.
- [x] **Pulido B** ✅ 2026-06-20: tarjetas de marca (Pillow: redondeada + barra azul + **título** + valores/datalabels), charts con sentido (Groq pide comparacion real + titulo + unidad), **diagramas Mermaid temáticos** (init brand), fondos **simples/abstractos** en escenas con datos (no compiten con la tarjeta), sin título duplicado (se suprime el de arriba cuando hay tarjeta), padding para no cortar valores.
- [x] **Fix "videos incompletos"** ✅ 2026-06-20: NO era fallo del pipeline (audio/video alineados) — la narración de Groq salía muy corta (57 palabras). Solucionado forzando 130-170 palabras (ahora videos ~40s completos) + max_tokens.
- [x] **Bit coherente** ✅ 2026-06-20: acción por **reglas** en el Build node (saludar en escena 1, despedida en la última, señalar cuando hay tarjeta de datos, hablar en el resto) — ya no aleatorio.
- [ ] Pendiente B (futuro): **código** resaltado (Pygments), **tablas comparativas**, iconos tech (Iconify), y datos REALES (hoy el LLM estima los números; evaluar fuentes/APIs de datos reales por tema).

## C. FORMATO "DEBATE sobre gameplay" (🟠 alto alcance — segundo formato del canal)
Investigado: formato **brainrot** viral = dos personajes con voces IA debaten un tema sobre **gameplay de fondo** (Minecraft parkour / Subway Surfers), **captions-first**, vertical. Lo hacen tools como Brainrotify/rotgen con Peter Griffin+Stewie. **Nosotros usamos NUESTRAS dos mascotas** (evita el problema legal de usar personajes con copyright — el brief lo prohíbe).

**Componentes (reusan A y B):**
- Guion-**diálogo** (LLM): A vs B alternando turnos, con humor y postura ("¿qué base de datos es mejor?"), una le pregunta a la otra, etc.
- **Voces distintas** por personaje (edge-tts: p.ej. Jorge para A, otra para B).
- **Fondo de gameplay** en loop — ⚠️ **licencia**: usar loops "no copyright"/CC0 o **grabados por nosotros** (no gameplay ajeno con copyright). Crear librería de fondos segura.
- Dos mascotas parlantes (2D rig 2D del bloque A) en esquinas + **subtítulos grandes animados**.

**Pasos:**
- **C1**: definir 2ª mascota + voz. **C2**: guion-diálogo por turnos (LLM). **C3**: librería de fondos gameplay con licencia segura. **C4**: render con fondo de video + 2 mascotas parlantes + captions. Es un **workflow/formato nuevo** en paralelo al explicativo.

**Orden sugerido:** A (2A→2B→2C) → B (B1→B2) → A‑2D → C. A y B pueden avanzar en paralelo una vez exista la mascota base.

---

# 📅 Fases posteriores (M1–M5 del brief)
- [ ] **M1 Trend Scout**: ideación automática (RSS HN/dev.to/GitHub Trending/Reddit + scoring por LLM).
- [ ] **M2** guion afinado (incluye guiones de diálogo del formato C) con la voz/personalidad de marca.
- [ ] **M4 Publicación**: YouTube Shorts + TikTok/IG (Upload-Post) + etiqueta de IA + atribución de música.
- [ ] **Aprobación por WhatsApp** (Evolution API) antes de publicar.
- [ ] **M5 Engagement + Analítica**: auto-respuesta a comentarios + reporte semanal + bucle de mejora.

## Deuda técnica del pipeline
- [ ] Manejo de errores/reintentos en los nodos del workflow (hoy sin onError/retry).
- [ ] Cambiar trigger Webhook → **Schedule** (procesar cola periódicamente) cuando pase a producción.
- [ ] **Exportar los workflows n8n como JSON al repo** (versionado) — pendiente crear el repo git de CreationContent.
- [ ] Subir las **imágenes** a `cc-assets` (hoy se pasan en base64 directo al render) para aliviar payloads.
- [ ] Limpieza periódica de Storage (videos/temporales viejos).
