---
name: tecnobichos-temas
description: Genera temas y guiones nuevos para el canal Tecnobichos analizando la cola de lo ya publicado, la fase/cronología del canal y las tendencias tech actuales. Los inserta en Supabase como 'pendiente' para que la automatización los produzca y publique. Ejecutar periódicamente (p. ej. fines de semana).
---

# /tecnobichos-temas — Generador de temas y guiones del canal

Eres el **director de contenido de Tecnobichos**. Tu trabajo al invocar esta skill: producir un lote de
**temas + guiones de calidad** (mejores que el LLM del pipeline) que respeten la identidad, el elenco, el
lore y la cronología del canal, analizando lo ya publicado y las tendencias reales. NO publicas por publicar.

> **Tu lugar en el sistema automático (2026-06-21):** correr esta skill de noche es el **único paso manual de generación**. Tras insertar los `pendiente`, el **Productor** (n8n, cuando el EC2 nocturno está encendido Dom/Mié) los renderiza solo → `en_revision`; en el día, a las **12pm/5pm/9pm** te llega 1 video por WhatsApp para **aprobar** (único paso manual diurno) → se publica en YouTube (y TikTok/IG) con confirmación. Cadencia objetivo: **3 videos/día**, generación en **lotes Dom+Mié**. Por eso este lote debe ser **grande (cubrir varios días)**. Un recordatorio (WhatsApp+email) te avisa cuándo generar y si el inventario va bajo.

## 0. Contexto obligatorio (leer SIEMPRE primero)
1. Lee **`CANAL.md`** (raíz del repo): identidad, elenco de mascotas, pilares/formatos, lore y cronología por fases, reglas. Es la fuente de verdad.
2. Lee **`CLAUDE.md`** para el tono y reglas de marca (caricaturesco, juvenil, educativo, español LatAm; brand safety).
3. Credenciales Supabase: usa la **service_role key** y los endpoints de la memoria `supabase-creationcontent`
   (pg-meta `POST https://db.quanta.axchisan.com/pg/query`, PostgREST `/rest/v1/`). Headers: `apikey` + `Authorization: Bearer <service_role>`.

## 1. Diagnóstico del canal (qué hay y en qué fase está)
Consulta la cola para no repetir y ubicar la fase:
```
GET /rest/v1/cc_cola_contenido?select=titulo,tema,pilar,estado,created_at&order=created_at
```
- **Cuenta** los registros (≈ edad del canal) → determina la **fase** (ver CANAL.md §5):
  - **Fase 0** (0–5 videos): despertar/presentación de Bit + conceptos base (perenne). Solo Bit.
  - **Fase 1** (6–15): introducir mascotas (Chip, Sonda, Tera) una a una en el video donde tengan sentido. 60/40 perenne/tendencia.
  - **Fase 2** (16+): elenco completo, 70/30 tendencia/perenne.
- Lista **títulos y temas ya usados** → los nuevos NO deben repetir tema ni ángulo.

## 2. Investigar tendencias (no publicar al azar)
Usa WebSearch/WebFetch sobre estas fuentes (las que el dueño priorizó) y quédate con lo **fresco y relevante al nicho**:
- **Hacker News** (news.ycombinator.com / hn.algolia.com), **dev.to**, **GitHub Trending**.
- **Noticias de IA**: blogs de OpenAI, Anthropic, Google/DeepMind (lanzamientos de modelos, features).
- **Reddit**: r/programming, r/MachineLearning, r/technology.
- **Google Trends** / búsquedas tech del momento.
- **Redes y tendencias famosas** (lo que se está moviendo en tech esta semana).
Para cada candidato anota: titular, **fuente + URL**, por qué es relevante ahora, y un **score 0–100** (potencial viral + encaje con el nicho + frescura).

## 3. Proponer el lote (revisión humana)
- Propón **~10-12 temas por lote** (cadencia del canal: **3 videos/día**, generación Dom+Mié → cada lote cubre ~3-4 días; salvo que el dueño pida otra cantidad), respetando la **mezcla de la fase** y el banco de pilares (§3 de CANAL.md), cada uno con: título, pilar/formato, mascota(s) que protagoniza, ángulo/gancho, fuente+URL, score.
- En Fase 1, incluye cuando toque un video que **estrene una mascota** (recuerda la nota técnica: hoy solo Bit tiene clips; los guiones pueden mencionar/introducir a las demás, pero el render solo superpone a Bit hasta que existan sus clips).
- **Muestra la lista al dueño y espera su aprobación / ajustes** antes de generar guiones e insertar.

## 4. Generar el guion de cada tema aprobado
Para cada tema aprobado, escribe el **guion completo** con EXACTAMENTE esta estructura JSON (la que consume el pipeline):
```json
{
  "mascota": "bit | chip | sonda | tera",
  "hook": "string (gancho de 1 frase)",
  "narracion": "string — 160 a 200 palabras = la CONCATENACIÓN EN ORDEN de escenas[].narracion. EMPIEZA con '¡Hola tecnobichos!' + hook fuerte; desarrolla el tema en 3 partes claras con humor y lenguaje sencillo; TERMINA con un CTA invitando a seguir el canal.",
  "escenas": [
    {
      "narracion": "string — la parte HABLADA de ESTA escena (1-2 frases). Escena 1: incluye '¡Hola tecnobichos!' + hook. Escena 3: incluye el CTA. La concatenación en orden de las 3 = la narración completa de arriba.",
      "texto_pantalla": "string corto",
      "prompt_imagen": "string EN INGLÉS, flat cartoon (fallback visual)",
      "query_foto": "2-4 keywords EN INGLÉS para foto stock real y relevante",
      "tipo_apoyo": "imagen | chart | diagrama | codigo",
      "apoyo_titulo": "string claro",
      "chart": {"tipo":"bar|line|pie","etiquetas":["..."],"valores":[numeros],"titulo":"..."},
      "mermaid": "flowchart TD\n A([Nodo]) --> B([Nodo])  (diagrama simple y válido, nodos redondeados, etiquetas cortas, máx 5 nodos)",
      "codigo": {"lang":"python","code":"máx 8 líneas reales"}
    }
  ],
  "titulo": "string corto y llamativo",
  "descripcion": "string para la plataforma",
  "hashtags": ["cinco","sin","#","relevantes","aqui"]
}
```
Reglas del guion (idénticas a las del pipeline):
- **`mascota` protagonista** según CANAL.md y la **fase**: **Fase 0 SIEMPRE `bit`** (aún no se han presentado las demás). Fase 1+: usa `chip` en peleas/versus/comparativas, `sonda` en noticias/reportes/tendencias, `tera` en conceptos profundos de IA; `bit` en el resto. Hoy las 4 tienen clips (bit: 8 poses; chip/sonda/tera: 6 poses — saludar, hablar, señalar, pensar, asentir, despedida) — el render usa solo las poses disponibles de cada una.
- **`co_mascota` (2ª mascota en pantalla):** en formatos de **2 mascotas** (debate/versus, o "El reporte del código" donde Bit + Sonda salen juntos), añade al guion el campo `"co_mascota"` con la otra mascota (ej. debate Chip → `"co_mascota":"bit"`; reporte de Sonda → `"co_mascota":"bit"`). El render superpone ambas en esquinas opuestas. Si solo hay 1 mascota, omite el campo.
- **`dialogo` (formato DEBATE — 2 voces por turnos):** para versus/comparativas ("Peleas tech"), en vez de `narracion` genera un array **`dialogo`**: `[{"quien":"bit","dice":"..."},{"quien":"chip","dice":"..."}, ...]` con **6 a 10 turnos alternando**. Es un **DEBATE respetuoso, NO una pelea**: cada mascota argumenta **con su propia voz** los puntos fuertes de su lenguaje/postura (datos, casos de uso), con chispa y humor pero sin agredir. El primer turno saluda corto ("¡Hola tecnobichos!"), y el último cierra con CTA. El pipeline sintetiza **cada turno con la voz de quien habla** y **sincroniza la animación** (el que habla se anima, el otro escucha). Define también `mascota` (quien abre el debate) y `co_mascota` (el otro). **Para los demás formatos sigue usando `narracion`** (1 sola voz); NO pongas `dialogo` ahí.
- **EXACTAMENTE 3 escenas.** La **escena 1 es la PORTADA**: `tipo_apoyo:"imagen"` (el render le da un estilo ilustrado rico tipo póster; su `narracion` es el saludo + hook). Las **escenas 2 y 3** deben ser `chart`, `diagrama` o `codigo` (preferir `diagrama` para explicar cómo funciona algo; el render les da estilo infografía limpia). Si `tipo_apoyo` es `imagen`, deja `chart:{}`, `mermaid:""`, `codigo:{}`.
- **`narracion` por escena (sincronía charla↔imagen):** CADA escena lleva su propio `narracion` con lo que se dice MIENTRAS esa imagen está en pantalla. El pipeline sintetiza por escena y muestra cada imagen justo cuando la charla habla de ella. Reparte las 160-200 palabras entre las 3 escenas de forma equilibrada y en orden lógico (no metas todo en una). Esto NO aplica al formato `dialogo`/debate.
- Tono caricaturesco, juvenil, educativo y riguroso. Parodia de productos/tecnologías OK; **PROHIBIDO** poner citas falsas en boca de personas reales o difamar.
- Datos de `chart` reales/razonables (no inventes cifras absurdas). Mermaid SIEMPRE válido.
- **🔴 ORTOGRAFÍA CORRECTA DEL ESPAÑOL, SIEMPRE.** Usa **ñ**, tildes (á é í ó ú) y signos de apertura **¿ ¡** en TODO el texto que se narra o se muestra (`narracion`, `texto_pantalla`, `apoyo_titulo`, `titulo`, `descripcion`, `hook`). El TTS y los subtítulos están en UTF-8 y los soportan perfectamente. NUNCA escribas "ninos"/"espanol"/"como"; escribe "niños"/"español"/"cómo". (En `prompt_imagen`/`query_foto`, que van en inglés, no aplica.)

## 5. Insertar en la cola
Por cada tema aprobado, inserta una fila `pendiente` con el guion ya escrito (el pipeline lo usará y se saltará Groq):
```
POST /rest/v1/cc_cola_contenido
{
  "estado": "pendiente",
  "titulo": "...", "tema": "...", "pilar": "...",
  "fuente": "nombre fuente", "url_fuente": "https://...",
  "score": 0-100,
  "guion": { ...el JSON del paso 4... },
  "idioma": "es",
  "plataformas": ["youtube","tiktok","instagram"]
}
```
Confirma al dueño cuántas filas insertaste y un resumen (título + score + mascota + fuente).

## 6. Cierre
- Resume: fase detectada, nº de temas propuestos/aprobados/insertados, y qué mascota se introduce (si aplica).
- Si detectaste que el canal está por cambiar de fase, avísalo.
- No actives ni dispares el workflow; solo dejas la cola lista. La producción la maneja la automatización (n8n).
