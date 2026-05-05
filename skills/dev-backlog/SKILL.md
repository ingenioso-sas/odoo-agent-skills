---
name: dev.backlog
description: Módulo central para gestionar el Backlog Técnico (TODO, DEBT, IDEA). Úsalo cuando el usuario necesite agregar, revisar o procesar tareas pendientes.
model: sonnet
argument-hint: "[action] [item]"
---

### GUÍA DE EJECUCIÓN DEL AGENTE

> [!IMPORTANT]
> Cuando encuentres un bloque con el texto "⛔ INVOKE TOOL", debes **ejecutar la herramienta** correspondiente, nunca imprimir el JSON en la terminal.
> 
> Ejemplo:
> `AskUserQuestion(questions=[{...}])` -> LLAMA a la tool con estos parámetros. No uses `echo`, `cat` ni preguntes en texto plano. Las tablas marcadas como "REFERENCE ONLY" son informativas y no deben imprimirse.

# Comando principal: `/dev.backlog`

> **Nota de compatibilidad**: El alias `/dev.todos` sigue siendo válido para invocar esta skill.

**Propósito Principal**: Administrar la deuda técnica, las ideas y las tareas pendientes.

**Modos de Operación**:
- `/dev.backlog` → Muestra el listado completo de items.
- `/dev.backlog add` → Inicia el flujo interactivo para registrar un nuevo elemento.
- `/dev.backlog add --audio` → Registra un elemento usando transcripción de voz.
- `/dev.backlog pick` → Selecciona un elemento existente y genera un feature a partir de él.
- `/dev.backlog resolve <ID>` → Cambia el estado de un elemento a resuelto.

---

## Resumen Rápido (Cheatsheet)

> Al usar `/dev.backlog help`, debes mostrar **exclusivamente** esta sección.

**Estructura**: `/dev.backlog [acción] [parámetros]`

| Modificador | Comportamiento |
|-------------|----------------|
| (vacío) | Lista todo el backlog ordenado por prioridad |
| `add` | Creación interactiva de item |
| `add --audio` | Creación mediante captura de audio |
| `pick` | Inicia una rama/feature desde el item |
| `resolve <ID>`| Marca un ID específico como completado |
| `--type <T>` | Filtra por TODO, DEBT o IDEA |
| `--priority <P>`| Filtra por nivel de prioridad |

---

## Archivo Central de Backlog

Todos los registros se almacenan globalmente en: `dev/backlog.md`

Existen 3 categorías soportadas:
1. **TODOs**: Tareas técnicas por hacer.
2. **DEBT**: Deuda técnica reconocida.
3. **IDEAS**: Sugerencias y mejoras a futuro.

---

## Comandos y Flujos Detallados

### 1. Listado del Backlog (`/dev.backlog`)
Al invocar sin parámetros, muestra el contenido de `dev/backlog.md` estructurado y ordenado por prioridad (Alta, Media, Baja).

Si el archivo está vacío o no existe, muestra un mensaje indicando que no hay elementos y sugiere usar `/dev.backlog add`.

### 2. Creación de Elementos (`/dev.backlog add`)
Inicia una secuencia interactiva solicitando:

Para **TODO** y **DEBT**:
- Título
- Prioridad (Alta/Media/Baja)
- Contexto (¿Por qué es necesario?)
- Archivos afectados (opcional)
- Complejidad (S/M/L/XL)
- *Para DEBT:* Riesgo si se ignora.

Para **IDEA**:
- Título, Prioridad, Contexto
- Impacto Potencial (UX, Rendimiento, etc.)
- Notas adicionales.

### 3. Selección y Conversión a Feature (`/dev.backlog pick`)
Muestra la lista de ítems numerada. El usuario selecciona uno y el agente:
1. Sugiere un nombre de feature (ej. `refactor-payment-validation`).
2. Crea el feature en `dev/wip/[feature-name]/`.
3. Pre-puebla el contexto inicial para `/dev.spec` (Problema, Archivos, Complejidad).
4. Actualiza `meta.md` con:
   ```yaml
   from_backlog: [ID]
   workflow_mode: full | technical-only | tasks-only
   auto_generated:
     functional: false | true
     technical: false | true
   ```
5. Marca el ítem en el backlog como `in-progress` apuntando a la ruta del feature.

**Modos de Flujo de Trabajo (Solo DEBT/TODO):**
> **Carga Perezosa (Lazy-loaded)**: Al procesar items DEBT o TODO, el agente DEBE leer `references/workflow-modes.md` para las reglas de modo de flujo de trabajo. Para IDEAS siempre se usa el flujo completo.

**Plantillas Auto-Generadas:**
> **Carga Perezosa (Lazy-loaded)**: Si se usan modos auto-generados (2 o 3), el agente DEBE leer `references/auto-spec-template.md`.

### 4. Resolución de Elementos (`/dev.backlog resolve <ID>`)
Permite marcar un elemento como completado. Pregunta interactivamente el tipo de resolución:
1. Completado (Implementado)
2. No se hará (Obsoleto)
3. Duplicado
Luego lo mueve a la sección "✅ Resolved Items" en `dev/backlog.md`.

---

## Captura Automática (Durante `/dev.build` o `/dev.fix`)

Si el agente nota una mejora obvia durante la implementación, debe pausar e invocar la herramienta de pregunta al usuario:

**⛔ INVOKE TOOL (do not print this, CALL the tool):**
```json
AskUserQuestion(
  questions=[{
    "question": "What would you like to do with this improvement?",
    "header": "Action",
    "options": [
      {"label": "Fix now", "description": "Address immediately in current task"},
      {"label": "Add as TODO", "description": "Track for later implementation"},
      {"label": "Add as DEBT", "description": "Document as technical debt"},
      {"label": "Skip", "description": "Not important, ignore"}
    ],
    "multiSelect": false
  }]
)
```

**Regla de Oro para "Fix Now"**: Solo ofrece "Fix now" como primera opción si el cambio es de baja complejidad (trivial), tiene bajo riesgo, y está directamente relacionado con la tarea actual sin causar *scope creep*.

Si elige "Fix now", aplícalo inmediatamente. Si elige "Add to backlog", guarda el ítem anotando la rama actual como el "Origen".

---

## Instrucciones y Restricciones Estrictas del Agente

1. **Gestión de Flags Especiales**:
   - `help`: Imprimir solo la tabla de resumen y salir. No ejecutar nada más.
   - `--audio` (con `add`): Invocar la skill `audio-capture`. Si hay transcripción, inferir el Tipo, Título y Contexto a partir de ella, pero continuar interactivamente para el resto. Si hay error, pedir ingreso manual.

2. **Formato de Archivo e IDs**:
   - Usar `dev/backlog.md` siempre. Si no existe, crearlo.
   - IDs únicos con formato `{TYPE}-{NNN}` (ej. TODO-001). Auto-incrementales basados en el máximo existente para ese tipo.
   - Mantener siempre el origen (Origin) de dónde surgió el ítem.

3. **Restricciones de Workflow para Auto-Specs (`pick`)**:
   - `IDEA` omite la pregunta de workflow mode y usa el pipeline completo.
   - Las especificaciones auto-generadas DEBEN tener `> **Auto-generated** from backlog item [ID]` al inicio.
   - **Funcional Auto-generado**: Usa formato ligero (problema + 1-2 historias + scope).
   - **Técnico Auto-generado (modo 3)**: REQUIERE exploración previa del código usando los subagentes `dev-explorer` y `dev-system-designer` en los archivos afectados.
   - `meta.md`: Debe actualizarse con `workflow_mode` y `auto_generated`.
   - Compatibilidad: Features sin `workflow_mode` en `meta.md` asumen `full` mode por defecto.

4. **Trazabilidad**: 
   - Las etapas auto-aprobadas deben marcar en `meta.md`: `approved_by: auto-generated` y `approved_at: [timestamp]`.
   - Los ítems del backlog nunca bloquean la ejecución de `/dev.finish`. Son meramente informativos.
