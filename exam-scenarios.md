# Escenarios recurrentes en el examen CCA-F

El CCA-F basa sus 60 preguntas en **6 arquetipos de escenario**. Estudiar el patrón de cada uno te permite reconocer qué dominio se está evaluando y predecir las trampas habituales del examen.

> Para cada arquetipo: descripción → dominios implicados → señales de la pregunta → trampas típicas → checklist de respuesta.

---

## Arquetipo 1 — Agente de soporte al cliente con escalado

### Descripción
Sistema agéntico que atiende tickets/llamadas: clasifica el caso, consulta sistemas internos (CRM, billing), responde, y en ciertos casos **escala a un humano**.

### Dominios implicados
- **D1.4** Workflows con enforcement (verificación de identidad obligatoria).
- **D1.5** Hooks que bloquean operaciones críticas (ej. reembolso > $500).
- **D5.2** Escalado y resolución de ambigüedad.
- **D5.5** Calibración de confianza.

### Señales de la pregunta
- Aparece "verify customer identity before…".
- Hay límites duros: montos, tipos de cuenta, jurisdicciones.
- El cliente "pide hablar con un humano".
- Política ambigua o silente sobre un caso edge.

### Trampas típicas
- 🔴 Confiar en el **prompt** para enforcement crítico ("the agent should verify identity first"). **Solución correcta:** hook `PreToolUse` que bloquea la tool de reembolso si no se ha llamado antes a `verify_identity`.
- 🔴 Usar **sentiment / confidence autoreportado** como señal de escalado. No son proxies fiables.
- 🔴 Escalar sin pasar contexto estructurado al humano (ID, root cause, monto, acción recomendada).
- 🔴 Investigar primero cuando el cliente **pide explícitamente** un humano.

### Checklist de respuesta correcta
- [ ] Enforcement determinista vía hook, no vía prompt.
- [ ] Triggers de escalado **explícitos**: petición de humano, gap de política, incapacidad de avanzar.
- [ ] Handoff con **payload estructurado** (no prosa).
- [ ] Sentiment ≠ trigger de escalado.

---

## Arquetipo 2 — Pipeline multi-agente de research (coordinator–subagent)

### Descripción
Un coordinador descompone una pregunta de research compleja, delega sub-tareas a subagentes en paralelo, y **agrega/sintetiza** los resultados con atribución.

### Dominios implicados
- **D1.2** Orquestación coordinator–subagent (hub-and-spoke).
- **D1.3** Spawning, paso de contexto, `fork_session`.
- **D1.6** Estrategias de descomposición.
- **D5.3** Propagación de errores en multi-agente.
- **D5.6** Provenance y manejo de incertidumbre.

### Señales de la pregunta
- "Research X across multiple sources".
- Hay datos contradictorios entre fuentes.
- El sistema debe **citar** o atribuir afirmaciones.
- Se menciona descomposición "demasiado estrecha" o "incompleta".

### Trampas típicas
- 🔴 Subagentes que se hablan entre sí (rompe hub-and-spoke).
- 🔴 Asumir que el subagente **hereda el contexto** del coordinador — hay que pasarlo en el prompt.
- 🔴 Descomposición demasiado granular que **deja gaps** sin cubrir.
- 🔴 Suprimir errores: "Search unavailable" sin propagar contexto.
- 🔴 Elegir arbitrariamente entre fuentes conflictivas en lugar de **anotar el conflicto**.
- 🔴 No preservar URLs/excerpts → atribución se pierde en el resumen final.

### Checklist de respuesta correcta
- [ ] Coordinador como **único** punto de comunicación entre subagentes.
- [ ] Prompts a subagentes con contexto explícito.
- [ ] Spawning **en paralelo** (múltiples Task calls en una respuesta).
- [ ] Errores reportados con contexto estructurado (tipo, query, resultados parciales).
- [ ] Mappings **claim ↔ source** preservados.
- [ ] Fechas de publicación incluidas en la síntesis.
- [ ] Render según tipo de contenido (tablas para financiero, prosa para news).

---

## Arquetipo 3 — Claude Code en CI/CD

### Descripción
Pipeline de CI que invoca Claude Code para revisar PRs, generar tests, validar contratos, etc. en modo **no interactivo**.

### Dominios implicados
- **D3.6** Claude Code en CI/CD.
- **D3.1** CLAUDE.md para contexto del proyecto en CI.
- **D4.3** Output estructurado.
- **D4.6** Multi-instancia para review.

### Señales de la pregunta
- "GitHub Actions / GitLab CI runs Claude Code on each PR…".
- "Output must be parseable" / "block the merge if…".
- Se duplican comentarios en re-revisiones.
- Mismo Claude que **generó** el código lo está revisando.

### Trampas típicas
- 🔴 Olvidar `--print` / `-p` → CI se cuelga esperando entrada interactiva.
- 🔴 No pedir output estructurado (`--output-format json` + `--json-schema`).
- 🔴 Re-revisar tras nuevos commits sin pasar los **hallazgos previos** → comentarios duplicados.
- 🔴 Misma instancia genera y revisa → menos detección de issues sutiles. **Usar instancias independientes**.
- 🔴 Olvidar incluir `CLAUDE.md` en el entorno de CI → falta contexto del proyecto.

### Checklist de respuesta correcta
- [ ] `claude -p` con `--output-format json` y schema explícito.
- [ ] CLAUDE.md cargado en CI.
- [ ] Instancia **independiente** para review (no la que generó).
- [ ] Hallazgos previos en el prompt para evitar duplicados.

---

## Arquetipo 4 — Asistente de productividad para developers

### Descripción
Skills, slash commands, hooks, MCP servers configurados para que un equipo de devs use Claude Code de forma productiva y consistente.

### Dominios implicados
- **D3.1** CLAUDE.md jerarquía y modular.
- **D3.2** Skills y slash commands.
- **D3.3** Reglas path-specific.
- **D2.4** MCP server integration.

### Señales de la pregunta
- "The team wants consistent style across X files".
- "User-level vs project-level".
- "Should the skill be invoked automatically or on-demand?"
- Convenciones que solo aplican a ciertos paths (ej. tests).

### Trampas típicas
- 🔴 Poner reglas globales en `CLAUDE.md` cuando aplican sólo a un subdirectorio → contexto inflado.
- 🔴 Confundir `.claude/` (proyecto, versionado) con `~/.claude/` (personal, no versionado).
- 🔴 Usar CLAUDE.md por subdirectorio cuando los archivos están dispersos → `.claude/rules/` con `paths:` es mejor.
- 🔴 No usar `context: fork` para skills con output verboso → contamina sesión.
- 🔴 Skills como "siempre cargado" cuando deberían ser on-demand → CLAUDE.md sería lo apropiado.

### Checklist de respuesta correcta
- [ ] Project-level vs user-level correcto según si se versiona.
- [ ] `.claude/rules/` con `paths` para convenciones path-specific dispersas.
- [ ] Skills: on-demand. CLAUDE.md: siempre.
- [ ] `context: fork` para outputs verbosos.
- [ ] `@import` para modularizar CLAUDE.md.

---

## Arquetipo 5 — Extracción de datos estructurados de documentos

### Descripción
Pipeline que recibe documentos no estructurados (PDFs, emails, contratos) y extrae datos en JSON validado.

### Dominios implicados
- **D4.3** Output estructurado con tool use + schemas.
- **D4.2** Few-shot prompting.
- **D4.4** Validación, retry, self-correction.
- **D4.5** Batch processing.
- **D5.5** Calibración de confianza y human review.

### Señales de la pregunta
- "Extract X, Y, Z from documents".
- "Schema must enforce…".
- "97% accuracy" — alertarte de métricas agregadas.
- Volumen alto, ventana de procesamiento amplia → pensar en batch.
- Documentos con formatos heterogéneos.

### Trampas típicas
- 🔴 Confiar en que **el schema estricto** previene errores semánticos. Sólo previene errores **sintácticos**.
- 🔴 Few-shot mal usado (1 ejemplo, sin razonamiento) → no resuelve ambigüedad.
- 🔴 Retry con feedback cuando la info **no está en la fuente** → bucle infinito.
- 🔴 Batch API para workflows **bloqueantes** (pre-merge check) — incorrecto.
- 🔴 Métrica agregada del 97% que oculta **mal rendimiento por tipo de documento**.
- 🔴 No usar `nullable` en campos que pueden faltar → el modelo **fabrica** valores.

### Checklist de respuesta correcta
- [ ] Tool use con schema bien diseñado: enums + `"unclear"`/`"other"`, nullable, `detail` fields.
- [ ] 2–4 few-shot examples con razonamiento.
- [ ] Self-correction: `calculated_total` vs `stated_total`, `conflict_detected`.
- [ ] Stratified sampling para validar por segmento.
- [ ] Batch API para overnight; síncrono para bloqueante.
- [ ] Routing a humano por baja confianza o conflicto.

---

## Arquetipo 6 — Generación de código + review automatizado

### Descripción
Pipeline de generación de código (feature, refactor, tests) con un pass de review automatizado. A veces el mismo Claude genera y revisa, a veces otro distinto.

### Dominios implicados
- **D4.1** Criterios explícitos.
- **D4.6** Multi-instancia y multi-pass review.
- **D1.6** Descomposición de tareas (per-file + cross-file).
- **D3.4** Plan mode para diseño.

### Señales de la pregunta
- "Self-review missed issues".
- "Comments are too noisy / many false positives".
- "Should the review be per-file or whole-PR?".
- "How do we catch issues across files?".

### Trampas típicas
- 🔴 **Self-review**: misma instancia genera y revisa → retiene su razonamiento, cuestiona menos.
- 🔴 Criterios vagos ("verify comments are accurate") → falsos positivos. Reemplazar por concretos ("flag when comment contradicts code").
- 🔴 Sólo per-file review → pierde issues de integración cross-file.
- 🔴 Sólo whole-PR review → dilución de atención si es grande.
- 🔴 "Be conservative" / "only high-confidence" en el prompt **no** sube precisión.

### Checklist de respuesta correcta
- [ ] Instancia independiente para review.
- [ ] Criterios concretos + ejemplos por nivel de severidad.
- [ ] Multi-pass: **per-file** (detalle) + **cross-file integration** (integración).
- [ ] Plan mode para diseño antes de implementar.
- [ ] Few-shot con falso positivo vs issue real para reducir ruido.

---

## Cómo abordar una pregunta del examen — protocolo de 5 pasos

1. **Identificar arquetipo** (1-6 arriba). Esto activa el dominio relevante y las trampas conocidas.
2. **Detectar señales** de la pregunta (palabras clave, contexto del escenario).
3. **Eliminar opciones con anti-patrones conocidos** (parsing de texto, self-review, prompts en lugar de hooks, métricas agregadas, etc.).
4. **Buscar la opción que aplica el patrón canónico** (hub-and-spoke, structured output, context: fork, etc.).
5. **Verificar con la checklist** del arquetipo: si todos los puntos están cubiertos en la opción, es la correcta.

---

## Banderas rojas universales (siempre mal)

Si una opción incluye CUALQUIERA de esto, casi siempre es **incorrecta**:

- ❌ Parsear texto natural del asistente para decidir flujo.
- ❌ Tope arbitrario de iteraciones (`max_iterations = 5`).
- ❌ Subagentes que se hablan entre sí.
- ❌ Hereda contexto del coordinador automáticamente.
- ❌ Self-review (mismo agent genera y revisa).
- ❌ "Be conservative" / "only high-confidence" como técnica de precisión.
- ❌ Confiar en sentiment/confidence autoreportados.
- ❌ Métricas agregadas sin segmentación.
- ❌ Suprimir errores silenciosamente.
- ❌ Resumen vago que pierde números/fechas/IDs.
- ❌ Choose-one entre fuentes conflictivas en lugar de anotar.
- ❌ MCP error con mensaje genérico "Operation failed".
- ❌ Tool genérica (`fetch_url`) cuando se necesita una restringida (`load_document`).
- ❌ Demasiadas tools en un agente (>10) sin scoping.
- ❌ Batch API para workflows bloqueantes (pre-merge).
- ❌ Olvidar `--print` en CI.
- ❌ Usar prompt para enforcement de regla de negocio crítica.
