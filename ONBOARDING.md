# Cómo usar el Asistente de n8n con Claude Code

Este repo conecta Claude Code con nuestra instancia de **n8n Cloud** (`fundcraft.app.n8n.cloud`) para diseñar, construir y mantener workflows de n8n directamente desde el chat. Esta guía es para clonarlo y dejarlo funcionando en tu máquina.

## Requisitos previos

- **Claude Code** instalado.
- **Git** instalado (`git --version` para comprobar).
- **Docker Desktop** instalado y corriendo (el servidor MCP corre en un contenedor).
- Acceso al repo de GitHub: pide a Erik que te invite como colaborador de `ErikTurciosFC/programador-n8n-assistent` (es privado).
- Una clave SSH en GitHub. Si no tienes una:
  ```
  ssh-keygen -t rsa -b 4096
  cat ~/.ssh/id_rsa.pub
  ```
  Copia el resultado en GitHub → **Settings → SSH and GPG keys → New SSH key**.

## 1. Clonar el repo

Este repo usa **git submodules** (`n8n-mcp` y `n8n-skills`), así que hay que clonar con `--recurse-submodules`:

```bash
git clone --recurse-submodules git@github.com:ErikTurciosFC/programador-n8n-assistent.git
cd programador-n8n-assistent
```

Si ya clonaste sin ese flag, corre esto dentro de la carpeta:
```bash
git submodule update --init --recursive
```

## 2. Descargar la imagen del servidor MCP

```bash
docker pull ghcr.io/czlonkowski/n8n-mcp:latest
```

Docker Desktop debe estar **corriendo** cada vez que uses Claude Code en este proyecto — el MCP server se levanta como contenedor bajo demanda.

## 3. Configurar tu API key de n8n Cloud

El repo **no** trae ninguna API key (por seguridad, cada quien usa la suya). Hay que crear un archivo local que nunca se sube a git:

1. Entra a `https://fundcraft.app.n8n.cloud` → **Settings → n8n API → Create an API key**.
2. Crea el archivo `.claude/settings.local.json` en la raíz del proyecto con este contenido:

```json
{
  "env": {
    "N8N_API_URL": "https://fundcraft.app.n8n.cloud",
    "N8N_API_KEY": "TU_API_KEY_AQUI"
  }
}
```

Este archivo ya está en `.gitignore` — no hace falta (ni se debe) commitearlo.

## 4. Abrir el proyecto y verificar la conexión

1. Abre Claude Code en la carpeta del repo.
2. Corre `/mcp` y confirma que `n8n-mcp` aparece como conectado.
3. Si no conecta: revisa que Docker Desktop esté corriendo y recarga la sesión de Claude Code.

## Cómo funciona el asistente

- Las **skills** de `.claude/skills/` se activan solas según lo que pidas (expresiones, manejo de errores, sub-workflows, agentes de IA, etc.) — no hace falta invocarlas a mano.
- Reglas de seguridad ya configuradas (ver [CLAUDE.md](CLAUDE.md) para el detalle completo):
  - Claude puede **crear y modificar** workflows sin pedir permiso.
  - Claude **siempre pregunta antes de activar** un workflow en la instancia real, o antes de borrar algo (workflows, credenciales, ejecuciones).
  - Nunca hardcodea API keys ni tokens en los JSON de los workflows — usa el gestor de credenciales nativo de n8n.
- Responde por defecto en español.

## Ejemplo de uso

> "Créame un workflow que, cuando llegue un lead nuevo a HubSpot, lo mande a un canal de Slack."

Claude va a buscar los nodos correctos con `n8n-mcp`, armar el workflow siguiendo los patrones de `n8n-skills`, validarlo, y mostrarte un resumen antes de dejarlo activo.

## Dudas o problemas

Habla con Erik Turcios (erik.turcios@fundcraft.lu).
