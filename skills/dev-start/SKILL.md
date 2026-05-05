---
name: dev.start
description: Inicializa una nueva funcionalidad o proyecto en el entorno de desarrollo. Maneja la creación de la estructura de carpetas, configuración del repositorio base, detección de stack y meta.md. Permite modos express, lite, audio, o resumir desde backlog.
model: sonnet
argument-hint: "[feature-description] [--express|--lite|--audio|--from-backlog|--reopen]"
---

### REGLAS ESTRICTAS DE LECTURA E INTERACCIÓN

> [!IMPORTANT]
> Si en algún momento del flujo encuentras:
> 
> ⛔ INVOKE TOOL (do not print this, CALL the tool):
> `AskUserQuestion(questions=[{...}])`
> 
> ESTO ES UNA LLAMADA A LA HERRAMIENTA. DEBES LLAMARLA. NUNCA imprimas el JSON en la respuesta. NUNCA imprimas opciones con `echo` ni pidas que el usuario escriba la respuesta como texto plano si la herramienta está disponible.

# Comando: `/dev.start`

**Misión**: Preparar el entorno para una nueva característica (Feature) creando el scaffold necesario, gestionando ramas y detectando tecnología.

**Sintaxis**: `/dev.start [feature-description] [flags]`

**Opciones Relevantes**:
- `/dev.start "descripcion"` → Flujo normal.
- `--express` → Omite preguntas innecesarias.
- `--lite` → Template reducido (~80 líneas) para MVPs.
- `--audio` → Dictado por voz de los requerimientos.
- `--from-backlog <ID>` → Convierte un ítem de backlog en feature WIP.
- `--rename [new-name]` → Renombra la feature actual (solo si no hay tareas activas).
- `--reopen [name]` → Revive una feature archivada.

---

## Flujo de Trabajo (Workflows)

### Paso 0: Verificación de Perfil (BLOQUEANTE)

Antes de hacer NADA, revisa si existe el archivo `~/.dev-sdd-kit/user-profile.yaml`.
Si NO existe, DEBES usar `AskUserQuestion` para que el usuario elija entre:
- **Business/Product Focus**: Todo se decide automáticamente, modo express por defecto, sin capas técnicas visibles.
- **Technical Focus**: Control total sobre arquitectura y código.
Luego de elegir el perfil, guarda el YAML y entonces continúa.

### Paso 1: Validación de Input

Verifica si el nombre de la feature es válido (kebab-case). Si el usuario escribió un prompt largo, infiere el nombre automáticamente y continúa.
La feature NO debe existir previamente en la carpeta `dev/wip/`. Si existe, añade sufijos para que sea única o pregunta con `AskUserQuestion`.

### Paso 2: Detección Inicial (Móvil vs Backend)

Usa script para revisar la plataforma:
```bash
stack_result=$(bash ~/.dev-sdd-kit/tools/detection/detect-stack.sh . --json 2>/dev/null)
```
Si es `android` o `ios`, omite toda validación gary. Asegúrate de verificar que las habilidades móviles (`dev-frontender-android` o `ios`) están instaladas.
Si no es móvil, revisa que gary CLI está configurado (`gary-prereqs.sh`), que el token Zero Trust es válido, y busca la existencia del archivo `.gary`.

**Si falta el archivo `.gary`**: Infiere el nombre desde la carpeta o usa `AskUserQuestion` para crear la app (via `mcp__garyPumaMCP__create_application()`). 
> **CRÍTICO**: Si creas la app con MCP, debes guardar `app_created_via_puma=true` y el stack seleccionado para usar `--template` en la fase de clonación. Al crear apps web, solicita seleccionar el sistema al que pertenece usando la lista de sistemas de Puma MCP.

### Paso 3: Clonación y Scaffolding

Descarga el repositorio:
```bash
# SI SE CREÓ CON PUMA RECIÉN, OBLIGATORIO:
gary get <app-name> --template "$template" 2>&1
```
*(Si no se creó con Puma, no agregues el `--template`)*

Tras descargar:
1. Mueve todo al directorio actual.
2. Ejecuta `detect-scaffolding-status.sh` para ver si hay que borrar archivos de ejemplo (archivos dummy de Java, Go, Python, Node, etc.).

Revisa la presencia de archivos mínimos necesarios (`pom.xml`, `go.mod`, `package.json`, `.gary`, `Dockerfile` o los manifiestos móviles correspondientes).

### Paso 4: Greenfield vs Brownfield

Determina si es proyecto nuevo (Greenfield) o heredado (Brownfield).
Si es Brownfield y NO existen las carpetas `dev/specs`, muestra una alerta recomendando correr `/dev.reverse-eng` primero para ganar contexto, usando un `AskUserQuestion` para frenar o continuar.
Si avanza en Brownfield, extrae dependencias y patrones de entrada usando `analyze-structure.sh`.

### Paso 5: Selección del Tipo de Proyecto

Exhibe una tabla comparando `Prototype`, `MVP` y `Production`.
Usa `AskUserQuestion` para que decida.
*(Nota: Si es Prototype, nos saltaremos el archivo de convenciones `PROJECT.md`)*.

### Paso 6: Estructura de Directorios WIP

Genera una marca de fecha YYYYMMDD y el nombre base inferido.
Crea el path: `dev/wip/YYYYMMDD-nombre-feature/` con sus subcarpetas (`1-functional`, `2-technical`, `3-tasks`, `4-implementation/artifacts`).
Crea el archivo `meta.md` allí dentro, insertando toda la configuración recolectada (perfil, stack, etc).
> Nunca uses prefijos secuenciales (001, 002) para nuevas features. Siempre el prefijo de fecha de hoy.

### Paso 7: Git Checkout

```bash
git checkout -b "feature/$feature_name"
```
Asegura que el Working Tree esté limpio antes de moverse de rama.

### Paso 8: Integración CLAUDE.md

Si la carpeta `.claude/` existe, inyecta (o actualiza) una sección "## dev SDD Kit" en el archivo `CLAUDE.md`.
Si el proyecto es móvil, inserta la regla "Mobile Implementation Rule", forzando a leer el SKILL.md de la tecnología móvil en todas las tareas posteriores.

### Paso 9: Cierre e Instrucciones

Lee `PATTERNS.md` si existe para conocer convenciones heredadas.
Imprime el resumen de éxito:
```
✅ Feature 'nombre' inicializada
📁 dev/wip/YYYYMMDD-nombre/
```
Llama a `AskUserQuestion` para sugerir opciones de continuación como `/dev.spec (Recommended)`, `/dev.spec --audio` o `/dev.check`.

---

## Banderas Auxiliares

### `--audio`
Delega en `Skill("audio-capture")`. Pasa el resultado como texto base para empezar.

### `--rename [nuevo-nombre]`
Permite arreglar el título de la feature si el nombre inferido fue malo.
Mueve la carpeta, ajusta `meta.md` y `tasks.json`. Falla si ya hay tareas de implementación en curso.

### `--from-backlog <ID>`
Toma los requerimientos de `dev/backlog.md`, inicia la feature precargando el contexto y marca el ítem como "in-progress".

### `--reopen [name]`
Revierte una feature terminada (`dev/features/`) de vuelta al `dev/wip/` para añadirle requerimientos y retrabajarla. (Requiere leer `references/reopen-workflow.md`).
