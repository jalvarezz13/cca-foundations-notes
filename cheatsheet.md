# Cheatsheet final — CCA-F

Repaso intensivo 24-48 h antes del examen. Una página por concepto. Memoriza los **nombres exactos** y los **valores literales**.

---

## 1️⃣ Loop agéntico

```
Request → response → ¿stop_reason?
  end_turn   → FIN
  tool_use   → ejecuta tools → añade tool_result al historial → siguiente request
  max_tokens → decisión de aplicación
  refusal    → salir
```

- **Señal canónica**: `stop_reason`. NO parsear texto.
- **Sin tope** arbitrario de iteraciones.
- `tool_result` **primero** en `content` del user message.
- `tool_result.tool_use_id` debe corresponder con el `tool_use.id`.
- N `tool_use` en un turno → N `tool_result` en el siguiente.

---

## 2️⃣ Multi-agente

- **Hub-and-spoke**: coordinador hablado con todos. Subagentes NO entre sí.
- Subagentes **NO heredan historial** — pasar contexto en el prompt.
- **Task tool** spawnea — requiere `"Task"` en `allowedTools`.
- Spawn **paralelo**: múltiples `tool_use` Task en un mismo turno.
- `fork_session` ≠ `--resume`:
  - fork → rama con historial copiado
  - resume → continúa la misma
- Descomposición fina ≠ buena: **dilución / cobertura incompleta**.
- Per-file + cross-file para review.

---

## 3️⃣ AgentDefinition (Python)

```python
@dataclass
class AgentDefinition:
    description: str       # clave para discovery
    prompt: str            # system prompt del subagente
    tools: list[str] | None
    disallowedTools: list[str] | None
    model: str | None
    skills: list[str] | None
```

---

## 4️⃣ Hooks

- `PreToolUse`: bloquea / modifica / escala antes de ejecutar.
- `PostToolUse`: normaliza / transforma / trimea el resultado.
- **Determinista** (100%). Prompt es probabilístico (~95%).
- Para reglas de negocio críticas → hook, NO prompt.

---

## 5️⃣ Tool descriptions

- **Description es el factor PRIMARIO** de selección.
- Incluir: **qué**, **cuándo**, **input format + ejemplo**, **qué devuelve**, **cómo falla**, **cuándo NO usar**.
- Mal: `analyze_content` vs `analyze_document` (solapadas) → misrouting.
- Bien: específicas y mutuamente excluyentes.
- Renombrar genéricas: `fetch_url` → `load_document` (scoped, sin SSRF).

---

## 6️⃣ tool_choice

```python
"auto"                                  # default — modelo decide
"any"                                   # fuerza alguna tool
{"type": "tool", "name": "X"}           # fuerza tool específica
"none"                                  # prohíbe tools
```

- `auto` y `none` compatibles con extended thinking.
- `any` y `tool` **NO** compatibles con extended thinking.

---

## 7️⃣ MCP

- `.mcp.json` = proyecto (versionado)
- `~/.claude.json` = usuario (personal) — **NO `~/.mcp.json`** (no existe)
- Variables: `${VAR}`, `${VAR:-default}`
- Tools (model-controlled) vs Resources (datos/contexto)
- `isError: true` en `tool_result` = error de ejecución
- JSON-RPC error = error de protocolo (no llega al modelo)
- Errores estructurados:
  - `errorCategory`: `transient` | `validation` | `business` | `permission`
  - `isRetryable`: bool
  - mensaje human-readable
- Recuperación local en subagente; propagar al coordinador sólo lo no resoluble.
- Servidores comunitarios > servers propios para integraciones estándar.

---

## 8️⃣ Tools built-in Claude Code

| Tool | Para |
|---|---|
| Read | leer archivo (con offset/limit) |
| Write | crear/sobrescribir (debe haberse leído antes si existía) |
| Edit | reemplazar string ÚNICO (o `replace_all`) |
| Bash | shell (con cuidado) |
| Grep | buscar contenido (NO bash `grep`) |
| Glob | matching de paths (NO `find`) |

Exploración incremental: Glob → Grep → Read específico → Edit.

---

## 9️⃣ CLAUDE.md jerarquía

```
~/.claude/CLAUDE.md       ← USUARIO (no versionado)
./CLAUDE.md               ← PROYECTO (versionado)
./.claude/CLAUDE.md       ← PROYECTO (alternativa)
./CLAUDE.local.md         ← LOCAL (gitignored)
<subdir>/CLAUDE.md        ← carga al leer archivos del subdir
```

- `@import` → otros .md, hasta **5 niveles** anidamiento.
- `/memory` → ver qué archivos están cargados.
- `/compact` → comprimir contexto de sesión.

---

## 🔟 Skills / Slash commands

- `.claude/skills/<name>/SKILL.md` ← proyecto
- `~/.claude/skills/<name>/` ← usuario
- `.claude/commands/<name>.md` también crea `/<name>` (convergencia)
- Frontmatter: `name`, `description`, `context: fork`, `allowed-tools`, `argument-hint`, `disable-model-invocation`, `user-invocable`
- `context: fork` → ejecuta en subagente aislado (output verboso sin contaminar)

---

## 1️⃣1️⃣ Path-specific rules

`.claude/rules/<name>.md`:
```yaml
---
paths:
  - "**/*.test.ts"
---
```
- Sin `paths` → carga al inicio.
- Con `paths` → carga al leer/editar archivo matching.
- Mejor que `<subdir>/CLAUDE.md` cuando los archivos están dispersos.

---

## 1️⃣2️⃣ Plan mode

- `/plan` o `Shift+Tab` interactivo
- CLI: `--permission-mode plan`
- Sólo tools de lectura (Read, Grep, Glob, Bash read-only)
- Para cambios grandes, multi-archivo, decisiones arquitectónicas
- Combinar con `task(subagent_type="explore")` para discovery verboso

---

## 1️⃣3️⃣ CLI no interactivo (CI/CD)

```
-p, --print               modo one-shot
--output-format json      JSON estructurado
--output-format stream-json   chunks streaming
--json-schema <s>         fuerza estructura
--bare                    sin headers
--continue                última sesión del dir
--resume <id>             sesión específica
--fork-session            rama paralela
--permission-mode plan    sólo lectura
```

- `session_id` en JSON output → guarda para reanudar.
- Una instancia genera, **OTRA** revisa (no self-review).
- Re-review: **incluir hallazgos previos** en el prompt para evitar duplicados.
- En CI: NO se monta `~/.claude/` → versiona `./CLAUDE.md` y `.claude/rules/`.

---

## 1️⃣4️⃣ Prompt engineering

- **Criterios específicos** > vagos.
- **"Be conservative" / "only high-confidence" NO mejoran precisión.**
- Severidad con **ejemplos por nivel**.
- **Few-shot 3-5 ejemplos** envueltos en `<example>` / `<examples>`.
- Incluir **negativos** (qué NO flagear) para reducir falsos positivos.
- `<thinking>` tags para razonamiento explícito.
- Few-shot supera instrucciones detalladas en casos ambiguos.

---

## 1️⃣5️⃣ Structured output

- **Tool use con schema** → máxima fiabilidad.
- `tool_choice` forzado por nombre → garantiza extracción.
- Schema garantiza **forma**, NO **semántica**.
- Self-check fields:
  - `nullable` o `*_confidence: "not_found"` cuando puede faltar.
  - enums con `"unclear"` / `"other"` + `detail`.
  - `stated_total` + `calculated_total` + `discrepancy`.
  - `conflict_detected: bool` para inconsistencias.

---

## 1️⃣6️⃣ Validation / retry

- Retry con **error feedback** (añadir errores al prompt).
- **Inútil** si la info no está en la fuente.
- Self-correction oficial: **draft → review against criteria → refine**.
- "If can't find quote, **retract the claim**."
- `max_retries` finito (3 es típico).

---

## 1️⃣7️⃣ Message Batches API

- **50%** ahorro · **24h** ventana · **sin SLA latencia**
- Asíncrona — NO para pre-merge / chat
- `custom_id` OBLIGATORIO (orden no garantizado)
- Cada item = request **independiente**
- **NO** multi-turn tool calling en una request
- Resultados en JSONL, expira a 24h
- Refinar prompt en **sample pequeña** antes de batch grande
- Cálculo de frecuencia: `SLA - 24h = ventana segura`

---

## 1️⃣8️⃣ Context management

- **Lost in the middle**: datos largos arriba, query al final.
- **Case facts persistente** en cada prompt: IDs, montos, fechas.
- **Trim tool outputs** via `PostToolUse` hook.
- **XML tags** para estructurar prompt.
- 3 técnicas oficiales: **compaction**, **tool-result clearing**, **memory**.
- `/compact` en sesiones largas.

---

## 1️⃣9️⃣ Escalado

Triggers VÁLIDOS:
1. Petición explícita de humano → **honrar inmediatamente** (no investigar antes).
2. Gap / silencio de política.
3. Incapacidad de avanzar (datos insuficientes irreparables).

NO válidos:
- Sentiment / confidence autoreportado.
- "Caso complejo".

Múltiples matches → **pedir identificador**, no adivinar.

---

## 2️⃣0️⃣ Calibración de confianza

- Métricas **agregadas ocultan** segmentos malos.
- **Stratified sampling** por idioma, formato, tipo de campo.
- **Confidence calibrado** con held-out set etiquetado.
- Routing:
  - `high` + segment validado → auto
  - `medium` → human signoff
  - `low` o `conflict_detected` → human review
  - Política exige → human (independiente de confidence)
- Monitorea **drift** semanalmente.

---

## 2️⃣1️⃣ Provenance multi-fuente

- Mappings **claim ↔ source** explícitos (URL, título, excerpt, fechas).
- **`publication_date`** + **`accessed_date`** obligatorios.
- Conflictos: **anotar con atribución**, NO elegir arbitrario.
- Render por tipo:
  - Financial → tabla
  - News → prosa cronológica
  - Técnico → listas
- **Anthropic Citations** feature: `cited_text` NO cuenta como output token.

---

## 🚨 Anti-patrones universales (descarta en el examen)

- Parsear texto del asistente para decidir flujo.
- Tope arbitrario de iteraciones.
- Subagentes que se hablan entre sí.
- Self-review (misma instancia genera y revisa).
- "Be conservative" como técnica de precisión.
- Métricas agregadas sin segmentación.
- Métricas autoreportadas como triggers principales.
- Suprimir errores silenciosamente.
- Choose-one entre fuentes conflictivas.
- MCP error genérico "Operation failed".
- Tool genérica (`fetch_url`) cuando se necesita restringida (`load_document`).
- 18+ tools en un agente.
- Batch API para workflows bloqueantes.
- Olvidar `-p` / `--print` en CI.
- Prompt para enforcement de regla de negocio crítica (debe ser hook).
- ~/.mcp.json (NO existe — es ~/.claude.json).
- Asumir herencia de contexto en subagentes.

---

## ✅ Patrones canónicos (busca estas en las respuestas)

- `stop_reason == "end_turn"` como salida del loop.
- Hub-and-spoke con coordinador único.
- `PreToolUse` hook para enforcement crítico.
- `PostToolUse` hook para normalización determinista.
- Tool description detallada (qué, cuándo, formato, retorno, errores).
- Scoping de tools por rol (5-7 por agente).
- `tool_choice` forzado por nombre para extracción estructurada.
- `.claude/rules/` con `paths` para reglas dispersas.
- `context: fork` para skills verbosas.
- `--permission-mode plan` para review-only.
- Instancia independiente para review.
- Few-shot 3-5 con razonamiento.
- Schema con `nullable` + `*_confidence` + self-checks.
- Stratified sampling + calibración por segmento.
- Claim ↔ source mappings con fechas.
- Handoff con payload estructurado (case_id, root_cause, recommended_action).
- "Retract the claim" si no hay quote.

---

## 🎯 Estrategia de examen

1. **Identificar arquetipo** (1-6 de `exam-scenarios.md`).
2. **Descartar opciones con anti-patrones** (lista anti-patrones universales).
3. **Buscar la opción con patrón canónico**.
4. **Verificar con checklist del arquetipo**.
5. Si dudas entre dos: la que es **estructural** (hook, schema, scoping) > la que es **probabilística** (más prompt, modelo más grande).

**¡Mucha suerte!** 🍀
