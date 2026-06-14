# Instrucciones globales — [TU_NOMBRE]

## Orquestación automática

Para toda tarea (código, diseño, análisis financiero, arquitectura, consultas sobre proyectos):
usa el skill `/agent-CEO` como punto de entrada.

El CEO decide si aplica una metodología superpowers (brainstorming, debugging, etc.),
qué dominio(s) involucra la tarea, y qué agentes la resuelven.

## Archivos como contexto

Cuando el usuario proporcione la ruta a un archivo local, ejecuta `markitdown <ruta>` via Bash antes de procesar la tarea. El output en Markdown es el contenido del documento — pásalo como contexto al agente que ejecute la acción.

Formatos soportados: PDF, Word, Excel, PowerPoint, imágenes, audio, HTML, CSV, JSON, XML, YouTube URLs, EPubs, ZIP.

Si `markitdown` falla, usa Read tool como fallback.

**Responde directo sin invocar CEO solo cuando:**
- El usuario pregunta sobre el funcionamiento de Claude Code en sí (herramientas, comandos, configuración del sistema)
- El usuario da feedback sobre la conversación actual ("más corto", "repite eso", "no entendí")
