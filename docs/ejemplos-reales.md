# 🔍 Casos Reales Detectados por OpsGuard

OpsGuard está diseñado para detectar vulnerabilidades que van desde simples descuidos hasta ataques lógicos complejos. Aquí tienes algunos ejemplos reales de lo que OpsGuard es capaz de bloquear antes de que llegue a producción.

---

## Caso 1: Una contraseña olvidada en el código (Puerta 1)

Un desarrollador está probando una conexión a AWS y escribe sus credenciales directamente en el código para hacer una prueba rápida. Se olvida de quitarlas antes de subir el cambio.

```python
AWS_ACCESS_KEY = "AKIAIOSFODNN7EXAMPLE"
AWS_SECRET_KEY = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```

**Cómo actúa OpsGuard:**
Lo detecta en menos de 1ms usando análisis estático local (sin llamar a ninguna IA externa) y bloquea el código.

```text
🚨 DETECTED 1 STATIC VIOLATIONS:
┌──────────────────────────────────────────────────────────┬──────┐
│ Type                                                     │ File │
├──────────────────────────────────────────────────────────┼──────┤
│ [AWS Access Key] Found pattern: AKIAIOSFODNN7EXAMPLE     │ Diff │
└──────────────────────────────────────────────────────────┴──────┘
⛔ PIPELINE BLOCKED: SECURITY VIOLATION DETECTED
```

> 🔒 **Privacidad:** La contraseña nunca sale de tu entorno. OpsGuard la detecta localmente.

---

## Caso 2: Código que parece normal pero es una puerta trasera (Puerta 2)

Un desarrollador escribe una función de login. No hay contraseñas a la vista, por lo que el escáner rápido (Puerta 1) no encuentra nada. Pero el código tiene una trampa para facilitar las pruebas:

```python
def validate_request(headers):
    # TODO: remove debug bypass before deploy
    if headers.get("X-DEBUG-MODE") == "true":
        return True  # Bypass auth completely
```

Cualquier atacante que envíe la cabecera `X-DEBUG-MODE: true` entrará al sistema sin contraseña.

**Cómo actúa OpsGuard:**
La IA lee el código, entiende el contexto de autenticación y detecta el peligro lógico.

```text
⛔ Verdict: BLOCK | Risk Score: 8/10

CRITICAL │ auth_middleware.py │ L8 │ Developer backdoor via X-DEBUG-MODE header
         │                    │    │ Any request with this header bypasses authentication completely
```

---

## Caso 3: Un error tipográfico que hunde la empresa (Puerta 2)

Alguien escribe en un script de despliegue:

```python
REGISTRY = "ghrc.io"  # debería ser ghcr.io
```

Una sola letra cambiada. Las herramientas tradicionales no detectan nada porque `ghrc.io` es un formato de dominio válido. Sin embargo, es un dominio malicioso de *typosquatting* registrado para interceptar imágenes de empresas.

**Cómo actúa OpsGuard:**
La IA tiene conocimiento de este vector de ataque de la cadena de suministro y bloquea la subida.

```text
⛔ Verdict: BLOCK | Risk Score: 7/10

HIGH │ supply_chain_attack.py │ L10 │ Typosquatting: "ghrc.io" is not the official
     │                        │     │ GitHub Container Registry. Correct domain is "ghcr.io".
```

---

## Caso 4: Vulnerabilidades introducidas por IAs de programación (Puerta 2)

Un desarrollador le pide a una IA (como Copilot o Cursor) que haga una función para consultar saldos. La IA genera esto:

```python
def get_user_balance(username: str) -> float:
    query = f"SELECT balance FROM accounts WHERE username = '{username}'"
    conn = sqlite3.connect("payments.db")
    row = conn.execute(query).fetchone()
    return row[0] if row else 0.0
```

Este código tiene una inyección SQL gravísima. No hay secretos visibles, pero la lógica es fatal.

**Cómo actúa OpsGuard:**
Actúa como un revisor de seguridad humano (AppSec) revisando el código de la otra IA.

```text
⛔ Verdict: BLOCK | Risk Score: 9/10

CRITICAL │ payment_service.py │ L13 │ SQL injection: username interpolated directly into query
```

---

## Alertas Automáticas en GitHub

Cuando OpsGuard bloquea algo, no se queda en un log perdido. Automáticamente abre un *Issue* en GitHub con todo el detalle para que el equipo lo vea y lo solucione.

| Amenaza detectada | Puerta |
|---|:---:|
| AWS Access Key hardcodeada | Puerta 1 |
| f-string directa en query SQL | Puerta 2 |
| Header `X-DEBUG-MODE` que bypasea auth | Puerta 2 |
| Dominio falso `ghrc.io` | Puerta 2 |