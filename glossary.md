# Glosario técnico — CCA-F

Vocabulario clave que aparece de forma literal en el examen y en la documentación oficial. Memoriza los **nombres exactos** y los **valores válidos** (no parafrasear).

---

## A — API y agentes

### `AgentDefinition`
Estructura del Claude Agent SDK que define un subagente: incluye su **descripción**, su **system prompt**, las **tools permitidas** y otras restricciones. Es lo que el coordinador usa para spawnear y validar la invocación de un subagente.

### Agentic loop (bucle agéntico)
Ciclo iterativo: el modelo emite una respuesta → si `stop_reason == "tool_use"` ejecutamos las tools → añadimos los `tool_result` al historial → enviamos nueva request → repetimos hasta `stop_reason == "end_turn"`.

### `allowed-tools` (frontmatter de skill/command)
Lista YAML en el frontmatter de una Skill o slash command que restringe las tools que el agente puede usar durante esa invocación.

### `allowedTools` (opción del Agent SDK)
Equivalente programático: array de nombres de tools que el agente puede invocar. Para spawnear subagentes hay que incluir `"Task"`.

### `argument-hint` (frontmatter de skill/command)
Texto corto que indica al modelo qué argumentos espera el comando/skill — guía la invocación pero no la fuerza.

---

## C — Claude Code y configuración

### `.claude/CLAUDE.md`
Archivo de configuración a **nivel de proyecto** (también puede vivir en la raíz como `CLAUDE.md`). Se carga automáticamente al iniciar Claude Code en ese proyecto.

### `~/.claude/CLAUDE.md`
Archivo a **nivel de usuario**. Personal, **no se versiona** con el proyecto.

### `.claude/commands/`
Carpeta de **slash commands de proyecto** (compartidos vía git).

### `~/.claude/commands/`
Carpeta de **slash commands de usuario** (personales).

### `.claude/rules/`
Carpeta de **reglas con frontmatter YAML**: cada archivo `.md` puede llevar un campo `paths` con globs (`paths: ["src/**/*.test.ts"]`) — se carga sólo cuando se edita un archivo que matchea.

### `.claude/skills/`
Carpeta de **Skills**: bloques de instrucciones que se invocan on-demand vía un nombre. Frontmatter típico: `name`, `description`, `context: fork`, `allowed-tools`, `argument-hint`.

### `context: fork`
Valor de frontmatter que indica que la Skill debe ejecutarse en un **contexto aislado** (subagente) para no contaminar la sesión principal con outputs verbosos.

### `@import`
Sintaxis dentro de un `CLAUDE.md` para incluir otro archivo markdown — permite modularizar la configuración.

### `--print` / `-p`
Flag de Claude Code para ejecutar en **modo no interactivo** (one-shot). Imprime la respuesta y sale. Esencial para CI/CD.

### `--output-format json`
Flag que pide la respuesta como JSON estructurado en lugar de prosa.

### `--json-schema`
Flag que permite pasar un schema JSON al que Claude debe ajustar su salida.

### `--resume <session-name>`
Reanuda una sesión guardada concreta.

### `/memory`
Comando interno de Claude Code que muestra **qué archivos de memoria** (`CLAUDE.md`, rules) están actualmente cargados en contexto.

### `/compact`
Comando interno que **comprime el contexto** de la sesión actual — útil cuando la conversación se hace muy larga.

---

## E — Errores y MCP

### `end_turn`
Valor de `stop_reason` que indica que el modelo **terminó su turno** sin pedir más tools. Es la señal correcta para salir del agentic loop.

### `isError` (MCP)
Flag booleano en la respuesta de una MCP tool que indica que el resultado es un error. Permite que el modelo distinga error de output normal.

### `errorCategory`
Metadato sugerido (no estándar fijo) para clasificar errores: `transient`, `validation`, `business`, `permission` — guía la decisión de retry vs escalar.

### `isRetryable`
Booleano sugerido que indica si tiene sentido reintentar la operación que falló.

---

## F — Forking y batching

### `fork_session`
Mecanismo del Agent SDK para crear **ramas paralelas** desde un baseline compartido — útil para explorar varias hipótesis sin contaminar la sesión original.

### Message Batches API
API de Anthropic que permite enviar **lotes grandes** de requests con **50% de descuento**, ventana de respuesta de hasta **24 horas** y **sin SLA de latencia**.

### `custom_id`
Campo obligatorio en cada request de un batch para **correlacionar** la respuesta con su request original.

---

## H — Hooks y handoff

### `PreToolUse`
Hook del Agent SDK que se dispara **antes** de ejecutar una tool — puede bloquear la llamada o modificar el input.

### `PostToolUse`
Hook que se dispara **después** de ejecutar una tool — permite normalizar/transformar el resultado antes de que el modelo lo procese.

### Handoff
Protocolo estructurado para transferir un caso a otro agente (o a un humano) con todo el contexto necesario (IDs, root cause, monto, acción recomendada).

### Hub-and-spoke
Arquitectura multi-agente donde un **coordinador central** habla con todos los subagentes y los subagentes **no se hablan entre sí** directamente.

---

## J — JSON y prompting

### JSON Schema (en tool definition)
Schema que define la **forma del input** que la tool espera — Anthropic lo valida y el modelo se ajusta a él.

### Few-shot prompting
Técnica de prompting que incluye **2-4 ejemplos** input/output dentro del prompt para guiar al modelo. Más efectiva que descripciones detalladas cuando hay ambigüedad.

---

## M — MCP

### MCP (Model Context Protocol)
Protocolo abierto que estandariza cómo Claude (y otros LLMs) se conectan a fuentes de datos externas y tools. Spec en <https://modelcontextprotocol.io>.

### `.mcp.json`
Archivo de configuración MCP a **nivel de proyecto** — define qué servidores MCP están disponibles.

### `~/.claude.json`
Archivo de configuración a **nivel de usuario** (no `~/.mcp.json` — ojo con la confusión). Define MCP servers personales.

### MCP Resources
Conceptos del protocolo: además de tools, un servidor MCP puede exponer **recursos** (catálogos de contenido como issues, schemas, documentos) que el modelo lee directamente — reduce tool calls exploratorios.

### MCP Tools
Operaciones expuestas por un servidor MCP. Cada una tiene nombre, descripción, input schema y devuelve un resultado (con flag `isError` si falla).

---

## P — Plan mode y planning

### Plan mode
Modo de Claude Code en que el agente **investiga y propone** un plan **sin ejecutar cambios destructivos**. Útil antes de cambios grandes o multi-archivo.

---

## S — Sesión y stop reasons

### Session forking
Ver `fork_session`.

### `stop_reason`
Campo de la respuesta de la Messages API. Valores: `"end_turn"`, `"tool_use"`, `"max_tokens"`, `"stop_sequence"`. **Es la señal canónica** para decidir si seguir iterando el agentic loop.

### Subagent
Agente spawneado por un coordinador, con **contexto aislado** y un `AgentDefinition` propio. No ve el historial del coordinador a menos que se le pase explícitamente.

---

## T — Tools

### Task tool
Tool built-in que **spawnea un subagente** dentro del Agent SDK. Para usarla, el agente que la invoca debe tener `"Task"` en su `allowedTools`.

### `tool_choice`
Parámetro de la Messages API que controla cómo el modelo elige tools:
- `"auto"` — el modelo decide si llamar a alguna tool o no.
- `"any"` — fuerza al modelo a llamar **alguna** tool, pero elige cuál.
- `{"type": "tool", "name": "..."}` — fuerza la tool **específica**.

### `tool_use`
Valor de `stop_reason` que indica que el modelo quiere ejecutar una o más tools. Hay que ejecutarlas y devolver `tool_result` en el siguiente turno.

### `tool_result`
Bloque que se devuelve en la siguiente request, con `tool_use_id` correspondiente. Es lo que cierra la iteración del loop.

---

## Built-in tools de Claude Code

| Tool | Para qué |
|---|---|
| `Read` | Leer un archivo completo (con offset/limit opcional). |
| `Write` | Crear o sobrescribir un archivo completo. |
| `Edit` | Modificación puntual con string de anclaje único (debe ser único en el archivo). |
| `Bash` | Ejecutar comandos shell. |
| `Grep` | Búsqueda por contenido (patrones, regex). |
| `Glob` | Matching de paths (`**/*.test.tsx`). |

---

## Frontmatter YAML (resumen rápido)

```yaml
---
name: my-skill              # Identificador
description: ...            # Cuándo invocar (clave para que el modelo la descubra)
context: fork               # Aislar en subagente
allowed-tools: [Read, Grep] # Whitelist de tools
argument-hint: "<filepath>" # Pista de argumento esperado
paths:                      # Sólo para .claude/rules/ — globs de aplicación
  - "src/**/*.test.ts"
---
```

---

## Recordatorios de examen

- `stop_reason == "end_turn"` → salir del loop. **No parsear texto** para decidir.
- `tool_choice = "any"` ≠ `"auto"`. `"any"` **fuerza** una tool, `"auto"` deja elegir.
- `~/.claude.json` (user-level MCP) ≠ `~/.mcp.json` (no existe).
- `fork_session` ≠ `--resume`. Fork crea **rama paralela**, resume **continúa** la misma.
- `Edit` falla si el string no es único → fallback a `Read` + `Write`.
- Subagentes **no heredan** historial automáticamente — hay que pasarlo en el prompt.
- Batch API: **no soporta multi-turn tool calling** en una sola request.
