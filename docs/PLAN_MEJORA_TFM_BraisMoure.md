# Plan de Mejora TFM — OpsGuard-AI
## Máster en Desarrollo con IA · BIG school

---

> **Evaluador:** Brais Moure (@mouredev)
> **Ingeniero de software freelance · GitHub Star · Microsoft MVP**
> **Proyecto evaluado:** OpsGuard-AI — Context-Aware Security Gate for DevOps Pipelines
> **Autor:** Óscar Sánchez Pérez
> **Fecha:** Marzo 2026
> **Basado en evaluación:** Febrero 2026 (commit `65457c3`)

---

## Por qué este documento existe

La evaluación original cerró con un 9.15/10 y un veredicto claro: sobresaliente. Pero una nota no es un producto terminado. Este documento no es una crítica —es una hoja de ruta. Lo que sigue es la lista ordenada de lo que yo, si fuera Óscar, atacaría primero. Ordenado de mayor a menor margen de mejora, porque así es como se priorizan las cosas en un backlog real.

Sin rodeos.

---

## Tabla de Mejoras — Orden Descendente de Prioridad

| # | Criterio | Nota Actual | Nota Máxima | Gap | Prioridad |
|---|----------|:-----------:|:-----------:|:---:|:---------:|
| 1 | Testing y Calidad | 8.0 / 10 | 10.0 / 10 | **−2.0** | ALTA |
| 2 | Implementación del Código | 8.5 / 10 | 10.0 / 10 | **−1.5** | ALTA |
| 3 | Arquitectura y Diseño Técnico | 9.0 / 10 | 10.0 / 10 | **−1.0** | MEDIA |
| 4 | Documentación | 9.0 / 10 | 10.0 / 10 | **−1.0** | MEDIA |
| 5 | Originalidad e Impacto Potencial | 9.0 / 10 | 10.0 / 10 | **−1.0** | MEDIA |
| 6 | Concepto y Propuesta de Valor | 9.5 / 10 | 10.0 / 10 | **−0.5** | BAJA |
| 7 | Integración de Inteligencia Artificial | 9.5 / 10 | 10.0 / 10 | **−0.5** | BAJA |
| 8 | CI/CD y DevOps | 9.5 / 10 | 10.0 / 10 | **−0.5** | BAJA |
| 9 | Buenas Prácticas de Desarrollo | 9.5 / 10 | 10.0 / 10 | **−0.5** | BAJA |
| 10 | Alineación con el Plan de Estudios | 10.0 / 10 | 10.0 / 10 | **0.0** | — |

---

## Detalle de Mejoras por Criterio

---

### 1. Testing y Calidad — Gap: −2.0 pts (8.0 → 10.0)

**El problema principal:** El motor de Gate 2 —la parte más crítica e innovadora del sistema— no tiene ni un solo test unitario. En el módulo de **Testing** (Alan Buscaglia) y en **Métricas, coverage y complejidad** (Xavier Portilla / Daniela Maissi) el estándar que enseñamos es claro: política mínima del 80% de cobertura como gate en CI. Aquí esa política está mencionada en el CHANGELOG pero no está ejecutada.

**Qué mejoraría, en este orden:**

1. **Tests unitarios para `ai.py` con mocks de OpenRouter.** No hace falta llamar a la API real para testear el comportamiento del motor. Con `unittest.mock` o `pytest-mock` se pueden simular respuestas válidas, JSON malformado, timeouts, respuestas de tipo lista inesperada y el comportamiento fail-closed. Son exactamente los casos que Playwright y @testing-library nos enseñan a cubrir primero: los caminos de error, no solo el happy path.

2. **Gate de cobertura en CI.** Añadir `--cov-fail-under=80` al job de `test` en el workflow. Si el código baja del 80%, el pipeline falla. Esto convierte la cobertura de una aspiración a una restricción real. Una línea en el YAML. Sin excusas.

3. **Test de integración end-to-end.** Un test que tome un diff fixture del "Shooting Range", ejecute `main.py` completo (Gate 1 + Gate 2 mockeado) y verifique el exit code y la salida de consola. Ese sería el test más valioso de toda la suite porque verifica el contrato público de la herramienta, no sus internos.

**Módulos del máster que respaldan esta mejora:**
- Testing (Alan Buscaglia): TDD, pruebas unitarias, de integración y E2E
- Métricas, coverage y complejidad (Xavier Portilla): política mínima del 80%
- Calidad → Métricas, coverage y complejidad (Daniela Maissi): cobertura como gate en CI

---

### 2. Implementación del Código — Gap: −1.5 pts (8.5 → 10.0)

**El problema principal:** Hay tres decisiones de implementación que quedaron a medias y que, juntas, suman el gap de 1.5 puntos.

**Qué mejoraría, en este orden:**

1. **Externalizar el `SYSTEM_PROMPT` al directorio `prompts/`.** El directorio existe. La intención era esa. El prompt está hardcodeado en `ai.py`. Esto no es correcto: un prompt es un artefacto de configuración, no de código. Moverlo a `prompts/system_prompt.txt` (o `.md`) y cargarlo con `Path(__file__).parent.parent / "prompts" / "system_prompt.txt"` permite versionar el prompt, revisarlo en un PR como texto limpio, y actualizarlo sin tocar Python. Esto conecta directamente con el módulo de **Reglas, bases de conocimiento y documentación** (Kiko Palomares): la documentación y los datos estructurados como motor de la IA.

2. **Documentar el truncado de diff como deuda técnica explícita.** `MAX_DIFF_CHARS = 30_000` descarta el final del diff sin estrategia. En una `v1.0`, vale. Pero debería vivir en un comentario `# TODO(debt): ver ADR-0005` o directamente en un ADR-0005 que explique el problema (diffs grandes), la solución actual (truncado simple), y las alternativas futuras (priorizar hunks por tamaño, chunking semántico). El módulo de **Code Smells, refactor y deuda** (Alan Buscaglia) es explícito en esto: documenta la deuda técnica con ADRs.

3. **Eliminar la dependencia fantasma y cualquier resto de configuración heredada sin revisión.** Pequeño pero significativo: las dependencias se añaden cuando se usan y se eliminan cuando dejan de usarse. Un `poetry show --unused` periódico en el CI como paso de auditoría lo previene. El módulo de **Buenas prácticas** lo dice sin ambigüedad.

**Módulos del máster que respaldan esta mejora:**
- Reglas, bases de conocimiento y documentación (Kiko Palomares)
- Code Smells, refactor y deuda (Alan Buscaglia)
- Buenas prácticas (Brais Moure)

---

### 3. Arquitectura y Diseño Técnico — Gap: −1.0 pts (9.0 → 10.0)

**El problema principal:** La decisión de `fail-closed` (risk_score: 10 ante cualquier fallo de IA) es la correcta en un contexto de seguridad, pero no está documentada como decisión consciente. Y las decisiones no documentadas son deuda técnica arquitectónica.

**Qué mejoraría:**

1. **ADR-0004: Política de fail-closed en Gate 2.** Un ADR es exactamente el artefacto para documentar trade-offs. El contexto es claro: si el motor de IA falla (timeout, error de red, respuesta inesperada), ¿qué hacemos? Las opciones son: fail-open (dejar pasar, priorizar disponibilidad) vs fail-closed (bloquear, priorizar seguridad). La decisión tomada es fail-closed. Las consecuencias positivas: nunca dejamos pasar código potencialmente malicioso por un error técnico. Las consecuencias negativas: un corte de OpenRouter puede bloquear todos los PRs del equipo. Eso merece un ADR, no un comentario en el código.

2. **Documentar el contrato de entrada/salida de cada módulo.** `ingest.py`, `security.py` y `ai.py` tienen contratos implícitos muy claros. Hacerlos explícitos en los docstrings con tipos de retorno y excepciones esperadas completa la arquitectura por capas que ya está bien ejecutada.

**Módulos del máster que respaldan esta mejora:**
- Análisis de Requisitos (Martín Cristóbal): documentar decisiones
- Buenas Prácticas y Principios de Diseño (Martín Cristóbal)
- Code Smells, refactor y deuda (Alan Buscaglia): ADRs como documentación de decisiones clave

---

### 4. Documentación — Gap: −1.0 pts (9.0 → 10.0)

**El problema principal:** El sistema de prompts —la inteligencia central del Gate 2— es una caja negra en el repositorio público. El directorio `prompts/` está vacío o ausente. Eso es una laguna importante para cualquier colaborador externo.

**Qué mejoraría:**

1. **Documentar la evolución del prompt en `prompts/CHANGELOG_PROMPTS.md`.** ¿Cómo era el primer prompt? ¿Qué falsos positivos generó? ¿Qué reglas se añadieron y por qué? Esa historia de iteración es uno de los artefactos más valiosos de un proyecto que usa LLMs. El módulo de **Documentación con IA + Mermaid** (Alan Buscaglia y Xavier Portilla) es explícito: la documentación evoluciona junto al código.

2. **Añadir un `CONTRIBUTING.md`** que explique cómo añadir nuevos patrones de detección al Gate 1, cómo actualizar el prompt del Gate 2, y cómo ejecutar la suite de tests localmente. Si el proyecto va al Marketplace (que debería), este archivo es la diferencia entre un proyecto que la comunidad puede contribuir y uno que no.

3. **Diagrama de estados del pipeline en el README.** El flowchart y el sequence diagram están bien, pero falta un diagrama que muestre los posibles estados de salida del sistema: `BLOCKED_GATE1`, `BLOCKED_GATE2`, `APPROVED`, `SKIPPED`, `ERROR_FAIL_CLOSED`. Mermaid tiene soporte para stateDiagram-v2. Ese diagrama resuelve en 10 líneas la pregunta más frecuente de cualquier nuevo usuario: "¿qué puede pasar cuando ejecuto esto?".

**Módulos del máster que respaldan esta mejora:**
- Documentación con IA + Mermaid (Alan Buscaglia / Xavier Portilla)
- Reglas, bases de conocimiento y documentación (Kiko Palomares / Alan Buscaglia)

---

### 5. Originalidad e Impacto Potencial — Gap: −1.0 pts (9.0 → 10.0)

**El problema principal:** El proyecto tiene recorrido de producto real pero se quedó a un paso de cruzar la línea de TFM a herramienta open source utilizable por cualquier equipo.

**Qué mejoraría:**

1. **Publicar la GitHub Action en el Marketplace.** El `action.yml` existe o puede existir en 30 minutos. Publicarla en el Marketplace con una descripción clara, categoría "Security" y un ejemplo de uso en el README convierte el proyecto en algo que cualquier equipo puede añadir a su pipeline con 4 líneas de YAML. Esa es la diferencia entre un TFM excelente y un producto open source.

2. **Comparativa empírica frente a herramientas existentes.** Semgrep, Gitleaks, Trivy —son los escáneres más usados. ¿En qué casos OpsGuard detecta algo que ellos no? ¿En qué casos ellos son mejores? Una tabla de comparativa con 5-10 casos concretos (incluyendo el typosquatting como ejemplo estelar de OpsGuard ganando) convierte la propuesta de valor de narrativa a evidencia. Eso es lo que diferencia un README de producto de un README académico.

3. **Badge dinámico de estado del security-scan en el README.** El proyecto se autoanaliza. Que el README muestre ese badge en tiempo real es la demostración más directa de que la herramienta funciona y está activa.

**Módulos del máster que respaldan esta mejora:**
- Ciclo de vida del desarrollo de Software (Martín Cristóbal): de idea a despliegue
- Proyecto Final de Máster: carta de presentación profesional

---

### 6. Concepto y Propuesta de Valor — Gap: −0.5 pts (9.5 → 10.0)

**El problema principal:** La narrativa de producto es potente pero le falta el contexto competitivo.

**Qué mejoraría:**

Una sección en el README o en la documentación titulada **"¿Por qué OpsGuard y no X?"** con una tabla comparativa frente a Gitleaks, Semgrep y Trivy. No para atacar a esas herramientas —son excelentes en lo suyo— sino para delimitar el nicho específico donde OpsGuard gana: el análisis semántico de lógica compleja que los escáneres estáticos ignoran. El typosquatting es el ejemplo perfecto. Esa comparativa es lo que convierte una buena idea en una propuesta de valor defendible ante cualquier CTO.

---

### 7. Integración de Inteligencia Artificial — Gap: −0.5 pts (9.5 → 10.0)

**El problema principal:** La justificación del modelo por defecto es económica pero no empírica.

**Qué mejoraría:**

Un benchmark documentado en `docs/MODEL_BENCHMARK.md` con resultados reales comparando `google/gemini-2.0-flash-001`, `anthropic/claude-haiku` y `openai/gpt-4o-mini` sobre el mismo conjunto de fixtures del Shooting Range. Métricas: precisión de detección, coste por análisis, latencia media. El ADR-0003 menciona este objetivo. Materializarlo en datos reales cierra el loop y convierte una decisión de ingeniería en una decisión basada en evidencia. Eso es exactamente lo que enseñamos en **Integración de APIs y plataformas IA populares** (Kiko Palomares): saber cuándo usar cada una y cómo combinarlas.

---

### 8. CI/CD y DevOps — Gap: −0.5 pts (9.5 → 10.0)

**El problema principal:** El pipeline es excelente pero le falta el paso final de distribución.

**Qué mejoraría:**

1. **Job de `release`** que automatice el versionado semántico y publique la GitHub Action como artefacto cuando se hace merge a `main` con una etiqueta de versión. Con `actions/create-release` y el CHANGELOG existente, esto es trabajo de una tarde.

2. **Notificación en caso de bloqueo.** Un step opcional de notificación (Slack webhook, GitHub Issue automático) cuando `security-scan` bloquea un PR en producción. En el módulo de **DevOps y CI/CD** (Xavier Portilla) hablamos de pipelines que no solo detectan —también informan.

---

### 9. Buenas Prácticas de Desarrollo — Gap: −0.5 pts (9.5 → 10.0)

**El problema principal:** Hay un único punto menor: la dependencia fantasma heredada sin revisión.

**Qué mejoraría:**

Añadir `poetry check` y `pip-audit` (o `safety check`) como pasos del job de `lint`. El primero valida que `pyproject.toml` es coherente. El segundo detecta dependencias con vulnerabilidades conocidas. Ambos son ciudadanos naturales de un pipeline de seguridad. Para un proyecto que se dedica a detectar vulnerabilidades en código ajeno, no tenerlos en el suyo propio es una contradicción menor pero visible.

---

### 10. Alineación con el Plan de Estudios — Gap: 0.0 pts (10.0 → 10.0)

**No hay nada que mejorar aquí.** Este criterio está en el máximo y, en mi opinión, es el criterio que más importa desde el punto de vista académico. Cada módulo del máster tiene representación directa en el proyecto:

| Módulo | Evidencia | Estado |
|--------|-----------|--------|
| Ecosistema del desarrollo moderno | 48+ PRs, Conventional Commits, GitHub Actions | Perfecto |
| Buenas prácticas | Black, Pytest, Quality Audit Sprint | Perfecto |
| Paradigmas de Programación | Python idiomático, excepciones tipadas | Perfecto |
| Ingeniería de Software | ADRs, versiones semánticas | Perfecto |
| Arquitectura de Software | Two-Gate System, módulos desacoplados | Perfecto |
| Fundamentos de la IA / IA generativa | Gate 2 con Gemini Flash, LLM con criterio | Perfecto |
| Prompt engineering para developers | CRITICAL CONTEXTUAL RULES, iteración sobre resultados | Perfecto |
| Herramientas (Claude Code, Copilot, CLI) | IA en el proceso, no solo en el producto | Perfecto |
| DevOps y CI/CD | 3 jobs, matrix Python, caché | Perfecto |
| Calidad / Testing | Pytest, FinOps telemetry | Sólido |
| Seguridad / OWASP Top 10 | Detección de secretos, SQL injection, supply chain | Perfecto |
| Documentación con IA + Mermaid | Flowchart + sequence diagram en README | Perfecto |

---

## Resumen Ejecutivo de Mejoras

| # | Criterio | Gap | Acción Principal | Esfuerzo Estimado |
|---|----------|:---:|------------------|:-----------------:|
| 1 | Testing y Calidad | **−2.0** | Tests para `ai.py` + gate cobertura 80% en CI + test E2E | 1-2 días |
| 2 | Implementación del Código | **−1.5** | Externalizar prompt + ADR-0005 deuda truncado + auditoría deps | 1 día |
| 3 | Arquitectura y Diseño Técnico | **−1.0** | ADR-0004 fail-closed + contratos explícitos módulos | Medio día |
| 4 | Documentación | **−1.0** | CHANGELOG prompts + CONTRIBUTING.md + diagrama estados | 1 día |
| 5 | Originalidad e Impacto Potencial | **−1.0** | Publicar en GitHub Marketplace + comparativa vs Gitleaks/Semgrep | 1-2 días |
| 6 | Concepto y Propuesta de Valor | **−0.5** | Tabla comparativa competidores en README | Medio día |
| 7 | Integración de IA | **−0.5** | Benchmark documentado de modelos con datos reales | 1 día |
| 8 | CI/CD y DevOps | **−0.5** | Job de release automatizado + notificación en bloqueo | Medio día |
| 9 | Buenas Prácticas | **−0.5** | `poetry check` + `pip-audit` en job lint | 2 horas |
| 10 | Alineación con Plan de Estudios | **0.0** | Nada. No toques lo que funciona. | — |

**Total gap acumulado: −8.0 puntos distribuidos en ~6-8 días de trabajo.**

Si Óscar implementa las 4 primeras mejoras (Testing, Código, Arquitectura, Documentación), la nota subiría a un **9.8/10** sin cambiar ni una línea del producto. Solo documentación, tests y decisiones explícitas. Eso dice mucho del nivel del proyecto base: los puntos que faltan no son de código, son de completitud profesional.

---

## Conclusión

Este plan de mejora no existe porque el TFM sea insuficiente. Existe porque este proyecto merece convertirse en algo más que un TFM. Los gaps son pequeños, concretos y accionables. No requieren reescribir nada —requieren completar lo que ya está iniciado.

El CHANGELOG ya es bueno. El siguiente paso es que el proyecto tenga usuarios reales que lo estén usando en producción. Para eso, necesita el Marketplace, el benchmark de modelos y la suite de tests completa.

Ese es el camino de un desarrollador que acaba de demostrar que tiene criterio de ingeniería: no parar cuando el tutor dice "sobresaliente".

---

*Brais Moure (@mouredev)*
*Ingeniero de Software · GitHub Star · Microsoft MVP*
*Cofundador de Pilbeo · Tutor del Máster en Desarrollo con IA, BIG school*

---

> *Este documento es un plan de mejora complementario a la evaluación original (Febrero 2026).*
> *Generado sobre el commit `65457c3` (rama `main`) · Marzo 2026*
