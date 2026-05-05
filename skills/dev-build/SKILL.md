---
name: dev.build
description: Ejecuta la implementación de tareas siguiendo la estrategia aprobada. Invocable cuando las tareas están en estado "approved" y se procede a la codificación. Maneja ejecución por capas, infraestructura, migraciones, builds de frontend y validaciones de CI.
model: opus
argument-hint: "[task-id|--next|--all]"
---

### NORMAS DE LECTURA DE ESTA SKILL

> [!IMPORTANT]
> Cuando encuentres un bloque similar a:
> 
> ⛔ INVOKE TOOL (do not print this, CALL the tool):
> `AskUserQuestion(questions=[{...}])`
> 
> **DEBES invocar la herramienta**. No imprimas el JSON en la consola. No uses bash para emular la interacción (prohibido usar echo, cat, printf).

---
hooks:
  TaskCompleted:
    - hooks:
        - type: command
          command: "~/.dev-sdd-kit/tools/shared/check-quality-task.sh"
---

# Módulo Principal: `/dev.build`

**Función**: Llevar a cabo la implementación de las tareas (features) según el plan pre-aprobado.

**Sintaxis Básica**:
- `/dev.build` → Inicia la implementación de tareas pendientes basándose en el modo actual.
- `/dev.build task TASK-XXX` → Implementa de forma aislada una tarea específica.
- `/dev.build phase N` → Ejecuta exclusivamente una fase particular.
- `/dev.build --layer N` → Llega hasta la capa N.
- `/dev.build --resume` → Retoma una sesión que fue interrumpida.
- `/dev.build --next` → Auto-continua con el siguiente ítem pendiente.

---

## Atajos de Ayuda (Cheatsheet)

> Al invocar `/dev.build help`, muestra únicamente este bloque.

**Comando**: `/dev.build [objetivo] [opciones]`

| Opciones | Comportamiento |
|----------|----------------|
| (sin flag)| Implementación secuencial de todo lo pendiente |
| `task ID` | Procesa una única tarea (ej. TASK-005) |
| `phase N` | Corre una fase específica |
| `--layer N` | Procesa hasta concluir la capa indicada |
| `--resume`| Retoma tras una interrupción |
| `--next` | Salta a la siguiente tarea lista |

---

## Integración con Plan Mode (Opcional)

> **CRÍTICO**: Plan Mode de Claude Code se utiliza para tareas complejas pero está **deshabilitado por defecto**.

El sistema de planificación previa está controlado por `PROJECT.md` o `~/.dev-sdd-kit/config.yaml`:
```yaml
plan_mode:
  build_complex_tasks: false
  build_layer_transitions: false
  build_gary_test_recovery: false
```

**Condiciones de Disparo (si está activado)**:
- **Tareas complejas**: Complejidad Alta, más de 5 archivos, Capa 2, o más de 4 criterios de aceptación.
- **Transiciones de Capa**: Pasar de Capa N a N+1 con contexto > 50% y más de 10 tareas.
- **Recuperación CI**: Primer fallo ambiguo, confianza < 70%, o más de 2 intentos de arreglo fallidos.

**Flujo en Entornos sin Claude Code (Fallback)**:
Si la tool `EnterPlanMode` no existe, el agente explora el código (modo solo lectura), diseña el plan, lo muestra en chat, e invoca `AskUserQuestion` con opciones: "Approve", "Modify", "Cancel". Tras aprobar, se ejecuta.

---

## Controles de Calidad (BLOQUEANTES)

> **Obligatorio**: Después de CADA tarea, no solo al final.

**Ciclo Iterativo por Tarea**:
1. Implementar (Código de producción)
2. Testear (Tests unitarios/integración, omitir si es prototipo)
3. Calidad (Delegación):
   ```python
   Task(
       subagent_type="dev-validator-runner",
       prompt="""
       Validate files: [modified_files]
       Run Layer 3 quality gates: performance, security, code-review
       Return unified JSON verdict.
       """
   )
   ```
4. **Fix**: Corregir TODOS los hallazgos (Críticos, Mayores y **Menores**). Los menores no son opcionales.
5. Re-check: Volver a validar hasta tener cero problemas.
6. Finalizar: Marcar done, hacer commit.

> Los veredictos se guardan en `dev/wip/<feature>/verdicts/` pero no se comitean.

### Integración ApplicationSecurityMCP

**Antes de añadir librerías (Escaneo de Dependencias)**:
Llamar a `mcp__ApplicationSecurityMCP__safe_add_dependency(technology, ecosystem, name_user, name_repository, dependencies)`. Si hay vulnerabilidad, intentar actualizar. Si persiste, advertir y bloquear.

**Búsqueda de Toolkits (Condicional)**:
Solo si `dev-security-expert` o sugerencias de corrección recomiendan un SDK interno: invocar `mcp__ApplicationSecurityMCP__search_security_toolkits(...)`.

---

## Sistema de Hooks Extensibles

Resolución en 3 capas: sobrescrituras de usuario (`~/.dev-sdd-kit/skill-hooks.json`) > configuración del repositorio > auto-declaración (de las skills).
Se ejecutan según: `hook.mode == "required"` (siempre) o `"available"` (solo si es relevante).

| Disparador (Trigger) | Momento |
|----------------------|---------|
| `before-start` | Antes del paso 1 |
| `after-implementation`| Después del paso 5 |
| `before-approval` | Antes de finalizar y preguntar próximos pasos |

---

## Pasos de Ejecución (Flujo Secuencial)

### [Punto Extensión: before-start]

### Paso 1: Detección Determinística de Fase

Se DEBE usar bash para detección:
```bash
phase_result=$(bash ~/.dev-sdd-kit/tools/detection/detect-phase.sh dev/wip/[feature] --json)
current_stage=$(echo "$phase_result" | grep -o '"stage":"[^"]*"' | cut -d'"' -f4)

if [ "$current_stage" != "implementation" ]; then
    echo "❌ Tasks not approved. Run /dev.plan --approve first."
    exit 1
fi

stack_result=$(bash ~/.dev-sdd-kit/tools/detection/detect-stack.sh . --json 2>/dev/null)
platform=$(echo "$stack_result" | grep -o '"platform":"[^"]*"' | cut -d'"' -f4)

if [ "$platform" = "android" ] || [ "$platform" = "ios" ]; then
    # Verificar si el plugin está presente... (ver referencia original para paths de claude/plugins)
    # Si no está, abortar con error.
fi
```
Si el contexto es >80%, se requiere compactación obligatoria vía `context-guardian`.

### Paso 2: Lectura de Tareas
Leer tareas desde `dev/wip/[feature]/3-tasks/tasks.json` vía `jq`.

### Paso 3: Ejecución en Capas (Layer-Based Execution)

- **Capa 1 (Local)**: Ejecución en paralelo si hay tareas independientes (sin archivos compartidos ni dependencias de datos). Se usarán instancias aisladas (ej. `dev-android-implementer` en worktrees separados). Tras finalizar, se fusionan los worktrees, se compila y se hace commit.
- **Capa 2 (Integración gary)**: Secuencial por defecto (tienen efectos secundarios). Validación CI local vía RP MCP. Commit.
- **Capa 3 (Calidad)**: Revisiones profundas y completas. Commit.

**Aviso de Contexto entre Capas (Context Advisory)**:
Si al terminar una capa el contexto supera 50% (70% recomendado, >80% obligatorio), sugerir compactación o `/clear`. Se muestra un aviso y (en modo Standard) se invoca `AskUserQuestion` para que el usuario elija hacer `/clear + --resume`, seguir, o `/dev.check --compact`.

### Paso 3.3: Creación de Infraestructura (Condicional)
Se procesan todas las `INFRA-TASK-*`. Requiere `gary CLI` instalado y sesión iniciada (`gary login`).
Flujo: Ejecutar comando de creación -> Validar con comando list -> Marcar completado. (Idempotente).

### Paso 3.5: Rama de Migración (Condicional)
Si `migration.detected == true`, el agente lee `references/database-migration.md`.

### Paso 4: Implementación por Tarea

Delegación según tipo/plataforma:
- Web/Backend: `dev-implementer`. Envía Decisiones de Diseño extraídas del Technical Spec (no re-proponer alternativas).
- Android: `dev-android-implementer`.
- iOS: `dev-ios-implementer`.
- Tests Pequeños: `dev-small-test-writer`.
- Tests E2E: solo si `ltp_enabled` usando `dev-large-test-writer`.

**Preámbulo Mobile Obligatorio**:
Para Android/iOS, el subagente DEBE resolver `skill_root`, ejecutar `ls`, y usar el contenido de `SKILL.md` para navegar y procesar la documentación de Everest/Andes ANTES de codificar.

### Paso 5: Persistencia de Estado y Contexto
Al terminar una tarea:
1. Validar usando `dev-validator-runner`.
2. Actualizar estado persistente en disco (obligatorio para resistir `/clear`):
   ```bash
   jq '(.tasks[] | select(.id == "TASK-XXX")) .status = "completed"' ...
   ```
3. Chequeo intermedio de contexto: Si excede límites, hacer commit intermedio y sugerir `/clear`.

### [Punto Extensión: after-implementation]

### Paso 6: Validación Final de Integración

1. **Cumplimiento gary (3 Capas de validación)** (Salvo en Android/iOS, donde se compila nativamente con gradle o xcode).
   - *Capa 1*: Dockerfiles, /ping, versión.
   - *Capa 2*: Expertos de SDK (Java, Go, Python, Node).
     - **Frontend (Nordic/Andes)**: Uso correcto de tags (`nordic/image` no `<img>`), design tokens, linting JS/CSS.
     - **Figma (design-to-code)**: Si hay un hash de Figma y es Frontend, forzar uso de la versión correcta de Andes (8, 9 o X) y verificar que los imports mapean con el technical spec.
     - **Arquitectura de Servicios**: Validación de KVS, BigQueue, Secrets (prohibido URLs en secrets, se considerará CRÍTICO).
   - *Capa 3*: Ejecución CI Pipeline.
2. **Layer 3 Quality Gates**: Invocación final de `dev-validator-runner`.
3. **Patrones Anti-Calidad**: `validate-code.sh` (Evitar inyecciones SQL, queries N+1).
4. **CI Local (RP MCP)**: Comprueba si `gary-release-process-beta` existe. Si no, aborta pidiendo su instalación vía `claude plugin install`. Si existe, se invoca a través de `Skill("gary-release-process-beta")`.

### Paso 7: Validación Sync
Comprobar desviación de las especificaciones: `/dev.check --sync`.
Si hay gaps (ej. campos faltantes, tests que exigen comportamientos no documentados), advertir y sugerir `/dev.spec --iterate` o guardarlo en `/dev.backlog`.

### [Punto Extensión: before-approval]

### Paso 8: Transición Interactiva Final
Tras completar todo, si en modo Standard: invocar `AskUserQuestion` recomendando `/dev.finish`.
En modo Express: saltar a `/dev.finish` automáticamente.

---

## Soporte y Lenguajes (Java/Jakarta)

**Regla de Imports en Java**:
Antes de generar código Java, analizar importaciones existentes:
`grep -r "import javax\.servlet" src/` vs `jakarta\.servlet`.
Si usa jakarta, seguir usándolo. Si usa javax, continuar con javax. Si no hay nada, usar **jakarta** por defecto.
NUNCA mezclar javax y jakarta en el mismo proyecto.

## Seguimiento y Artefactos

- **Progreso**: Mantener `dev/wip/[feature]/4-implementation/progress.md` actualizado (status y hash de commit).
- **Proyectos (Tipo)**:
  - *Prototipos*: sin test unitarios ni coverage, revisión de código obligatoria.
  - *Producción*: full testing (>80%), coverage, y revisión.
- **Mejoras descubiertas**: Si no son triviales, capturar con `/dev.backlog`.
