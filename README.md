# Claude Multi-Agent Template

Sistema multi-agente para Claude Code con arquitectura CEO → Leads → Specialists. Copia esto a `~/.claude/` y tendrás un orquestador completo que rutea tareas por dominio automáticamente.

---

## Arquitectura

```
CEO (orquestador)
├── tech-lead        → java-specialist, python-specialist, ui-ux-specialist, seguridad-specialist, planificador-specialist, memoria-specialist
├── business-lead    → contratos-mx-specialist
├── finance-lead     → [tus skills de parseo bancario]
└── wellness-lead    → nutricion-specialist*, fuerza-specialist*, movilidad-specialist*, calendario-specialist*
```

> `*` Los specialists de wellness no están incluidos en este template — son altamente personales. El wellness-lead funciona sin ellos usando conocimiento general.

---

## Instalación rápida

```bash
# 1. Clona el repo
git clone https://github.com/[usuario]/claude-agents-template.git

# 2. Copia a ~/.claude
cp -r claude-agents-template/commands/* ~/.claude/commands/
cp -r claude-agents-template/agents/* ~/.claude/agents/
cp claude-agents-template/CLAUDE.md ~/.claude/CLAUDE.md  # cuidado: sobreescribe el tuyo

# 3. Instala los MCPs (ver sección MCPs abajo)

# 4. Personaliza los placeholders (ver sección Personalización)
```

---

## MCPs — Configuración

### MCPs via CLI de Claude Code

```bash
# markitdown — convierte PDF, Word, Excel, imágenes a Markdown
claude mcp add markitdown --transport stdio -- uvx markitdown-mcp

# playwright — automatización de browser (LinkedIn, scraping, formularios)
claude plugin install playwright

# github — acceso a repos, commits, PRs
claude plugin install github

# context7 — documentación de librerías actualizada
claude plugin install context7
```

### MCPs HTTP (sin instalación local)

```bash
# Microsoft Learn — docs oficiales de Microsoft y Azure
claude mcp add microsoft-learn --transport http https://learn.microsoft.com/api/mcp

# RxResume — gestión de CVs (si vas a usar el agente HR-ATS)
claude mcp add rxresume --transport http https://rxresu.me/mcp

# Notion — workspaces y bases de datos
claude mcp add notion --transport http https://mcp.notion.com/mcp
```

> Los MCPs HTTP con OAuth (RxResume, Notion) pedirán autenticación en el navegador la primera vez que los uses.

### MCPs via claude.ai (OAuth en el navegador)

Estos se conectan desde [claude.ai](https://claude.ai) → **Settings → Integrations**:

| MCP | Uso |
|-----|-----|
| Google Drive | Acceso a archivos en Drive |
| Gmail | Lectura y redacción de emails |
| Google Calendar | Requerido por `wellness-lead` para agendar workouts |
| Canva | Assets visuales para `business-lead` |
| Microsoft 365 | Outlook, Teams, SharePoint |

### Verificar instalación

```bash
claude mcp list
# Deberías ver: markitdown ✔, microsoft-learn ✔, rxresume ✔, notion ✔
# playwright, github, context7 aparecen como plugins
```

---

## Personalización

Busca y reemplaza estos placeholders en todos los archivos:

| Placeholder | Reemplazar con |
|-------------|---------------|
| `[TU_NOMBRE]` | Tu nombre |
| `[TU_EMPRESA]` | El nombre de tu empresa o proyecto principal |
| `[TU_PROYECTO]` | Tu proyecto de software actual |
| `[TU_VAULT]` | Nombre de tu vault de Obsidian (si usas ObsidianMind) |
| `[TU_CARPETA_SALUD]` | Path a tu carpeta de datos fitness, ej: `~/Personal/Fitness` |
| `[TU_BANCO_1]`, `[TU_BANCO_2]` | Nombres de tus bancos en `agent-lead-finance.md` |
| `[TU_PATH]` | Tu username de macOS, ej: `juangarcia` → `-Users-juangarcia` |

```bash
# Reemplazar en todos los archivos de una vez (macOS):
find ~/.claude/commands ~/.claude/agents -name "*.md" -exec \
  sed -i '' 's/\[TU_NOMBRE\]/Juan García/g' {} \;
```

---

## Vault de conocimiento (opcional pero recomendado)

Los agentes usan `qmd` para búsqueda semántica en un vault de Obsidian. Sin esto, los agentes funcionan pero sin memoria histórica entre sesiones.

**Opciones:**
1. **ObsidianMind** (la que usamos nosotros) — vault de Obsidian + `qmd` para búsqueda semántica
2. **Cualquier carpeta de Markdown** — los agentes pueden hacer `grep` directamente
3. **Sin vault** — elimina las líneas `qmd` de los agentes y usa solo la memoria de agentes (`global-memory.md`)

---

## Uso

Una vez instalado, escribe cualquier tarea en Claude Code y el CEO la ruteará automáticamente:

```
# Ejemplos — el CEO decide a quién delegar:

"Arregla el bug en el servicio de pagos"
→ CEO → tech-lead → java-specialist

"Revisa este contrato antes de firmarlo"
→ CEO → business-lead → contratos-mx-specialist

"¿Cuánto gasté este mes?"
→ CEO → finance-lead → [tu skill de banco]

"Dame mi plan de entrenamiento para hoy"
→ CEO → wellness-lead → fuerza-specialist + nutricion-specialist + calendario-specialist

"Diseña la landing page de mi SaaS"
→ CEO → tech-lead → ui-ux-specialist
```

---

## Agregar specialists propios

Usa el comando `/agent-creator` en Claude Code — él te guía para crear un nuevo specialist con el formato correcto y lo integra automáticamente en la jerarquía.

```
# En Claude Code:
/agent-creator
```

---

## Estructura de archivos

```
~/.claude/
├── CLAUDE.md                          # Instrucciones globales — activa el CEO automáticamente
├── commands/
│   ├── agent-CEO.md                   # Orquestador
│   ├── agent-lead-tech.md             # Lead de desarrollo
│   ├── agent-lead-business.md         # Lead de negocio
│   ├── agent-lead-finance.md          # Lead de finanzas
│   ├── agent-lead-wellness.md         # Lead de salud/fitness
│   ├── agent-specialist-java.md
│   ├── agent-specialist-python.md
│   ├── agent-specialist-ui-ux.md
│   ├── agent-specialist-seguridad.md
│   ├── agent-specialist-planificador.md
│   ├── agent-specialist-memoria.md
│   └── agent-specialist-contratos-mx.md
└── agents/
    ├── CEO/global-memory.md           # Memoria persistente del CEO
    ├── tech-lead/global-memory.md
    ├── business-lead/global-memory.md
    ├── finance-lead/global-memory.md
    └── wellness-lead/global-memory.md
```

---

## Cómo funciona la memoria

Cada agente tiene un `global-memory.md` donde registra aprendizajes entre sesiones. El formato es:

```
- 2026-06-14: [descripción del aprendizaje]
```

Estos archivos crecen solos — cada agente los actualiza al terminar una tarea.

Para memoria a nivel de proyecto, los agentes buscan `{proyecto}/.claude/agents/{nombre}/memory.md`. Si no existe, lo crean.

---

## Contribuciones

PRs bienvenidos — especialmente nuevos specialists o mejoras a los leads existentes.
