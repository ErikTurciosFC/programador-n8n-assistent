# Programador N8N Assistant

## Objetivo del proyecto

Este proyecto existe para diseñar, construir y mantener workflows de **n8n** de alta calidad junto con Claude Code, usando el servidor MCP `n8n-mcp` y las skills de `n8n-skills`. La instancia objetivo es **n8n Cloud**.

Casos de uso principales (en orden de frecuencia esperada):
1. Integraciones entre apps/SaaS (CRMs, Slack, Google Workspace, HubSpot, etc.)
2. Automatización de datos y backoffice (ETL, sincronización, reportes, procesos internos)
3. Workflows con IA/agentes (nodos de IA, RAG, agentes, procesamiento de lenguaje natural)

## Herramientas disponibles

- **`n8n-mcp`** (https://github.com/czlonkowski/n8n-mcp, git submodule en `./n8n-mcp`): servidor MCP que expone ~2,131 nodos de n8n (core + comunitarios) con su documentación, propiedades y operaciones, y permite leer/crear/editar workflows directamente en la instancia de n8n conectada.
- **`n8n-skills`** (https://github.com/czlonkowski/n8n-skills, git submodule en `./n8n-skills`): 14 skills para Claude Code que enseñan patrones de calidad productiva en n8n (sintaxis de expresiones, patrones de workflow, manejo de errores, sub-workflows, agentes IA, self-hosting, etc.). Se activan automáticamente según el contexto de la consulta.

El servidor MCP se ejecuta vía **Docker** (no npx, porque esta máquina no tiene Node.js/npm instalado), usando la imagen `ghcr.io/czlonkowski/n8n-mcp:latest`. La configuración vive en `.mcp.json` (raíz del proyecto) y referencia `N8N_API_URL`/`N8N_API_KEY` sin valores embebidos — Docker los toma del entorno del proceso que lo invoca. Los hooks de `n8n-skills` (recordatorios proactivos + skill de enrutamiento) están registrados en `.claude/settings.json`, usando la variable portable `${CLAUDE_PROJECT_DIR}` en vez de rutas absolutas, para que funcionen sin cambios en cualquier máquina donde se clone el repo.

Este proyecto vive en GitHub como repo privado: `git@github.com:ErikTurciosFC/programador-n8n-assistent.git`. Al clonarlo hay que usar `git clone --recurse-submodules` (o `git submodule update --init --recursive` después de un clone normal) para traer `n8n-mcp` y `n8n-skills`.

Ver sección "Estado de instalación" para lo que falta.

## Instancia de n8n

- Tipo: **n8n Cloud**
- URL de la instancia: `https://fundcraft.app.n8n.cloud`
- API key: configurada como `N8N_API_KEY` en `.claude/settings.local.json` (no versionado). Conexión verificada (`GET /api/v1/workflows` → 200 OK). **Nunca** debe escribirse en texto plano en archivos versionados de este repo — solo vive en `.claude/settings.local.json`.

## Nivel de autonomía y reglas de seguridad

- Claude **puede crear y modificar** workflows vía `n8n-mcp` sin pedir confirmación previa.
- Claude **debe pedir confirmación explícita antes de activar** cualquier workflow en la instancia real (cambiar de `inactive` a `active`), y antes de cualquier operación destructiva (borrar workflows, borrar credenciales, borrar ejecuciones).
- Antes de tocar un workflow ya activo en producción, describir el cambio y su impacto, y esperar aprobación.
- No hardcodear API keys, tokens ni URLs sensibles en los JSON de workflow ni en archivos del repo; usar credenciales de n8n (el gestor de credenciales nativo) o variables de entorno.
- Si el `.env` u otro archivo con secretos existe en este proyecto, debe estar en `.gitignore` (cuando se inicialice git).
- Las credenciales de las apps/SaaS a integrar (HubSpot, Slack, Google, etc.) **no** están precreadas en n8n. Cuando un workflow necesite una credencial nueva, guiar al usuario paso a paso para crearla en n8n (tipo de credencial, dónde obtener las claves en la app de origen, cómo probarla) antes de continuar con el workflow.

## Flujo de trabajo esperado al construir un workflow

1. Entender el requerimiento (trigger, datos de entrada/salida, sistemas involucrados, condiciones de error).
2. Usar `n8n-mcp` para buscar y validar los nodos relevantes (tipo, versión, propiedades) antes de armar el JSON.
3. Diseñar el workflow siguiendo los patrones de `n8n-skills` (manejo de errores explícito, sub-workflows cuando haya lógica reutilizable, nodos de IA solo cuando aporten valor real).
4. Validar el workflow (sintaxis de expresiones, configuración de nodos) antes de proponerlo.
5. Mostrar al usuario un resumen del workflow (propósito, nodos clave, triggers, manejo de errores) y esperar confirmación antes de activarlo en la instancia.
6. Documentar brevemente cambios relevantes en el propio workflow (nombre claro, notas/sticky notes en n8n) en vez de generar documentación externa no solicitada.

## Convenciones

- Nombrar workflows de forma descriptiva: `[Área] Acción principal` (ej. `[CRM] Sincronizar leads de HubSpot a Sheets`).
- Preferir nodos nativos de n8n sobre nodos de Code cuando exista una alternativa nativa equivalente.
- Todo workflow con llamadas a APIs externas debe tener manejo de errores (nodo `Error Trigger`, ramas de error, o `retry on fail` configurado según corresponda).
- Los flujos con agentes de IA deben tener límites claros de alcance (herramientas permitidas, guardrails) para evitar comportamiento inesperado.

## Estado de instalación

- [x] Clonar `n8n-mcp` en `./n8n-mcp` y `n8n-skills` en `./n8n-skills`
- [x] Copiar las 15 skills de `n8n-skills` a `.claude/skills/` (proyecto)
- [x] Registrar hooks de `n8n-skills` (SessionStart, PreToolUse, PostToolUse) en `.claude/settings.json`
- [x] Crear `.mcp.json` con el servidor `n8n-mcp` vía Docker (imagen `ghcr.io/czlonkowski/n8n-mcp:latest`)
- [x] Descargar la imagen Docker (`docker pull ghcr.io/czlonkowski/n8n-mcp:latest`)
- [x] Configurar credenciales de la instancia n8n Cloud (URL + API key) en `.claude/settings.local.json` — verificadas contra la API
- [ ] Reiniciar/recargar Claude Code para que cargue `.mcp.json` y confirmar con `/mcp` que `n8n-mcp` está conectado

## Idioma

Responder por defecto en español dentro de este proyecto.
