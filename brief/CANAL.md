# 📺 Biblia del canal — Tecnobichos

> Fuente de verdad de la **identidad, elenco e historia** del canal. La skill de generación de
> temas/guiones (y todo el pipeline) debe respetar este documento al pie de la letra.
> Estado: **borrador 2026-06-21**. Las decisiones marcadas 🔶 esperan confirmación del dueño.

---

## 1. Identidad

- **Nombre:** **Tecnobichos**
- **Qué es:** canal de contenido tech (programación, IA, lenguajes, servidores, grandes empresas) en
  **clips cortos verticales (9:16, <60s)**, animados, caricaturescos y divertidos, pero **educativos y rigurosos**.
- **Público:** jóvenes ~16–28 (estudiantes, devs junior, entusiastas de IA) que quieren estar al día sin documentación densa.
- **Idioma:** Español LatAm neutro.
- **Plataformas:** YouTube Shorts + TikTok + Instagram Reels.
- **Tono:** enérgico, irónico, juvenil, memético — pero sin desinformar. Hook fuerte en 2-3s → 1 idea clara → remate/gracia → CTA suave.
- **Diferenciador:** somos **animados y veloces con mascotas**, con **infografías reales generadas por IA** (no cabezas parlantes ni tutoriales largos).
- **Mezcla de contenido:** ~70% **tendencia** (lo de esta semana) + ~30% **perenne** ("qué es X en 60s").

## 2. Elenco de mascotas (CONFIRMADO 2026-06-21)

La mascota da consistencia e identidad. **Bit** es la estrella; las demás aparecen según el formato.

| Mascota | Concepto | Rol | Personalidad | Aparece en |
|---|---|---|---|---|
| **Bit** 🔵 | `01_redondo_azul` (robot azul redondo) | **Anfitrión principal** | Curioso, optimista, enérgico, explica con claridad y humor. La cara del canal. | TODOS los videos |
| **Chip** 🟠 | `02_retro_naranja` (robot retro CRT) | Veterano / contrincante | Nostálgico gruñón "en mis tiempos…", defiende lo old-school. | Peleas tech, comparativas |
| **Sonda** 🟣 | `04_dron_flotante` (dron de un ojo) | Reportera / scout | Hiperactiva, chismosa, trae la noticia volando. | El reporte del código (noticias/tendencias) |
| **Tera** ⬜ | `03_minimal_pantalla` | IA sabia / explicadora | Calmada, precisa, "modo profe" para conceptos profundos de IA. | IA explicada, conceptos densos |

> ⚙️ Nota técnica: hoy solo **Bit** tiene clips animados (`cc-mascota/lib_*.mp4`). Chip/Sonda/Tera necesitan su set de clips (mismo proceso: LTX desde su imagen base) **antes** de protagonizar. La skill puede escribir guiones que los mencionen/introduzcan, pero el render solo superpone a Bit hasta que existan sus clips.

## 3. Pilares / formatos (series)

- **"X en 60 segundos"** — un lenguaje/herramienta/concepto rapidísimo (perenne). → Bit.
- **"El reporte del código"** — noticia tech de la semana con humor (tendencia). → Bit + Sonda.
- **"Peleas tech"** — versus animado (Python vs JS, REST vs GraphQL). → Bit vs Chip.
- **"¿Qué pasó con…?"** — historia de una empresa/tecnología. → Bit (+ Sonda).
- **"IA explicada como si tuvieras 5"** — modelos/conceptos de IA simples. → Bit + Tera.
- **"Servidores y nubes para humanos"** — infraestructura sin tecnicismos. → Bit.

## 4. Voz y estilo audiovisual

- **Voz:** **una voz por mascota** (edge-tts gratis) con karaoke. Parametrizable por mascota en `cc_config` (clave `tts`, mapa `{bit,chip,sonda,tera,_default}` con `provider/voice/fallback_voice/rate/pitch`) — el nodo TTS elige según `guion.mascota`, sin redeploy. Definitivas: Bit=Giuseppe (it-IT ML), Chip=Florian (de-DE ML, grave), Sonda=Emma (en-US ML, ágil), Tera=Ava (en-US ML, calmada). (ElevenLabs Liam disponible si se reactiva esa cuenta.)
- **Visual:** infografías caricaturescas (Gemini) a pantalla completa en TODAS las escenas, incluida la apertura (ya no se usa Pexels); cada imagen Gemini se respalda en `cc-assets/video-<id>/` y se reusa en re-renders; Bit + subtítulos karaoke encima.
- **Música:** dinámica por mood (chill/tech/groovy/inspiring), suave, con ducking. CC-BY (atribuir Kevin MacLeod).
- **Paleta:** azul de marca + acentos (naranja/morado de las mascotas). Texto grande y legible.

## 5. 📅 Lore y cronología del canal (CONFIRMADO 2026-06-21)

**Lore (historia de fondo):** **Bit es una IA que "despertó"** dentro de los servidores y, curiosa por el mundo
tech que la creó, sale a explorarlo y a explicárselo a la audiencia. En su viaje va **conociendo a las
demás mascotas** — Chip (el viejo robot que ya vivió todas las eras tech), Sonda (la dron que husmea las
noticias) y Tera (la IA sabia) — que se van uniendo al canal. Esto da una **narrativa de crecimiento**:
el canal y su mundo se construyen frente a la audiencia.

**Cronología por fases** (el canal se integra a las redes **poco a poco**, no arranca a tope):
- **Fase 0 — Despertar y presentación (primeros ~5 videos):** "¿Quién es Bit?", "¿Qué es Tecnobichos?",
  Bit despierta y se presenta; conceptos base (perenne). Sienta tono e identidad. *Solo Bit.*
- **Fase 1 — El mundo crece (videos ~6–15):** Bit **va conociendo a las mascotas** una a una, cada una
  estrenada en el video donde tiene sentido (un debate estrena a Chip; una noticia estrena a Sonda; un
  concepto de IA estrena a Tera). Mezcla 60/40 perenne/tendencia.
- **Fase 2 — Ritmo de canal (16+):** elenco completo activo, cadencia regular, mezcla 70/30 tendencia/perenne,
  engagement con la audiencia.

La skill detecta la fase contando cuántos videos lleva el canal y elige el contenido (y qué mascota
introducir) según corresponda a la cronología.

## 6. Reglas para la skill de generación de temas/guiones

La skill (ejecutada periódicamente por el dueño) debe:
1. **Leer `cc_cola_contenido`** (temas ya publicados/en cola) → **NO repetir** temas ni ángulos.
2. **Saber en qué fase está el canal** (contar cuántos videos hay) → elegir contenido coherente con la cronología (§5).
3. **Analizar tendencias** (HN, dev.to, GitHub Trending, noticias IA de OpenAI/Anthropic/Google, Reddit, Google Trends, redes) → no publicar por publicar.
4. **Generar temas + guiones** con el tono de §1, el formato/pilar adecuado (§3) y la mascota correcta (§2).
5. **Insertar** los temas aprobados en `cc_cola_contenido` (`estado='pendiente'`) para que la automatización los produzca.
6. Respetar brand safety (parodia de productos OK; nada de citas falsas de personas reales).

---
*Documento vivo. Se actualiza con cada decisión de marca.*
