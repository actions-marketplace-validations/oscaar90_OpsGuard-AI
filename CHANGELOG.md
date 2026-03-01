# CHANGELOG

Registro de cambios del proyecto **OpsGuard-AI** siguiendo el formato [Keep a Changelog](https://keepachangelog.com/es-ES/1.1.0/) y la especificación [Semantic Versioning](https://semver.org/).

Cada entrada está vinculada a su Pull Request en GitHub para trazabilidad completa del proceso de ingeniería.

---

## [1.0.1] - 2026-03-01 · Polish & Hardening

### Added

- **[tag v1.0.1]** `.gitignore` reforzado - Añade reglas para prevenir la subida accidental de artefactos internos: informes de evaluación y documentos de trabajo (`*Brais*.md`, `*oscar*.md`), PDFs (`*.pdf`), artefactos de cobertura (`.coverage`, `htmlcov/`) y ficheros temporales de editor (`*.kate-swp`, `*.swp`).

### Changed

- **[tag v1.0.1]** Gráficas comparativas de modelos en `docs/adr/0003-telemetria-y-finops.md` - Dos diagramas `xychart-beta` de Mermaid que se renderizan nativamente en GitHub: coste mensual estimado (1 000 PRs/mes) y latencia media por análisis. Hacen visualmente obvia la ventaja de Gemini Flash 2.0 frente a Claude Haiku 4.5 (9.5x más barato) y GPT-4o-mini (mejor latencia y menor coste).
- **[tag v1.0.1]** Sustitución de em dash (`-`) por guion normal (`-`) en 16 ficheros del proyecto - Normalización tipográfica en ficheros Markdown, Python y YAML.

---

## [1.0.0] - 2026-03-01 · Prompt Engineering Documentation Sprint

> Ciclo de mejora de documentación iniciado tras el análisis de brechas del informe de evaluación TFM. El criterio **Documentación** obtuvo un 9.0/10 (gap de −1.0 pt) con una observación concreta: "La documentación del directorio `prompts/` brilla por su ausencia. El sistema de prompts es una parte crítica de la inteligencia del sistema." Este sprint cierra esa brecha y completa el ciclo de mejora de calidad del TFM.

### Added

- **[PR #55]** `prompts/README.md` - Documentación completa del directorio de prompts. Cubre dos tipos de artefactos: el prompt de producción de Gate 2 (`system_prompt.txt`) y los prompts de desarrollo (01–06). Para `system_prompt.txt` documenta: anatomía del prompt (ROLE → CONTEXT → TASK → RULES → OUTPUT FORMAT), el origen de cada CRITICAL CONTEXTUAL RULE (falsos positivos detectados en dog-fooding), las decisiones de prompt engineering con alternativas descartadas (idioma, temperatura, rol, schema, reglas contextuales), y la evolución del prompt a través de 3 versiones. Para los prompts de desarrollo documenta el inventario completo (qué módulo construyó cada prompt, qué rol se asignó al LLM) y la relación entre prompts y ADRs como artefactos complementarios del mismo proceso de toma de decisiones. Materializa el módulo "Flujo de Desarrollo con IA" del plan de estudios.
  - Fichero: `prompts/README.md`
  - Rama: `docs/prompts-documentation` → `main`

---

## [0.9.0] - 2026-03-01 · GitHub Action Marketplace Sprint

> Ciclo de mejora de originalidad e impacto iniciado tras el análisis de brechas del informe de evaluación TFM. Los criterios **Originalidad e Impacto Potencial** (9.0/10, gap −1.0) y **CI/CD y DevOps** (9.5/10, gap −0.5) señalaban la misma deuda: OpsGuard existía como proyecto interno pero no como producto reutilizable. Este sprint completa la última milla: publica OpsGuard como GitHub Action disponible en el Marketplace y añade el workflow de release automático que convierte cada tag semántico en un artefacto versionado.

### Added

- **[PR #54]** `action.yml` - Definición de GitHub Action compuesta y publicable en el Marketplace. Reemplaza el stub anterior (que referenciaba un Dockerfile inexistente) con una composite action completamente funcional. Expone cuatro inputs configurables (`openrouter-api-key`, `risk-threshold`, `model`, `telemetry-mode`), gestiona su propio entorno Python + Poetry en `${{ github.action_path }}` con `virtualenvs.in-project true`, e invoca `opsguard scan` desde el workspace del consumidor. Con este fichero cualquier equipo puede añadir OpsGuard a su pipeline con `uses: oscaar90/OpsGuard-AI@v1`.
  - Fichero: `action.yml`
  - Rama: `feat/github-action-marketplace` → `main`

- **[PR #54]** `.github/workflows/release.yml` - Workflow de publicación automática de releases. Se activa en push de tags `v*.*.*` y crea una GitHub Release con notas auto-generadas a partir de los PRs mergeados desde el tag anterior. Establece el ciclo de vida de versiones semánticas que permite a los consumidores anclar a una versión estable (`@v1`, `@v0.9.0`) en lugar de depender de `@main`. Cubre la mejora de CI/CD que Brais señalaba: artefacto versionado y publicable.
  - Fichero: `.github/workflows/release.yml`
  - Rama: `feat/github-action-marketplace` → `main`

- **[PR #54]** Sección "Integra OpsGuard en tu pipeline (5 minutos)" en `README.md` - Documentación de integración orientada al consumidor externo. Guía paso a paso (añadir secreto → crear workflow) con snippet YAML listo para copiar y pegar. Tabla de inputs configurables. Convierte el README de documentación interna del proyecto en documentación de producto open source.
  - Fichero: `README.md`
  - Rama: `feat/github-action-marketplace` → `main`

---

## [0.8.0] - 2026-03-01 · AI Model Benchmark Sprint

> Ciclo de mejora de integración de IA iniciado tras el análisis de brechas del informe de evaluación TFM. El criterio **Integración de Inteligencia Artificial** obtuvo un 9.5/10 (gap de −0.5 pts) porque ADR-0003 declaraba como objetivo una comparativa empírica entre modelos que nunca se materializó en datos reales. Este sprint cierra esa deuda: se ejecuta el benchmark sobre los 4 fixtures de Gate 2 y se documentan los resultados con métricas de detección, latencia y coste.

### Added

- **[PR #53]** Resultados del benchmark de modelos en `docs/adr/0003-telemetria-y-finops.md` - Se añade la sección "Resultados del Benchmark de Modelos" que materializa la comparativa empírica prometida en ADR-0003. El benchmark ejecuta los 4 fixtures del Shooting Range (`legacy_login.py`, `auth_middleware.py`, `config.php`, `supply_chain_attack.py`) contra tres modelos disponibles via OpenRouter: `google/gemini-2.0-flash-001`, `anthropic/claude-haiku-4-5` y `openai/gpt-4o-mini`. Los resultados cubren: veredicto, risk score, latencia media (mediana de 3 ejecuciones), tokens consumidos y coste por llamada. La conclusión consolida `gemini-2.0-flash-001` como modelo por defecto con datos cuantitativos: detección 4/4, latencia media 2 848 ms y coste estimado de $0.24/mes a 1 000 PRs/mes - 9× más barato que Claude Haiku con paridad de detección.
  - Fichero: `docs/adr/0003-telemetria-y-finops.md`
  - Rama: `feat/model-benchmark` → `main`

---

## [0.7.0] - 2026-03-01 · Competitive Positioning Sprint

> Ciclo de mejora de propuesta de valor iniciado tras el análisis de brechas del informe de evaluación TFM. El criterio **Concepto y Propuesta de Valor** obtuvo un 9.5/10 (gap de −0.5 pts) porque la narrativa de producto no contextualizaba OpsGuard frente al ecosistema de herramientas existentes (Semgrep, Gitleaks, Trivy). Este sprint añade la comparativa que Brais pedía para redondear la propuesta diferencial.

### Added

- **[PR #52]** Sección comparativa "¿Por qué OpsGuard?" en `README.md` - Tabla de capacidades frente a Gitleaks, Semgrep y Trivy, con análisis del nicho específico de OpsGuard: detección semántica de lógica compleja (SQL Injection contextual, backdoors lógicos, typosquatting de dominios) que los escáneres estáticos no pueden cubrir. Incluye el patrón de uso recomendado (OpsGuard + Trivy como herramientas complementarias, no competidoras) y la diferenciación clave de privacidad (Gate 1 garantiza que los secretos nunca llegan a la API externa, ADR-0001).
  - Fichero: `README.md`
  - Rama: `docs/competitive-analysis` → `main`

---

## [0.6.0] - 2026-03-01 · Architecture Documentation Sprint

> Ciclo de mejora de arquitectura iniciado a partir del criterio **Arquitectura y Diseño Técnico** (9.0/10, gap −1.0 pt). La evaluación señalaba un único punto ciego: la política fail-closed de Gate 2 existía en el código pero no estaba documentada como decisión consciente. Este sprint lo cierra con el ADR que faltaba y completa los contratos de módulo de `src/ai.py`.

### Added

- **[PR #51]** ADR-0004: Política Fail-Closed en Gate 2 - Documenta formalmente el trade-off arquitectónico entre **disponibilidad** (fail-open: el pipeline continúa si la IA falla) y **seguridad** (fail-closed: el pipeline se bloquea si la IA falla). La decisión elegida es fail-closed: cualquier excepción en `analyze_diff()` retorna `verdict: BLOCK, risk_score: 10` sin propagar el error. El ADR recoge la justificación de seguridad, las consecuencias negativas (impacto en disponibilidad ante caídas de OpenRouter) y las mitigaciones operacionales recomendadas para producción.
  - Fichero: `docs/adr/0004-fail-closed-policy.md`
  - Rama: `feat/architecture-documentation` → `main`

- **[PR #51]** Docstrings de contrato en `src/ai.py` - `AIEngine` y `analyze_diff()` eran los únicos módulos del sistema sin contratos documentados. Se añaden:
  - Docstring de clase `AIEngine`: describe su responsabilidad (Gate 2 semántico), la dependencia de OpenRouter y la referencia explícita a ADR-0004 para la política de errores.
  - Docstring de `__init__`: documenta la excepción `AIEngineError` que lanza si falta `OPENROUTER_API_KEY`.
  - Docstring de `analyze_diff()`: documenta los args, el dict de retorno completo (keys `verdict`, `risk_score`, `explanation`, `findings`) y el contrato de no-lanzamiento de excepciones con referencia a ADR-0004 y ADR-0005.
  - Fichero: `src/ai.py`
  - Rama: `feat/architecture-documentation` → `main`

---

## [0.5.0] - 2026-03-01 · Code Quality Sprint

> Ciclo de mejora de implementación iniciado tras el análisis de brechas del informe de evaluación TFM. El criterio **Implementación del Código** obtuvo un 8.5/10 (gap de −1.5 pts) por tres decisiones de implementación incompletas: el `SYSTEM_PROMPT` hardcodeado en lugar de externalizado, la falta de documentación formal de la deuda técnica del truncado de diff, y la ausencia de auditoría automática de dependencias en CI.

### Added

- **[PR #50]** `SYSTEM_PROMPT` externalizado a `prompts/system_prompt.txt` - El prompt del sistema de Gate 2 estaba hardcodeado como string en `src/ai.py`. Se mueve al fichero `prompts/system_prompt.txt` y se carga en tiempo de importación con `Path(__file__).parent.parent / "prompts" / "system_prompt.txt"`. A partir de ahora el prompt es un artefacto de configuración versionado independientemente del código Python: puede revisarse, modificarse y auditarse en un PR como texto limpio, sin tocar `src/ai.py`.
  - Ficheros: `src/ai.py`, `prompts/system_prompt.txt`
  - Rama: `feat/code-quality-improvements` → `main`

- **[PR #50]** ADR-0005: Estrategia de Truncado de Diff - Documenta formalmente la decisión de `MAX_DIFF_CHARS = 30_000` que hasta ahora existía como constante sin contexto. El ADR recoge: el problema (coste y calidad en diffs grandes), la solución actual (truncado simple por longitud), las alternativas descartadas (chunking, priorización por hunk, resumen previo) y la deuda técnica reconocida para v2 (priorización por hunk). Es el ADR que faltaba para completar la cobertura documental de las decisiones arquitectónicas del sistema.
  - Fichero: `docs/adr/0005-diff-truncation-strategy.md`
  - Rama: `feat/code-quality-improvements` → `main`

- **[PR #50]** Auditoría automática de dependencias en el job `lint` - Se añaden dos pasos al job de Code Quality del workflow CI/CD: `poetry check` (valida que `pyproject.toml` es coherente con `poetry.lock`) y `poetry run pip-audit --progress-spinner off` (detecta CVEs conocidos en el entorno instalado). Durante la implementación, `pip-audit` detectó dos vulnerabilidades reales que han sido corregidas en este mismo PR:
  - `black 23.12.1` → `24.10.0` (PYSEC-2024-48)
  - `protobuf 5.29.5` → `7.34.0` (CVE-2026-0994, dependencia transitiva vía `openai`/`google-generativeai`; se fija con `protobuf = ">=5.29.6"` en `pyproject.toml`)
  - Ficheros: `.github/workflows/opsguard.yml`, `pyproject.toml`, `poetry.lock`
  - Rama: `feat/code-quality-improvements` → `main`

---

## [0.4.0] - 2026-03-01 · Testing Coverage Sprint

> Ciclo de mejora de calidad iniciado tras el análisis de brechas del informe `docs/PLAN_MEJORA_TFM_BraisMoure.md`. El criterio **Testing y Calidad** obtuvo un 8.0/10 (gap de −2.0 pts) por tres carencias concretas: ausencia de tests para `src/ai.py`, inexistencia de un gate de cobertura mínima en CI y falta de tests de integración end-to-end. Este sprint las cierra.

### Added

- **[PR #49]** Suite de tests unitarios para el motor de Gate 2 (`AIEngine`) - 13 tests en 5 clases que cubren la totalidad de `src/ai.py`, alcanzando **100% de cobertura** en el módulo más crítico del sistema:
  - `TestAIEngineInit` - inicialización correcta y fallo explícito sin `OPENROUTER_API_KEY`.
  - `TestAnalyzeDiffApprove` / `TestAnalyzeDiffBlock` - happy paths con veredictos APPROVE y BLOCK; validación del formato de respuesta y telemetría verbose.
  - `TestAnalyzeDiffFailClosed` - **validación de la política fail-closed**: timeout de API, JSON malformado, respuesta de tipo lista inesperado y dict sin la clave `verdict` retornan siempre `risk_score: 10` sin propagar la excepción. Este grupo de tests es la evidencia formal de que la política de seguridad más importante del sistema funciona como está documentada en ADR-0001.
  - `TestDiffTruncation` - verifica que diffs superiores a `MAX_DIFF_CHARS = 30_000` se truncan antes de enviarse al LLM y que diffs cortos se envían íntegros.
  - `TestTelemetryModes` - modos `silent` y `summary` no rompen el flujo normal.
  - Todos los tests usan `pytest-mock` (sin llamadas reales a la API de OpenRouter).
  - Fichero: `tests/test_ai.py`
  - Rama: `feat/testing-ai-engine-coverage` → `main`

- **[PR #49]** Tests de integración end-to-end del pipeline completo - 4 tests en `TestEndToEnd` que verifican el contrato público de la herramienta (exit codes) usando `typer.testing.CliRunner` con `GitManager` y `AIEngine` mockeados:
  - Gate 1 bloquea credenciales AWS del fixture real → exit code `1`.
  - Gate 2 bloquea SQL injection via mock (Gate 1 lo deja pasar) → exit code `1`.
  - Diff limpio pasa ambos gates → exit code `0`.
  - Pipeline sin `OPENROUTER_API_KEY` termina sin excepción no controlada → exit code `0`.
  - Fichero: `tests/test_e2e.py`
  - Rama: `feat/testing-ai-engine-coverage` → `main`

- **[PR #49]** Gate de cobertura mínima del 80% en CI - El job `test` del workflow pasa de `pytest tests/ -v` a `pytest tests/ -v --cov=src --cov-report=term-missing --cov-fail-under=80`. A partir de ahora, cualquier PR que reduzca la cobertura global de `src/` por debajo del 80% bloquea el merge automáticamente. Resultado en el primer run con el gate activo: **83.64% de cobertura total**, `src/ai.py` al **100%**.
  - Fichero: `.github/workflows/opsguard.yml`
  - Rama: `feat/testing-ai-engine-coverage` → `main`

- **[PR #49]** Dependencias de desarrollo `pytest-cov ^4.1.0` y `pytest-mock ^3.12.0` añadidas a `pyproject.toml` - `pytest-cov` provee el informe de cobertura integrado con pytest; `pytest-mock` expone el fixture `mocker` que permite hacer `mocker.patch.object()` sin strings frágiles de path.
  - Fichero: `pyproject.toml`, `poetry.lock`
  - Rama: `feat/testing-ai-engine-coverage` → `main`

---

## [0.3.0] - 2026-02-21 · Quality Audit Sprint

> Ciclo de mejora continua iniciado tras un análisis técnico exhaustivo del proyecto (`FEEDBACK.md`). Las correcciones cubren dos categorías: **blockers de runtime** que impedían la ejecución en entornos limpios, y **deuda técnica** que comprometía la configurabilidad y el principio de mínimo privilegio.

### Added

- **[PR #44]** Pipeline CI/CD reestructurado en 3 jobs independientes con caché de Poetry y matrix de Python - el workflow de un único job se divide en: `lint` (black --check), `test` (matrix Python 3.11 + 3.12), `security-scan` (opsguard scan, depende de que lint y test pasen). Se añade `actions/cache@v4` para el virtualenv de Poetry, reduciendo el tiempo de instalación en runs sucesivos. Se aplica `black` sobre todo el código fuente por primera vez (10 ficheros reformateados), garantizando que el job de lint no falle en el primer run.
  - Ficheros: `.github/workflows/opsguard.yml`, `src/`, `tests/`
  - Rama: `feat/cicd-pipeline-improvements` → `main`

- **[PR #43]** Sequence diagram de actores en runtime añadido al README - complementa el flowchart de alto nivel con un diagrama de secuencia Mermaid que muestra los actores reales (GitHub Actions, Gate 1, Gate 2, OpenRouter API) y los flujos condicionales de BLOCK/APPROVE con sus notas de contexto (ADR-0001).
  - Fichero: `README.md`
  - Rama: `feat/architecture-improvements` → `main`

### Fixed

- **[PR #43]** `_get_ci_shas()` se llamaba dos veces por ejecución - una en `get_staged_files()` y otra en `_get_ci_diff()`, parseando el fichero JSON del evento de GitHub dos veces. Se introduce `self._shas_cache` en `__init__`: la primera llamada parsea y almacena el resultado; las siguientes lo reutilizan directamente.
  - Fichero: `src/ingest.py`
  - Rama: `feat/architecture-improvements` → `main`

- **[PR #42]** Cadena de error en español en `src/main.py` traducida a inglés - `"Asegúrate de que GitManager tenga 'get_staged_files()'"` violaba ADR-0002. Última instancia de español en artefactos técnicos del proyecto.
  - Fichero: `src/main.py`
  - Rama: `fix/spanish-string-main-py` → `main`
  - Cierra: ADR-0002 (100% cumplimiento)

- **[PR #41]** Eliminación de dependencia fantasma `pydantic` y homogeneización de comentarios a inglés - `pydantic ^2.5.0` estaba declarado como dependencia de producción en `pyproject.toml` pero nunca importado en ningún módulo de `src/`. Se elimina para reducir el grafo de dependencias. Adicionalmente se traducen al inglés los comentarios restantes en español en `.github/workflows/opsguard.yml`, `.opsguardignore` y `src/console_ui.py` (`# --- TABLA FORENSE ---`), cerrando la última deuda detectada en el análisis de calidad de código.
  - Ficheros: `pyproject.toml`, `.github/workflows/opsguard.yml`, `.opsguardignore`, `src/console_ui.py`
  - Rama: `fix/code-quality-remaining` → `main`

- **[PR #24]** `NameError: MAX_DIFF_CHARS is not defined` - La constante que controla el truncado del diff enviado al LLM había sido referenciada pero nunca declarada. Causaba fallo total en runtime. Se define como `MAX_DIFF_CHARS = 30_000` junto a las constantes FinOps existentes.
  - Fichero: `src/ai.py`
  - Rama: `fix/diff-truncation-feedback` → `main`

- **[PR #23]** `ModuleNotFoundError: No module named 'pathspec'` - La librería `pathspec`, usada para procesar `.opsguardignore`, se importaba en `src/main.py` pero no estaba declarada en `pyproject.toml`. La herramienta fallaba en cualquier instalación limpia (`poetry install`), incluyendo CI/CD.
  - Fichero: `pyproject.toml`
  - Rama: `fix/add-pathspec-dependency` → `main`

- **[PR #26]** `OPSGUARD_RISK_THRESHOLD` declarado en el workflow pero ignorado en el código - El threshold de riesgo del Gate 2 estaba hardcodeado como `7` en `src/main.py` mientras el workflow CI/CD ya lo definía como variable de entorno. Se lee ahora vía `os.getenv("OPSGUARD_RISK_THRESHOLD", "7")`, haciendo el sistema configurable sin modificar código fuente.
  - Fichero: `src/main.py`
  - Rama: `fix/risk-threshold-configurable` → `main`

### Security

- **[PR #27]** Eliminado permiso `pull-requests: write` no utilizado del workflow de GitHub Actions - El permiso estaba declarado para una funcionalidad futura (comentarios automáticos en PRs) que no ha sido implementada. Conceder permisos no utilizados viola el principio de mínimo privilegio: en caso de un GitHub Actions injection, el token comprometido no tendrá capacidad de escritura sobre PRs.
  - Fichero: `.github/workflows/opsguard.yml`
  - Rama: `fix/workflow-least-privilege` → `main`

### Added

- **[PR #39]** Filtro de estado en el dashboard web (`ALL` / `BLOCKED` / `APPROVED`) - El Security Scan Feed mostraba todos los runs sin posibilidad de filtrar. Se añaden tres botones de filtro en la cabecera de la tabla; el activo se resalta con el color de su estado (azul/rojo/verde). Implementado con `useState` local sin dependencias adicionales.
  - Fichero: `web/src/app/page.tsx`
  - Rama: `feat/web-status-filter` → `main`

- **[PR #30]** Implementación de modos de telemetría ADR-0003 y migración a Rich UI - Se añaden tres modos configurables vía `OPSGUARD_TELEMETRY_MODE` (verbose / summary / silent). El modo `verbose` (por defecto) emite la tabla FinOps completa con tokens, coste y latencia usando `rich.Table`; `silent` suprime toda la telemetría para entornos CI restringidos. Sustituye los bloques ANSI/print crudos por componentes Rich estructurados.
  - Fichero: `src/ai.py`
  - Rama: `feat/telemetry-modes-rich-ui` → `main`
  - Cierra: FEEDBACK.md §4.1, §4.2, §5.1, §5.5

- **[PR #25]** Suite de tests unitarios para el motor de Gate 1 (`SecurityPolicy`) - 15 tests en 3 clases que cubren: carga de configuración y validación de YAML, detección correcta de secretos con patrón estructural (AWS keys), validación de que vulnerabilidades semánticas (SQL injection, backdoors, typosquatting) pasan correctamente al Gate 2 (IA), y comportamiento ante edge cases del formato diff (líneas eliminadas, cabeceras, diff vacío).
  - Fichero: `tests/test_security.py`
  - Rama: `test/add-unit-tests-security` → `main`

- **[PR #25]** Guía de demostración del Shooting Range (`tests/fixtures/README.md`) - Documento técnico para evaluadores con inventario de los 5 fixtures, atribución de gate por vulnerabilidad, y flujo paso a paso para reproducir el bloqueo del pipeline en GitHub Actions.

### Fixed (continued)

- **[PR #29]** `SkipScanSignal` - excepción tipada reemplaza string matching frágil - El código original lanzaba `GitIngestError("SKIP_SCAN: ...")` y en `main.py` lo capturaba comprobando `"SKIP_SCAN" in str(e)`. Se introduce `SkipScanSignal(GitIngestError)` como excepción propia; `main.py` la captura por tipo. Elimina acoplamiento implícito vía cadena de texto.
  - Ficheros: `src/ingest.py`, `src/main.py`
  - Rama: `fix/skip-scan-typed-exception` → `main`
  - Cierra: FEEDBACK.md §5.2

- **[PR #37]** Comentarios en español homogeneizados a inglés - 12 comentarios y un docstring en `src/console_ui.py`, `src/ingest.py`, `src/main.py` y `pyproject.toml` usaban español, violando ADR-0002 (todos los artefactos técnicos en inglés). Traducidos al inglés manteniendo el significado exacto.
  - Ficheros: `src/console_ui.py`, `src/ingest.py`, `src/main.py`, `pyproject.toml`
  - Rama: `fix/spanish-comments-to-english` → `main`
  - Cierra: FEEDBACK.md §4.3

- **[PR #36]** Modelo LLM hardcodeado - `self.model` leído ahora desde `OPSGUARD_MODEL` env var con `google/gemini-2.0-flash-001` como valor por defecto. El comportamiento sin configuración adicional es idéntico. Permite cambiar de modelo (p.ej. a `google/gemini-2.0-flash-thinking-exp` o cualquier modelo de OpenRouter) sin tocar código fuente, igual que ya funcionan `OPSGUARD_RISK_THRESHOLD` y `OPSGUARD_TELEMETRY_MODE`. Se actualiza `.env.example` con todas las variables disponibles y se añade tabla de referencia en el README.
  - Ficheros: `src/ai.py`, `.env.example`, `README.md`
  - Rama: `fix/model-from-env-var` → `main`
  - Cierra: FEEDBACK.md §8.1

- **[PR #34]** URL `git clone` malformada en README y `authors` sin personalizar en `pyproject.toml` - La URL del Quick Start estaba envuelta en sintaxis Markdown `[url](url)` dentro de un bloque de código, haciendo que se copiara con corchetes en lugar de como URL limpia. El campo `authors` contenía el placeholder `Your Name <you@example.com>`. Ambos corregidos.
  - Ficheros: `README.md`, `pyproject.toml`
  - Rama: `fix/readme-clone-url-and-authors` → `main`
  - Cierra: FEEDBACK.md §9.2, §9.1

- **[PR #35]** `src/net_diag.py` movido a `scripts/` y email de autor corregido - La utilidad de diagnóstico de red (wrapper de `ping`) residía en `src/` sin estar importada por ningún módulo del pipeline, lo que implicaba erróneamente que era parte del core. Se mueve a `scripts/`, lugar estándar para herramientas auxiliares standalone, y se documenta en el árbol de estructura del README. Adicionalmente se corrige el email del autor en `pyproject.toml`.
  - Ficheros: `scripts/net_diag.py` (antes `src/`), `README.md`, `pyproject.toml`
  - Rama: `fix/net-diag-placement-and-docs` → `main`
  - Cierra: FEEDBACK.md §5.4, §9.1 (email)

### Docs

- **[PR #32]** Añadidos campos `Date` y `Deciders` a los tres ADRs - ADR-0001, ADR-0002 y ADR-0003 carecían de los campos de metadata requeridos por el formato MADR estándar. Se añade una tabla `## Metadata` a cada documento con la fecha de decisión (`2026-02-01`) y el decisor (`Óscar Sánchez Pérez`).
  - Ficheros: `docs/adr/0001-*`, `docs/adr/0002-*`, `docs/adr/0003-*`
  - Rama: `docs/adr-metadata-date-deciders` → `main`
  - Cierra: FEEDBACK.md §9.4

- **[PR #31]** README reescrito para tribunal - Tres mejoras orientadas a la evaluación académica: (1) párrafo introductorio reformulado para situar explícitamente la actuación "en el momento del Pull Request, antes de que ningún cambio llegue a la rama principal"; (2) nota de encuadre del directorio `web/` indicando que la ingeniería central reside en `src/` y que el dashboard queda fuera del alcance de la evaluación; (3) nueva sección `📋 Registro de Cambios` con enlace al `CHANGELOG.md` y tabla resumen de versiones.
  - Fichero: `README.md`
  - Rama: `docs/readme-clarity-and-changelog` → `main`
  - Cierra: FEEDBACK.md §9.2 (parcial), recomendación del tribunal

---

## [0.2.0] - 2026-02-20/21 · Supply-Chain Detection & Action Alignment

> Extensión de las capacidades de detección con un caso real de ataque de supply-chain, corrección de la acción de GitHub y mejoras de documentación.

### Added

- **[PR #19]** Fixture de Supply-Chain Attack por typosquatting - `tests/fixtures/vulnerable_app/supply_chain_attack.py` simula el uso del dominio `ghrc.io` (transposición de `ghcr.io`, GitHub Container Registry). Demuestra que el Gate 2 (IA) es capaz de detectar anomalías semánticas que ningún escáner estático convencional identificaría.

- **[PR #20]** Documentación del caso real de typosquatting `ghrc.io` en `README.md` - Incluye tabla comparativa de resultados por gate, contexto del incidente real (Bug Bounty, febrero 2025) y enlace a la evidencia de CI/CD.

### Fixed

- **[PR #21]** `action.yml` referenciaba `OPENAI_API_KEY` en lugar de `OPENROUTER_API_KEY` - La GitHub Action publicable inyectaba una variable de entorno que el código nunca leía, haciendo que la acción fuera no funcional. Corregido el nombre del input y de la variable de entorno. El campo `author` también fue actualizado al nombre real del autor.
  - Fichero: `action.yml`

- **[PR #22]** Botón de refresco del dashboard sin estado de carga visual.
  - Fichero: `web/src/app/page.tsx`

### Docs

- **[PR #18]** Aclaración en `README.md` sobre el método de entrega de la API Key al profesorado (referencia al PDF adjunto en la entrega del TFM).

---

## [0.1.0] - 2026-02-01/02 · TFM Final Delivery

> Versión de entrega académica. Incluye el motor de análisis híbrido completo, documentación de arquitectura (ADRs), evidencias de ejecución y configuración de CI/CD.

### Added

- Motor de análisis estático (Gate 1): `src/security.py` con 14 patrones regex para detección de secretos (AWS, GitHub PATs, Stripe, Slack, claves RSA, etc.).
- Motor de análisis semántico (Gate 2): `src/ai.py` integrado con Google Gemini 2.0 Flash vía OpenRouter. Incluye telemetría FinOps con coste real por ejecución.
- CLI interactiva: `src/main.py` + `src/console_ui.py` con Rich/Typer. Soporta `.opsguardignore` para filtrado de rutas.
- Integración Git dual: `src/ingest.py` opera en modo local (diff HEAD) y modo CI (diff entre SHAs de PR) de forma transparente.
- Pipeline CI/CD: `.github/workflows/opsguard.yml` ejecuta el análisis automáticamente en cada Pull Request.
- Suite de fixtures vulnerables: `tests/fixtures/vulnerable_app/` con AWS credentials, SQL injection, developer backdoor y config PHP con secretos hardcodeados.
- Architecture Decision Records: ADR-0001 (Gatekeeper Local), ADR-0002 (Prompts en inglés), ADR-0003 (Telemetría FinOps).
- Evidencias de ejecución: logs y capturas en `docs/evidence/`.

### Docs

- **[PR #12–16]** Documentación técnica completa: `README.md`, `.env.example`, estructura de proyecto, ADRs refinados y evidencias de ejecución para validación del TFM.

### Chore

- **[PR #17]** Licencia propietaria - El código se hace público únicamente con fines de evaluación académica. Sin permiso para uso, copia o distribución sin autorización expresa del autor.

---

## Metodología de Desarrollo

Todas las modificaciones siguen la especificación **[Conventional Commits](https://www.conventionalcommits.org/)**:

| Prefijo | Uso |
|---------|-----|
| `fix` | Corrección de errores |
| `feat` | Nueva funcionalidad |
| `test` | Tests unitarios o fixtures |
| `docs` | Documentación |
| `chore` | Mantenimiento y configuración |
| `ci` | Cambios en pipelines de CI/CD |

Cada cambio pasa por una **Pull Request independiente** con descripción técnica, test plan y vinculación al punto del FEEDBACK.md que lo origina.

---

**TFM - Máster en Desarrollo con IA** | Óscar Sánchez Pérez | 2026
