# Claude Certified Architect — Foundations (CCA-F)

Apuntes detallados y estructurados para preparar la certificación **Claude Certified Architect – Foundations (CCA-F)** de Anthropic.

> **Formato del examen:** 60 preguntas de opción múltiple basadas en escenarios · 120 minutos · puntaje mínimo **720/1000** · USD $99 · inglés.
>
> **Tecnologías base:** Claude Code · Claude Agent SDK · Claude API · Model Context Protocol (MCP).

---

## Tabla de contenidos

| Dominio                                       | Peso | Carpeta                                                  |
| --------------------------------------------- | ---- | -------------------------------------------------------- |
| **1. Agentic Architecture & Orchestration**   | 27%  | [`01-agentic-architecture/`](./01-agentic-architecture/) |
| **2. Tool Design & MCP Integration**          | 18%  | [`02-tool-design-mcp/`](./02-tool-design-mcp/)           |
| **3. Claude Code Configuration & Workflows**  | 20%  | [`03-claude-code-config/`](./03-claude-code-config/)     |
| **4. Prompt Engineering & Structured Output** | 20%  | [`04-prompt-engineering/`](./04-prompt-engineering/)     |
| **5. Context Management & Reliability**       | ~15% | [`05-context-management/`](./05-context-management/)     |
| Glosario técnico                              | —    | [`glossary.md`](./glossary.md)                           |
| Escenarios de examen                          | —    | [`exam-scenarios.md`](./exam-scenarios.md)               |
| Cheatsheet final                              | —    | [`cheatsheet.md`](./cheatsheet.md)                       |

---

## Dominio 1 — Agentic Architecture & Orchestration (27%)

1. [Diseñar e implementar bucles agénticos (agentic loops)](./01-agentic-architecture/1.1-agentic-loops.md)
2. [Orquestar sistemas multi-agente (coordinator–subagent)](./01-agentic-architecture/1.2-multi-agent-orchestration.md)
3. [Invocación de subagentes, paso de contexto y spawning](./01-agentic-architecture/1.3-subagent-invocation.md)
4. [Workflows multi-paso con enforcement y handoff](./01-agentic-architecture/1.4-multistep-enforcement-handoff.md)
5. [Hooks del Agent SDK: intercepción y normalización](./01-agentic-architecture/1.5-agent-sdk-hooks.md)
6. [Estrategias de descomposición de tareas](./01-agentic-architecture/1.6-task-decomposition.md)
7. [Gestión de sesiones: estado, reanudación y forking](./01-agentic-architecture/1.7-session-management.md)

## Dominio 2 — Tool Design & MCP Integration (18%)

1. [Diseño de interfaces de tools con descripciones claras](./02-tool-design-mcp/2.1-tool-interface-design.md)
2. [Respuestas estructuradas de error en tools MCP](./02-tool-design-mcp/2.2-mcp-structured-errors.md)
3. [Distribución de tools entre agentes y `tool_choice`](./02-tool-design-mcp/2.3-tool-distribution-and-tool-choice.md)
4. [Integración de servidores MCP en Claude Code](./02-tool-design-mcp/2.4-mcp-server-integration.md)
5. [Tools built-in: Read, Write, Edit, Bash, Grep, Glob](./02-tool-design-mcp/2.5-builtin-tools.md)

## Dominio 3 — Claude Code Configuration & Workflows (20%)

1. [Archivos CLAUDE.md: jerarquía, scope y organización modular](./03-claude-code-config/3.1-claude-md-hierarchy.md)
2. [Slash commands y Skills personalizados](./03-claude-code-config/3.2-slash-commands-and-skills.md)
3. [Reglas path-specific para carga condicional](./03-claude-code-config/3.3-path-specific-rules.md)
4. [Plan mode vs ejecución directa](./03-claude-code-config/3.4-plan-mode.md)
5. [Refinamiento iterativo](./03-claude-code-config/3.5-iterative-refinement.md)
6. [Claude Code en pipelines CI/CD](./03-claude-code-config/3.6-claude-code-cicd.md)

## Dominio 4 — Prompt Engineering & Structured Output (20%)

1. [Prompts con criterios explícitos](./04-prompt-engineering/4.1-explicit-criteria.md)
2. [Few-shot prompting](./04-prompt-engineering/4.2-few-shot-prompting.md)
3. [Output estructurado con tool use y JSON schemas](./04-prompt-engineering/4.3-structured-output.md)
4. [Validación, retry y feedback loops](./04-prompt-engineering/4.4-validation-retry.md)
5. [Batch processing eficiente](./04-prompt-engineering/4.5-batch-processing.md)
6. [Arquitecturas de review multi-instancia y multi-pass](./04-prompt-engineering/4.6-multi-instance-review.md)

## Dominio 5 — Context Management & Reliability (~15%)

1. [Gestión de contexto en interacciones largas](./05-context-management/5.1-long-context-management.md)
2. [Escalado y resolución de ambigüedad](./05-context-management/5.2-escalation-and-ambiguity.md)
3. [Propagación de errores en sistemas multi-agente](./05-context-management/5.3-error-propagation.md)
4. [Contexto en exploración de codebases grandes](./05-context-management/5.4-large-codebase-context.md)
5. [Human review y calibración de confianza](./05-context-management/5.5-human-review-confidence.md)
6. [Provenance y manejo de incertidumbre en síntesis multi-fuente](./05-context-management/5.6-provenance-and-uncertainty.md)

---

## Cómo estudiar con este repositorio

1. **Lee en orden** los dominios — están secuenciados por dependencia conceptual: arquitectura → tools → configuración → prompting → contexto.
2. **Cada `*.md` sigue la misma estructura:**
   - 🎯 **Concepto clave** — la idea central en 1-2 párrafos.
   - 📚 **Profundización** — explicación técnica con código.
   - ⚠️ **Anti-patrones** — qué NO hacer y por qué.
   - ✅ **Patrón correcto** — la solución idiomática.
   - 🧪 **Escenario de examen** — pregunta tipo CCA-F + respuesta razonada.
   - 🔗 **Referencias oficiales** — links a docs.anthropic.com / modelcontextprotocol.io.
3. **Repasa el [glosario](./glossary.md)** antes del examen — la mayoría de preguntas usan terminología exacta de la documentación.
4. **Practica con [escenarios de examen](./exam-scenarios.md)** — los 6 arquetipos recurrentes.
5. **El [cheatsheet](./cheatsheet.md)** es para repaso final 24h antes del examen.

---

## Convenciones de los apuntes

- **Negrita** para conceptos críticos del examen.
- `código inline` para nombres de tool, flag, archivo, parámetro.
- Bloques de código con lenguaje explícito (`python`, `json`, `bash`, `markdown`).
- 🔴 = anti-patrón / 🟢 = patrón correcto / 🟡 = trade-off / matiz.
- Citas literales de docs oficiales van con `>` y URL al final.

---

## Recursos oficiales

- **Documentación Claude:** <https://docs.claude.com> · <https://docs.anthropic.com>
- **Claude Agent SDK:** <https://docs.claude.com/en/api/agent-sdk/overview>
- **Claude Code:** <https://docs.claude.com/en/docs/claude-code/overview>
- **Model Context Protocol:** <https://modelcontextprotocol.io>
- **Anthropic Cookbook:** <https://github.com/anthropics/anthropic-cookbook>
- **MCP servers oficiales:** <https://github.com/modelcontextprotocol/servers>

---

## Estado de los apuntes

- [x] Estructura inicial
- [x] Dominio 1 — Agentic Architecture
- [x] Dominio 2 — Tool Design & MCP
- [x] Dominio 3 — Claude Code Configuration
- [x] Dominio 4 — Prompt Engineering
- [x] Dominio 5 — Context Management
- [x] Glosario técnico
- [x] Escenarios de examen
- [x] Cheatsheet
