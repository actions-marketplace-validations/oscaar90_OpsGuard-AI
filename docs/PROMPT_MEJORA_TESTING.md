# Prompt para Claude Code — Mejora #1: Testing y Calidad

> Copia este prompt completo y ejecútalo en Claude Code desde la raíz del repositorio OpsGuard-AI.

---

## PROMPT

Eres un ingeniero de software senior trabajando en el repositorio OpsGuard-AI, un sistema de seguridad DevSecOps con arquitectura Two-Gate para análisis de PRs. Tu misión es implementar la mejora de Testing y Calidad identificada en `docs/PLAN_MEJORA_TFM_BraisMoure.md` (GAP −2.0 pts).

### CONTEXTO DEL PROYECTO

**Estructura relevante:**
- `src/ai.py` — Gate 2: motor LLM vía OpenRouter. Clase `AIEngine`, método `analyze_diff()`. **Sin tests actualmente.**
- `src/security.py` — Gate 1: regex determinista. Ya tiene 15 tests en `tests/test_security.py`.
- `src/main.py` — Orquestador CLI con Typer. Coordina Gate 1 + Gate 2.
- `src/ingest.py` — Abstracción Git. Ya tiene 13 tests en `tests/test_ingest.py`.
- `tests/fixtures/vulnerable_app/` — 5 fixtures reales: `aws_creds.env`, `legacy_login.py`, `auth_middleware.py`, `config.php`, `supply_chain_attack.py`.
- `pyproject.toml` — Poetry. Dev deps actuales: solo `pytest` y `black`. **Sin pytest-cov ni pytest-mock.**
- `.github/workflows/opsguard.yml` — Job `test` ejecuta `pytest tests/ -v`. **Sin gate de cobertura.**

**Comportamiento crítico de `src/ai.py`:**
- Política fail-closed: cualquier excepción en `analyze_diff()` retorna `{"verdict": "BLOCK", "risk_score": 10, "explanation": "...", "findings": []}`.
- Trunca el diff a `MAX_DIFF_CHARS = 30_000` antes de enviarlo al LLM.
- El `SYSTEM_PROMPT` está hardcodeado como string en el módulo.
- Telemetría configurable via `OPSGUARD_TELEMETRY_MODE` (verbose/summary/silent).
- Usa `openai.OpenAI` con `base_url="https://openrouter.ai/api/v1"` y la API key de `OPENROUTER_API_KEY`.
- El modelo se configura via `OPSGUARD_MODEL` (default: `google/gemini-2.0-flash-001`).
- Retorna un dict con keys: `verdict` (APPROVE/BLOCK), `risk_score` (0-10), `explanation` (str), `findings` (list).

---

### OBJETIVO

Implementar las 3 mejoras de testing en este orden exacto, en una rama nueva, y hacer merge a `main` si todo pasa correctamente.

---

### INSTRUCCIONES PASO A PASO

#### PASO 1 — Crear rama de trabajo

```bash
git checkout -b feat/testing-ai-engine-coverage
```

#### PASO 2 — Añadir dependencias de desarrollo

Añade a `pyproject.toml` en la sección `[tool.poetry.dev-dependencies]`:
- `pytest-cov = "^4.1.0"` — para medir cobertura
- `pytest-mock = "^3.12.0"` — para mocks con `mocker` fixture

Ejecuta `poetry lock --no-update && poetry install` para actualizar el entorno.

#### PASO 3 — Crear `tests/test_ai.py` con cobertura completa de `src/ai.py`

Crea el archivo `tests/test_ai.py` con los siguientes casos de test, organizados en clases. Usa `pytest-mock` (fixture `mocker`) para mockear las llamadas a la API de OpenRouter. **Nunca hagas llamadas reales a la API.**

**Clase `TestAIEngineInit`:**
- `test_init_success` — Inicializa `AIEngine` con `OPENROUTER_API_KEY="test-key"` en el entorno. Verifica que el objeto se crea sin error.
- `test_init_missing_api_key` — Sin `OPENROUTER_API_KEY` en el entorno. Verifica que lanza `AIEngineError` con mensaje que menciona la API key.

**Clase `TestAnalyzeDiffApprove`:**
- `test_approve_verdict` — Mockea `openai.OpenAI` para que `client.chat.completions.create()` retorne una respuesta con content JSON válido: `{"verdict": "APPROVE", "risk_score": 2, "explanation": "Clean code", "findings": []}`. Verifica que `analyze_diff("+ x = 1")` retorna dict con `verdict == "APPROVE"` y `risk_score == 2`.
- `test_approve_with_telemetry_verbose` — Igual que el anterior pero con `OPSGUARD_TELEMETRY_MODE=verbose`. Verifica que no lanza excepción y retorna el dict correcto.

**Clase `TestAnalyzeDiffBlock`:**
- `test_block_verdict` — Mockea la respuesta con `{"verdict": "BLOCK", "risk_score": 9, "explanation": "SQL injection found", "findings": ["SQL injection in login.py"]}`. Verifica `verdict == "BLOCK"` y `risk_score == 9`.

**Clase `TestAnalyzeDiffFailClosed` (política más importante):**
- `test_api_timeout_returns_block` — Mockea `client.chat.completions.create()` para que lance `Exception("Connection timeout")`. Verifica que `analyze_diff()` retorna `{"verdict": "BLOCK", "risk_score": 10, ...}` sin propagar la excepción. Esto valida la política fail-closed.
- `test_malformed_json_returns_block` — Mockea la respuesta con content `"esto no es json válido {{{"`. Verifica que retorna `verdict == "BLOCK"` y `risk_score == 10`.
- `test_response_is_list_returns_block` — Mockea la respuesta con content `'[{"verdict": "APPROVE"}]'` (lista en lugar de dict). Verifica fail-closed.
- `test_missing_verdict_key_returns_block` — Mockea la respuesta con JSON válido pero sin la key `verdict`: `{"risk_score": 5}`. Verifica fail-closed.

**Clase `TestDiffTruncation`:**
- `test_diff_truncated_to_max_chars` — Crea un diff de longitud `MAX_DIFF_CHARS + 5000`. Mockea la API. Captura los argumentos con los que se llama a `create()`. Verifica que el contenido enviado al LLM no supera `MAX_DIFF_CHARS` caracteres.
- `test_short_diff_not_truncated` — Crea un diff corto (100 chars). Verifica que se envía completo sin truncar.

**Clase `TestTelemetryModes`:**
- `test_telemetry_silent_mode` — Con `OPSGUARD_TELEMETRY_MODE=silent`, mockea una respuesta APPROVE. Verifica que retorna el resultado correcto.
- `test_telemetry_summary_mode` — Igual con `summary`.

**Fixture compartida sugerida:**
```python
@pytest.fixture
def mock_openrouter_response():
    """Crea una respuesta mock válida de OpenRouter."""
    # Retorna un objeto que simula openai.types.chat.ChatCompletion
    # con .choices[0].message.content = json_string
    # y .usage.prompt_tokens, .usage.completion_tokens
```

#### PASO 4 — Crear `tests/test_e2e.py` — Test de integración end-to-end

Crea `tests/test_e2e.py` con un test que verifique el contrato público de la herramienta completa.

**Clase `TestEndToEnd`:**
- `test_gate1_blocks_aws_credentials` — Usa el fixture `tests/fixtures/vulnerable_app/aws_creds.env`. Construye un diff simulado con el contenido de ese archivo (líneas prefijadas con `+`). Mockea `GitManager` para que `get_staged_files()` retorne `["aws_creds.env"]` y `get_diff()` retorne el diff construido. Mockea `AIEngine.analyze_diff()` (no debería llegar a llamarse si Gate 1 bloquea). Invoca la función `scan()` de `src/main.py` via `typer.testing.CliRunner`. Verifica que el exit code es `1` (BLOCKED).

- `test_gate2_blocks_sql_injection` — Usa el fixture `legacy_login.py`. Gate 1 no debe bloquearlo (SQL injection pasa Gate 1). Mockea `GitManager` y `AIEngine.analyze_diff()` para que retorne `{"verdict": "BLOCK", "risk_score": 8, "explanation": "SQL injection", "findings": ["SQL injection detected"]}`. Verifica exit code `1`.

- `test_clean_diff_passes_both_gates` — Construye un diff limpio (`+ x = 1\n+ y = 2`). Gate 1 no detecta nada. Mockea `AIEngine.analyze_diff()` con `{"verdict": "APPROVE", "risk_score": 1, "explanation": "Clean", "findings": []}`. Verifica exit code `0`.

- `test_pipeline_without_api_key` — Sin `OPENROUTER_API_KEY` en el entorno. Diff limpio. Verifica que el proceso termina con exit code `0` (Gate 1 pasa, Gate 2 se omite por falta de API key) y no lanza excepción no controlada.

#### PASO 5 — Actualizar el gate de cobertura en `.github/workflows/opsguard.yml`

En el job `test`, modifica el step de pytest de:
```yaml
- run: poetry run pytest tests/ -v
```
a:
```yaml
- run: poetry run pytest tests/ -v --cov=src --cov-report=term-missing --cov-fail-under=80
```

Este cambio hace que el CI falle si la cobertura de `src/` baja del 80%.

#### PASO 6 — Ejecutar los tests localmente y verificar

```bash
poetry run pytest tests/ -v --cov=src --cov-report=term-missing --cov-fail-under=80
```

Verifica que:
1. Todos los tests pasan (0 fallos).
2. La cobertura global de `src/` supera el 80%.
3. `src/ai.py` tiene cobertura > 85% gracias a los nuevos tests.

Si algún test falla, analiza el error, corrige la implementación del test (no el código de producción a menos que haya un bug real), y re-ejecuta.

#### PASO 7 — Verificar que el linter no falla

```bash
poetry run black --check src/ tests/
```

Si falla, ejecuta `poetry run black src/ tests/` para formatear y re-verifica.

#### PASO 8 — Commit con Conventional Commits

Solo si TODOS los tests pasan y el linter no reporta errores:

```bash
git add tests/test_ai.py tests/test_e2e.py pyproject.toml poetry.lock .github/workflows/opsguard.yml
git commit -m "test: add unit tests for AIEngine and E2E pipeline tests

- Add tests/test_ai.py with full coverage of src/ai.py:
  - AIEngine initialization (success and missing API key)
  - analyze_diff() happy paths (APPROVE and BLOCK verdicts)
  - Fail-closed policy validation (timeout, malformed JSON, unexpected types)
  - Diff truncation behavior (MAX_DIFF_CHARS enforcement)
  - Telemetry modes (verbose, summary, silent)
- Add tests/test_e2e.py with end-to-end pipeline tests:
  - Gate 1 blocks AWS credentials (exit code 1)
  - Gate 2 blocks SQL injection via mock (exit code 1)
  - Clean diff passes both gates (exit code 0)
  - Pipeline without API key runs safely (exit code 0)
- Add pytest-cov and pytest-mock to dev dependencies
- Add --cov-fail-under=80 gate to CI test job

Closes gap identified in docs/PLAN_MEJORA_TFM_BraisMoure.md (Testing -2.0pts)

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

#### PASO 9 — Merge a main (solo si todo es correcto)

Condiciones para hacer el merge:
- [ ] `poetry run pytest tests/ -v --cov=src --cov-fail-under=80` → 0 fallos, cobertura ≥ 80%
- [ ] `poetry run black --check src/ tests/` → sin errores de formato
- [ ] El commit está creado con el mensaje correcto

Si todas las condiciones se cumplen:

```bash
git checkout main
git merge feat/testing-ai-engine-coverage --no-ff -m "feat: implement testing improvements for AI engine coverage

Merge branch 'feat/testing-ai-engine-coverage'

Implements the testing quality improvement identified in the TFM evaluation
(GAP -2.0 points). Adds comprehensive unit tests for src/ai.py, E2E pipeline
tests, pytest-cov with 80% coverage gate in CI.

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

Si alguna condición NO se cumple, **no hagas el merge**. Reporta qué falló con detalle y sugiere cómo corregirlo.

---

### RESTRICCIONES IMPORTANTES

1. **No modifiques código de producción** (`src/`) a menos que encuentres un bug real que impida el testing. Si lo haces, documenta exactamente qué y por qué.
2. **No hagas llamadas reales a la API de OpenRouter.** Todos los tests de `ai.py` deben usar mocks.
3. **Los tests deben ser deterministas** — mismo resultado siempre, sin dependencia de red, reloj o estado externo.
4. **No uses `unittest.mock.patch` con strings frágiles** si `pytest-mock` con `mocker.patch.object()` es más seguro.
5. **El merge solo ocurre si pytest y black pasan sin errores.** No hay excepciones.

