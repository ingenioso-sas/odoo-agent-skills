---
name: dev.plan
description: Transforma especificaciones aprobadas en un listado de tareas ejecutables. Se utiliza una vez que los Functional y Technical Specs tienen el visto bueno y se requiere desglosar el trabajo en tareas con estimaciones y asignación por capas.
model: opus
argument-hint: "[--approve]"
---

### REGLAS DE EJECUCIÓN (INSTRUCCIONES PARA EL AGENTE)

> [!IMPORTANT]
> A lo largo de esta skill encontrarás bloques como este:
> 
> ⛔ INVOKE TOOL (do not print this, CALL the tool):
> `AskUserQuestion(questions=[{...}])`
> 
> Esta es una llamada a una herramienta (TOOL CALL). NO uses `echo`, `cat` ni `printf`. LLAMA directamente a la herramienta.

# Acción Principal: `/dev.plan`

**Objetivo**: Generar, refinar y dar por aprobadas las tareas de implementación.

**Opciones de Invocación**:
- `/dev.plan` → Comportamiento automático según el perfil/modo.
- `/dev.plan --refine` → Refina/modifica tareas preexistentes.
- `/dev.plan --approve` → Confirma tareas y escoge la estrategia de ejecución.
- `/dev.plan --resume` → Continúa una sesión de planeación en pausa.
- `/dev.plan --view` → Abre el visor HTML interactivo de las tareas.

---

## Verificaciones Previas (BLOQUEANTES)

| Condición | Herramienta | Acción si Falla |
|-----------|-------------|-----------------|
| Conflictos resueltos | `validate-spec-conflicts.sh` | Deriva a `/dev.spec` |
| Technical Spec aprobado | Leer `meta.md` | Deriva a `/dev.spec technical` |
| Contexto < 50% | Skill `context-guardian` | Aviso / Recomendación de compactar |

---

## Skill Hooks (Extensibilidad)

`/dev.plan` tiene 3 puntos de anclaje que se resuelven leyendo `skill-hooks.json`:
- `before-start` (Antes de validar fase)
- `after-implementation` (Tras la generación del listado de tareas)
- `before-approval` (Antes de pedir aprobación)

---

## Flujo de Trabajo Secuencial

### Paso 1: Detección de Fase y Chequeo de Contexto
Se DEBE usar la herramienta en bash:
```bash
phase_result=$(bash ~/.dev-sdd-kit/tools/detection/detect-phase.sh dev/wip/[feature] --json)
current_stage=$(echo "$phase_result" | grep -o '"stage":"[^"]*"' | cut -d'"' -f4)

# Si current_stage no es "tasks" ni "implementation", abortar.
```
Luego invoca `context-guardian`. Si el contexto es mayor a 70%, sugiere compactar.

### Paso 2: Validación Anti-Conflictos
Ejecuta `validate-spec-conflicts.sh`. Si hay desviaciones entre specs, detén la ejecución e indica usar `/dev.spec`.

### Paso 3: Lectura de Especificaciones (Inputs)
Lee los archivos `1-functional/spec.md` y `2-technical/spec.md`.
Verifica `meta.md`:
- Si ambos están auto-generados (`auto_generated.functional == true` y `technical == true`), apóyate pesadamente en el backlog.
- Si solo el funcional es auto-generado, usa el Technical Spec como principal fuente de verdad.

### Paso 4: Detección de Entorno y Servicios
*Se omite si la plataforma es iOS o Android.*
Si detectas bases de datos relacionales en el spec (backend/web):
- Si el perfil es "technical": Pregunta al usuario si desea crear un contenedor local, testcontainers o usar uno existente.
- Si el perfil es "non-technical": Selecciona autómaticamente la opción de contenedor local sin preguntar.

### Paso 5: Generación Estricta de Tareas

Las tareas se escriben **exclusivamente en formato JSON** en `3-tasks/tasks.json`. (NO crear `tasks.md`).
Para cada tarea, mapea las Decisiones de Diseño (`design_decisions: ["DD-1", ...]`) del Technical Spec para que el subagente de implementación no invente alternativas.

**Validación Mobile vs Web/Backend**:
- Si `platform = android | ios`: Las tareas NUNCA deben mencionar Dockerfile, /ping o gary compliance. Deben usar `./gradlew test` o `xcodebuild test`. Las librerías Everest listadas en la sección 3 del Technical Spec DEBEN usarse textualmente en las tareas.
- Si `platform = backend | web | ""`: Tareas obligatorias incluyen crear Dockerfile(s), endpoint /ping y validación de cumplimiento de plataforma gary.

**Regla Anti-Despliegue**: PROHIBIDO generar tareas de despliegue ("deploy", "gary deploy", "push to prod"). El agente solo codifica y testea.

**Migraciones de DB**: Si hay cambios de esquema, SIEMPRE usar `gary migration init`. Prohibido usar Flyway, Liquibase o crear .sql manualmente.

**Extracción de Escenarios E2E (Offloaded)**:
Ejecuta `genai-analyze-e2e.sh` para detectar escenarios E2E y su grafo de dependencias.
Si `total_scenarios > 0` y `ltp.enabled == true`, crea la tarea obligatoria `AUTO-TASK-E2E`.

### Paso 6: Selección de Estrategia
- Perfil `non-technical`: Auto-selecciona "Batched".
- Perfil `technical`: Si hay ≤5 tareas y todas de baja complejidad, auto-selecciona "Sequential". Si es más grande, usa `AskUserQuestion` para ofrecer "Sequential", "Batched" (Recomendado) o "Parallel".

### Paso 7: Aprobación y Presentación Visual

**ANTES** de pedir confirmación, asegúrate de correr el script de previsualización:
```bash
bash ~/.dev-sdd-kit/tools/state/display-tasks.sh dev/wip/[feature]/3-tasks/tasks.json
```
Luego muestra la tabla de resumen y por último invoca `AskUserQuestion`:
```json
AskUserQuestion(
  questions=[{
    "question": "Approve these tasks?",
    "header": "Tasks",
    "options": [
      {"label": "Yes, approve", "description": "Approve tasks and continue"},
      {"label": "Adjust tasks", "description": "Modify task list before approving"},
      {"label": "Cancel", "description": "Cancel task generation"}
    ],
    "multiSelect": false
  }]
)
```

### Paso 8 y 9: Context Advisory Post-Plan y Siguiente Paso
Si el contexto superó 50% al finalizar, recomienda hacer `/clear` + `/dev.build`.
Usa `AskUserQuestion` para que el usuario elija su siguiente acción (normalmente `/clear + /dev.build`).

---

## Formato del Archivo `tasks.json`

Debe incluir la metadata, la configuración local, estadísticas, la lista de tareas y el grafo de dependencias:
```json
{
  "feature": "nombre",
  "local_config": { "mysql": "container" },
  "stats": { "total": 2 },
  "tasks": [
    {
      "id": "TASK-001",
      "title": "Config",
      "description": "Corto.",
      "status": "pending",
      "layer": 1,
      "depends_on": [],
      "files": [],
      "acceptance_criteria": ["GATE: test passes"],
      "references": ["US-1"],
      "design_decisions": ["DD-1"]
    }
  ]
}
```

> **Generación de IDs**: Usa obligatoriamente `generate-ids.sh` para obtener IDs secuenciales y únicos de la forma `TASK-NNN`.

## Capas (Layers)
- **Capa 1**: Código local, unit tests.
- **Capa 2**: Dependencias gary, colas, bases de datos remotas.
- **Capa 3**: Revisiones de Calidad. DEBE incluir exactamente: `dev-code-reviewer`, `dev-performance-expert`, y `dev-security-expert`. (Para el de seguridad, invocar ApplicationSecurityMCP).

## Funcionalidad `--view`
Llama a `view-tasks.sh` apuntando al `tasks.json` actual para abrir una interfaz en HTML interactiva.

## Refinamiento de Tareas (`--refine`)
Permite dividir, borrar, o modificar la complejidad/prioridad de tareas vía diálogo interactivo con el usuario.
