# CHANGELOG

Registro de cambios del proyecto **OpsGuard-AI** siguiendo el formato [Keep a Changelog](https://keepachangelog.com/es/1.0.0/) y la especificación [Semantic Versioning](https://semver.org/).

Cada entrada está vinculada a su Pull Request en GitHub para trazabilidad completa del proceso de ingeniería.

---

## [0.3.0] — 2026-02-21 · Quality Audit Sprint

> Ciclo de mejora continua iniciado tras un análisis técnico exhaustivo del proyecto (`FEEDBACK.md`). Las correcciones cubren dos categorías: **blockers de runtime** que impedían la ejecución en entornos limpios, y **deuda técnica** que comprometía la configurabilidad y el principio de mínimo privilegio.

### Fixed

- **[PR #24]** `NameError: MAX_DIFF_CHARS is not defined` — La constante que controla el truncado del diff enviado al LLM había sido referenciada pero nunca declarada. Causaba fallo total en runtime. Se define como `MAX_DIFF_CHARS = 30_000` junto a las constantes FinOps existentes.
  - Fichero: `src/ai.py`
  - Rama: `fix/diff-truncation-feedback` → `main`

- **[PR #23]** `ModuleNotFoundError: No module named 'pathspec'` — La librería `pathspec`, usada para procesar `.opsguardignore`, se importaba en `src/main.py` pero no estaba declarada en `pyproject.toml`. La herramienta fallaba en cualquier instalación limpia (`poetry install`), incluyendo CI/CD.
  - Fichero: `pyproject.toml`
  - Rama: `fix/add-pathspec-dependency` → `main`

- **[PR #26]** `OPSGUARD_RISK_THRESHOLD` declarado en el workflow pero ignorado en el código — El threshold de riesgo del Gate 2 estaba hardcodeado como `7` en `src/main.py` mientras el workflow CI/CD ya lo definía como variable de entorno. Se lee ahora vía `os.getenv("OPSGUARD_RISK_THRESHOLD", "7")`, haciendo el sistema configurable sin modificar código fuente.
  - Fichero: `src/main.py`
  - Rama: `fix/risk-threshold-configurable` → `main`

### Security

- **[PR #27]** Eliminado permiso `pull-requests: write` no utilizado del workflow de GitHub Actions — El permiso estaba declarado para una funcionalidad futura (comentarios automáticos en PRs) que no ha sido implementada. Conceder permisos no utilizados viola el principio de mínimo privilegio: en caso de un GitHub Actions injection, el token comprometido no tendrá capacidad de escritura sobre PRs.
  - Fichero: `.github/workflows/opsguard.yml`
  - Rama: `fix/workflow-least-privilege` → `main`

### Added

- **[PR #25]** Suite de tests unitarios para el motor de Gate 1 (`SecurityPolicy`) — 15 tests en 3 clases que cubren: carga de configuración y validación de YAML, detección correcta de secretos con patrón estructural (AWS keys), validación de que vulnerabilidades semánticas (SQL injection, backdoors, typosquatting) pasan correctamente al Gate 2 (IA), y comportamiento ante edge cases del formato diff (líneas eliminadas, cabeceras, diff vacío).
  - Fichero: `tests/test_security.py`
  - Rama: `test/add-unit-tests-security` → `main`

- **[PR #25]** Guía de demostración del Shooting Range (`tests/fixtures/README.md`) — Documento técnico para evaluadores con inventario de los 5 fixtures, atribución de gate por vulnerabilidad, y flujo paso a paso para reproducir el bloqueo del pipeline en GitHub Actions.

---

## [0.2.0] — 2026-02-20/21 · Supply-Chain Detection & Action Alignment

> Extensión de las capacidades de detección con un caso real de ataque de supply-chain, corrección de la acción de GitHub y mejoras de documentación.

### Added

- **[PR #19]** Fixture de Supply-Chain Attack por typosquatting — `tests/fixtures/vulnerable_app/supply_chain_attack.py` simula el uso del dominio `ghrc.io` (transposición de `ghcr.io`, GitHub Container Registry). Demuestra que el Gate 2 (IA) es capaz de detectar anomalías semánticas que ningún escáner estático convencional identificaría.

- **[PR #20]** Documentación del caso real de typosquatting `ghrc.io` en `README.md` — Incluye tabla comparativa de resultados por gate, contexto del incidente real (Bug Bounty, febrero 2025) y enlace a la evidencia de CI/CD.

### Fixed

- **[PR #21]** `action.yml` referenciaba `OPENAI_API_KEY` en lugar de `OPENROUTER_API_KEY` — La GitHub Action publicable inyectaba una variable de entorno que el código nunca leía, haciendo que la acción fuera no funcional. Corregido el nombre del input y de la variable de entorno. El campo `author` también fue actualizado al nombre real del autor.
  - Fichero: `action.yml`

- **[PR #22]** Botón de refresco del dashboard sin estado de carga visual.
  - Fichero: `web/src/app/page.tsx`

### Docs

- **[PR #18]** Aclaración en `README.md` sobre el método de entrega de la API Key al profesorado (referencia al PDF adjunto en la entrega del TFM).

---

## [0.1.0] — 2026-02-01/02 · TFM Final Delivery

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

- **[PR #17]** Licencia propietaria — El código se hace público únicamente con fines de evaluación académica. Sin permiso para uso, copia o distribución sin autorización expresa del autor.

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

**TFM — Máster en Desarrollo con IA** | Óscar Sánchez Pérez | 2026
