---
name: prompt-audit
description: Audits how the user writes their prompts in Claude/Cowork by analyzing their real recent sessions, and returns a 0-100 score with a visual scorecard and concrete recommendations. Use whenever the user asks to evaluate, audit, score, or improve their prompting — e.g. "how good are my prompts", "audit my prompts", "prompt audit", "rate my prompting", "cómo estoy prompteando", "analizá mis prompts", "dame un puntaje de mis prompts" — or wants a one-off or recurring report on the quality of their instructions to Claude. The report is written in the user's language.
---

# Prompt audit

Analiza los prompts reales del usuario en sesiones recientes de Cowork y produce: un puntaje global 0-100, un scorecard visual por dimensión, y recomendaciones accionables basadas en citas textuales.

El valor de esta skill está en usar evidencia real. Nunca inventes ni parafrasees citas: si citás un prompt del usuario, tiene que ser textual (typos incluidos — los typos son parte del diagnóstico).

## Paso 1 — Recopilar prompts

1. Usá las herramientas de sesiones disponibles en el entorno para listar sesiones recientes (limit 20-30). El nombre exacto varía según el entorno: buscá tools tipo `list_sessions` / `read_transcript` / `search_session_transcripts` (en Cowork suelen ser `mcp__session_info__*`; en otros entornos pueden colgar de otro servidor). Si no aparecen en tu lista de tools, buscalas con ToolSearch antes de asumir que no existen; si realmente no hay acceso a sesiones, explicáselo al usuario y no inventes datos.
2. Elegí una muestra diversa de 6-10 sesiones: mezclá dominios (código, negocio, escritura, investigación, personal). Excluí la sesión actual y las corridas de scheduled tasks (no contienen prompts escritos por el usuario). Si esta auditoría es recurrente (ej. diaria), priorizá sesiones nuevas o activas desde la última corrida; si no hay suficientes, completá con las más recientes.
3. Leé la transcripción de cada sesión elegida con la tool correspondiente (limit 25-40 mensajes; si acepta `max_wait_seconds`, pasá 0; en paralelo cuando puedas).
4. Extraé solo los mensajes `[user]` escritos por la persona. Descartá: expansiones de skills (empiezan con "Base directory for this skill"), archivos adjuntos, e imágenes. Necesitás al menos ~8 prompts reales; si hay menos, ampliá la muestra de sesiones antes de puntuar.

## Paso 2 — Puntuar

Evaluá 6 dimensiones, cada una 0-10. El puntaje global es el promedio × 10, redondeado (0-100).

| Dimensión | Qué mide |
|---|---|
| Contexto y objetivo de negocio | ¿Da contexto rico, metas medibles, restricciones del mundo real? |
| Iteración y feedback específico | ¿Corrige con feedback quirúrgico ("esta frase está fea, la diría invertida") o con vaguedades ("mejoralo")? |
| Técnicas avanzadas | Roles ("sos un consultor de X"), anti-sesgo ("sé honesto, no intentes caerme bien"), handoffs comprimidos entre agentes, uso de scheduled tasks. |
| Claridad del pedido | ¿Se entiende sin adivinar qué quiere? ¿Los términos clave están bien escritos o el dictado/typos generan ambigüedad? |
| Formato de salida y criterios de éxito | ¿Especifica entregable, extensión, estructura y criterio de priorización upfront, o lo pide en un segundo turno? ¿Pide verificación ("no estimes datos")? |
| Estructura (un pedido a la vez) | ¿Prompts enfocados, o mega-prompts con 10 pedidos sin jerarquía e instrucciones contradictorias? |

Calibración: 40-55 = usuario casual; 56-70 = intermedio; 71-85 = avanzado-intermedio (usa técnicas que pocos usan pero pierde puntos en formato/estructura); 86+ = reservalo para prompting consistentemente excelente en todas las dimensiones. Sé honesto, no inflaciones el puntaje: el usuario pidió explícitamente este análisis para mejorar.

## Paso 3 — Scorecard visual

Si tenés las herramientas `mcp__visualize__*`: llamá `read_me` (módulo `data_viz`) y después `show_widget` con este layout (adaptá puntajes y colores):

- Izquierda: tile con el puntaje global grande (font-voice, ~52px), "de 100" y el nivel debajo; a la derecha una línea indicando en cuántas sesiones se basó.
- Debajo: una barra horizontal por dimensión con etiqueta y n/10. Colores por rango: 8-10 verde `#1D9E75` (fortaleza), 6-7 azul `#2a78d6` (sólido), 0-5 ámbar `#EF9F27` (a mejorar).
- Al pie: leyenda de los tres colores.
- Usá variables CSS (`var(--surface-1)`, `var(--text-primary)`, etc.) para que funcione en dark mode.

Si no hay herramientas de visualización, presentá el scorecard como tabla markdown con el puntaje global arriba.

## Paso 4 — Prosa (fortalezas y recomendaciones)

Después del widget, escribí en prosa (sin bullets excesivos), en el idioma del usuario:

1. Arrancá con el puntaje y una frase de posicionamiento ("estás por encima del usuario promedio" solo si es cierto).
2. Fortalezas con citas textuales de sus propios prompts entre comillas, explicando por qué cada técnica funciona.
3. Las 2-3 debilidades que más puntos cuestan, cada una con: el ejemplo real citado, por qué es un problema, y un fix concreto reescrito ("mismo contenido, pero cerrá con: prioridad 1: X. Formato: Y"). El fix reescrito es lo más valioso del reporte — no des consejos genéricos tipo "sé más claro".
4. Si la auditoría es recurrente, comparó contra el puntaje anterior si aparece en el contexto de la tarea, y señalá qué cambió.

Cerrá ofreciendo un siguiente paso útil (ej. template de prompt para tareas grandes basado en sus patrones).
