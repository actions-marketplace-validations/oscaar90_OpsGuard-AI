# 💻 Guía de Pruebas en Local

Si prefieres ver cómo funciona OpsGuard AI en tu propia máquina antes de integrarlo en tu pipeline de GitHub, puedes ejecutarlo localmente.

## Requisitos

- Python 3.12 o superior
- [Poetry](https://python-poetry.org/docs/) para gestionar dependencias.

## 1. Instalación

Clona el repositorio y descarga las dependencias:

```bash
git clone https://github.com/oscaar90/OpsGuard-AI.git
cd OpsGuard-AI
poetry install
```

## 2. Configuración

Copia el archivo de ejemplo de variables de entorno:

```bash
cp .env.example .env
```

Abre el archivo `.env` y añade tu clave de OpenRouter:
`OPENROUTER_API_KEY=tu_clave_aqui`

*(Nota: Solo necesitas la clave para la "Puerta 2" (IA). La "Puerta 1" (secretos) funciona 100% offline).*

## 3. Pruebas con código vulnerable

El repositorio incluye ejemplos de código peligroso en `tests/fixtures/vulnerable_app/` para que pruebes el sistema.

### Prueba 1: Detección estática (Sin gastar tokens de IA)
Vamos a intentar "colar" un archivo con credenciales de AWS.

```bash
cp tests/fixtures/vulnerable_app/aws_creds.env src/aws_creds.env
git add src/aws_creds.env
poetry run opsguard scan
```
**Resultado esperado:** OpsGuard bloquea la acción instantáneamente en la Puerta 1.

### Prueba 2: Detección contextual (Con IA)
Vamos a intentar subir código con una vulnerabilidad lógica de inyección SQL.

```bash
cp tests/fixtures/vulnerable_app/legacy_login.py src/legacy_login.py
git add src/legacy_login.py
poetry run opsguard scan
```
**Resultado esperado:** La Puerta 1 no encuentra contraseñas fijas, pero la Puerta 2 (IA) analiza el código, detecta la inyección SQL y bloquea el commit.

## 4. Ejecutar los tests automáticos

Puedes ejecutar la suite completa de pruebas unitarias (no requieren API Key):

```bash
poetry run pytest tests/ -v
```