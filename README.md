# 🛡️ OpsGuard AI

> **El guardia de seguridad que nunca duerme, revisando tu código antes de que sea un problema.**

![Python](https://img.shields.io/badge/python-3.12-blue)
![Status](https://img.shields.io/badge/status-stable-green)
![CI/CD](https://img.shields.io/badge/github--actions-enabled-brightgreen)

---

## 🤔 ¿De qué va esto? (Explicado para todos)

Imagina que tienes un restaurante y cada día llegan nuevos ingredientes. Antes de meterlos en la cocina, alguien tiene que revisarlos para asegurarse de que no hay nada en mal estado. 

En el mundo del software pasa lo mismo. Los programadores escriben código nuevo todos los días. A veces, por error o por prisa, dejan "puertas abiertas" para los hackers (como contraseñas olvidadas, o fallos de seguridad graves).

**OpsGuard es ese revisor en la puerta de tu cocina.** Analiza cada cambio en el código automáticamente y, si detecta algo peligroso, bloquea la entrada antes de que llegue a producción y afecte a tus clientes.

## ✈️ ¿Cómo funciona? (El control del aeropuerto)

OpsGuard revisa tu código simulando el control de seguridad de un aeropuerto, en dos fases:

1. **El escáner de rayos X (Búsqueda ultrarrápida):** Revisa el código en milisegundos buscando cosas obvias y prohibidas, como contraseñas, tokens o claves secretas. Funciona 100% en local.
2. **El agente experto (Inteligencia Artificial):** Si el escáner no pita, una IA lee el código para entender el *contexto*. Busca trampas ocultas, errores de lógica o código generado por otras IAs (como Copilot) que parezca normal pero sea vulnerable.

**Si en cualquiera de las dos puertas se detecta un problema, el código no pasa.**

## 💡 ¿Por qué OpsGuard?

A diferencia de otras herramientas clásicas que solo buscan palabras exactas en un diccionario, OpsGuard **entiende** lo que hace el código.

- 🔒 **Tus secretos están a salvo:** Las contraseñas se detectan en local, nunca viajan a ninguna IA.
- 💸 **Casi gratis:** Cuesta menos de un céntimo ($0.001) por cada revisión de código.
- 🤖 **Vigila a las otras IAs:** Detecta cuando herramientas como GitHub Copilot generan código inseguro.
- 🔔 **Avisa al instante:** Si bloquea algo, crea un aviso detallado en GitHub automáticamente explicando cómo arreglarlo.

📊 **[Ver comparativa con otras herramientas y métricas de rendimiento](./docs/benchmark-models.md)**

## 🕵️ Ejemplos de lo que detecta
- Contraseñas, Tokens, claves de APIs (AWS, Stripe, GitHub, etc.) o credenciales olvidadas en el código.
- Ataques de inyección (ej. SQL Injections).
- Puertas traseras ("Backdoors") dejadas activas para hacer pruebas.
- Errores tipográficos catastróficos que apuntan a servidores piratas (Typosquatting).

🔍 **[Ver casos reales de código bloqueado por OpsGuard](./docs/ejemplos-reales.md)**

## 🚀 Ponlo a trabajar en 2 minutos

OpsGuard se instala directamente en tu repositorio de GitHub. No tienes que mantener servidores ni instalar nada raro.

1. Consigue una API Key gratuita en [OpenRouter](https://openrouter.ai).
2. Añádela a tu repositorio de GitHub en *Settings → Secrets* con el nombre `OPENROUTER_API_KEY`. *(Esto es lo que permite que la acción de GitHub se conecte a la IA y analice el código de forma automática cada vez que alguien intente subir cambios en un Pull Request).*
3. Crea un archivo llamado `.github/workflows/opsguard.yml` en tu repositorio y pega esto:

```yaml
name: OpsGuard Security Gate

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: oscaar90/OpsGuard-AI@v1
        with:
          openrouter-api-key: ${{ secrets.OPENROUTER_API_KEY }}
```

**¡Ya está!** Tu código ahora tiene un guardia de seguridad revisando cada cambio 24/7.

---

## 📚 Documentación para Ingenieros

Si quieres meterte en las tripas técnicas del proyecto, aquí tienes todo el detalle:

- 💻 **[Guía de instalación y pruebas en local](./docs/guia-local.md)**
- 🏗️ **[Decisiones de Arquitectura (ADRs)](./docs/adr/)**
- 🤖 **[Estrategia de Inteligencia Artificial (Prompts)](./prompts/)**
- 📜 **[Registro de Cambios (Changelog)](./CHANGELOG.md)**

---
*OpsGuard AI está publicado bajo la [Elastic License 2.0 (ELv2)](LICENSE).*