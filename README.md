# Claude Multi-Agent Template

Sistema multi-agente para Claude Code con arquitectura TPM + especialistas por dominio. Copia esto a `~/.claude/` y tendrás orquestación automática que rutea tareas según el contexto.

---

## Arquitectura

```
/agent-TPM    → desarrollo, scripting, automatización, arquitectura
/agent-WAPT   → web application penetration testing
/agent-RED    → red team, active directory, internal network
/agent-EM     → engagement management, cronogramas, comunicación con clientes
/agent-ACCESS → preflight check: verifica conectividad de targets antes de iniciar pruebas
```

El TPM internamente puede rutear a WAPT, RED, EM o ACCESS si la tarea toca esos dominios.

---

## Instalación rápida

```bash
# 1. Clona el repo
git clone https://github.com/emiyelbarto/claude-agents-template.git

# 2. Backup de lo que ya existe en ~/.claude
BACKUP_DIR=~/.claude-backup-$(date +%Y%m%d-%H%M%S)
mkdir -p "$BACKUP_DIR"
[ -f ~/.claude/CLAUDE.md ]    && cp ~/.claude/CLAUDE.md "$BACKUP_DIR/"
[ -d ~/.claude/commands ]     && cp -r ~/.claude/commands "$BACKUP_DIR/"
[ -d ~/.claude/agents ]       && cp -r ~/.claude/agents "$BACKUP_DIR/"
echo "Backup guardado en $BACKUP_DIR"

# 3. Copia a ~/.claude
cp -r claude-agents-template/commands/* ~/.claude/commands/
cp -r claude-agents-template/agents/* ~/.claude/agents/
cp claude-agents-template/CLAUDE.md ~/.claude/CLAUDE.md

# 4. Instala los MCPs (ver sección MCPs abajo)

# 5. Personaliza los placeholders (ver sección Personalización)
```

---

## MCPs — Configuración

### MCPs via CLI de Claude Code

```bash
# markitdown — convierte PDF, Word, Excel, imágenes a Markdown
claude mcp add markitdown --transport stdio -- uvx markitdown-mcp

# nvd-cve — consulta CVEs desde la base de datos NVD
claude mcp add nvd-cve --transport stdio -- npx -y nvd-cve-mcp-server

# metasploit — acceso a módulos, hosts y vulnerabilidades (Metasploit Framework v6.4+)
# Requiere: msfdb init && msfdb start
claude mcp add metasploit --transport stdio -- msfmcpd

# github — acceso a repos, commits, PRs
claude plugin install github
```

### Burp Suite MCP (Community y Pro)

```bash
# 1. Instalar extensión desde BApp Store: "MCP Server" (by PortSwigger)
# 2. En Burp: pestaña MCP → "Install to Claude Desktop"
#    Esto genera la config automáticamente apuntando a http://127.0.0.1:9876/sse
# Ref: https://github.com/PortSwigger/mcp-server
```

### MCPs via claude.ai (OAuth en el navegador)

Estos se conectan desde [claude.ai](https://claude.ai) → **Settings → Integrations**:

| MCP | Uso |
|-----|-----|
| Google Drive | Acceso a archivos en Drive |
| Gmail | Lectura y redacción de emails — agent-EM para comunicación con clientes |
| Google Calendar | Gestión de calendario |
| Canva | Diseño y assets visuales |
| Microsoft 365 | Outlook, Teams, SharePoint |

### Verificar instalación

```bash
claude mcp list
# Deberías ver: markitdown, nvd-cve, metasploit
# github aparece como plugin
```

---

## Personalización

Busca y reemplaza estos placeholders en todos los archivos:

| Placeholder | Reemplazar con |
|-------------|---------------|
| `[TU_NOMBRE]` | Tu nombre completo |
| `[TU_VAULT]` | Nombre de tu vault de Obsidian (si usas ObsidianMind) |
| `[TU_PATH]` | Tu username, ej: `juangarcia` → `-Users-juangarcia` |

```bash
# Reemplazar en todos los archivos de una vez (macOS):
find ~/.claude/commands ~/.claude/agents -name "*.md" -exec \
  sed -i '' 's/\[TU_NOMBRE\]/Tu Nombre Aqui/g' {} \;
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

Una vez instalado, escribe cualquier tarea en Claude Code y el agente correcto toma control:

```
# El CLAUDE.md activa el agente adecuado según el dominio:

"Necesito un script para automatizar la recolección de logs del servidor"
→ /agent-TPM → resuelve directamente

"Revisa el cronograma de la auditoría y responde al correo del cliente con las fechas acordadas"
→ /agent-EM → resuelve directamente

"Ejecuta un reconocimiento web inicial sobre el nuevo alcance e identifica posibles vectores de ataque"
→ /agent-WAPT → resuelve directamente

"Genera el plan de pruebas para evaluar la seguridad de la red interna de este Directorio Activo"
→ /agent-RED → resuelve directamente

"Lanza una auditoría completa combinada web e interna"
→ /agent-TPM → ruteó a agent-WAPT + agent-RED en paralelo

"Verifica que todos los targets del scope estén accesibles antes de empezar"
→ /agent-ACCESS → tabla de conectividad en segundos
```

---

## Agregar agents propios

Usa el comando `/agent-creator` en Claude Code — te guía para crear un nuevo agente con el formato correcto y lo integra automáticamente en la jerarquía.

```
# En Claude Code:
/agent-creator
```

---

## Estructura de archivos

```
~/.claude/
├── CLAUDE.md                          # Instrucciones globales — activa el agente correcto automáticamente
├── commands/
│   ├── agent-TPM.md                   # Orquestador y Technical Program Manager
│   ├── agent-WAPT.md                  # Senior Web Application Penetration Tester
│   ├── agent-RED.md                   # Senior Red Team Penetration Tester
│   ├── agent-EM.md                    # Engagement Manager
│   └── agent-ACCESS.md               # Intern de pre-engagement — preflight check
└── agents/
    ├── TPM/global-memory.md           # Memoria persistente del TPM
    ├── WAPT/global-memory.md          # Memoria persistente del WAPT
    ├── RED/global-memory.md           # Memoria persistente del RED
    ├── EM/global-memory.md            # Memoria persistente del EM
    └── ACCESS/global-memory.md        # Memoria persistente del ACCESS
```

---

## Cómo funciona la memoria

Cada agente tiene un `global-memory.md` donde registra aprendizajes entre sesiones. El formato es:

```
- 2026-06-14: [descripción del aprendizaje]
```

Estos archivos crecen solos — cada agente los actualiza al terminar una tarea.

Para memoria a nivel de proyecto, los agentes buscan `{proyecto}/.claude/agents/{nombre}/memory.md`. Si no existe, lo crean.
