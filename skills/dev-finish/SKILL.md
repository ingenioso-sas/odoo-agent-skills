---
name: dev.finish
description: Finaliza la implementación del feature, ejecuta las últimas validaciones de calidad y seguridad, y archiva el proyecto. Se utiliza al culminar todas las tareas y pasar CI local, preparando el feature para pasar de wip/ a features/.
model: sonnet
---

### REGLAS DE EJECUCIÓN (AGENT INSTRUCTIONS)

> [!IMPORTANT]
> Si en algún punto encuentras:
> 
> ⛔ INVOKE TOOL (do not print this, CALL the tool):
> `AskUserQuestion(questions=[{...}])`
> 
> Significa que **DEBES EJECUTAR LA TOOL**. Está terminantemente prohibido imprimir el JSON en la terminal usando echo, cat, o usar preguntas de texto interactivo fuera del sistema de herramientas.

# Acción Principal: `/dev.finish`

**Propósito**: Someter a validación rigurosa un feature concluido y archivar su directorio.

**Opciones de Uso**:
- `/dev.finish` → Flujo completo de validación y archivo (según modo).
- `--force` → Omite validaciones secundarias.
- `--skip-tests` → (No recomendado) Salta la ejecución de tests.

**Pre-requisito Estricto**: La "Final Validation" de `/dev.build` debe haber resultado exitosa.

---

## Delegación Obligatoria (Validation Delegation)

> **CRÍTICO**: Es obligatorio utilizar las skills universales y los subagentes para realizar verificaciones profundas.

**Orden de Validaciones (Bloqueantes)**:
1. `Skill(skill="dev-validator")` → Compilación, tests, coverage, cumplimiento con gary.
2. `Skill(skill="dev-security-expert")` → Análisis de reglas de seguridad y detección de vulnerabilidades (ApplicationSecurityMCP).
3. `Skill(skill="dev-code-reviewer")` → Aprobación final de código (Bloqueante).
4. `Task(subagent_type="dev-layer-analyzer", ...)` → Verificación de consistencia entre Spec, Tareas y Código (exclusivo de Claude Code).

*Los pasos 1 al 3 deben pasar para poder archivar.*

---

## Verificación de Contexto (Context Advisory)

Si al invocar `/dev.finish` el contexto de la conversación excede el 60%, se recomienda usar `dev-layer-analyzer`. Si supera el 80%, debes proponer una compactación. No obstante, dado que `/dev.finish` requiere bajo uso de tokens (pues valida el trabajo ya hecho en `/dev.build`), se puede proceder.

---

## Extensibilidad con Skill Hooks

Soporta 3 puntos de anclaje (Extension points):
- `before-start` (Antes de validar fase)
- `after-implementation` (Tras pasar validaciones, antes de archivar)
- `before-approval` (Antes de pedir al usuario confirmación de archivado)

*(El agente resuelve hooks mediante `.claude/skill-hooks.json` y `~/.dev-sdd-kit/skill-hooks.json` fusionándolos con precedencia: user > repo > auto. Los `hook.mode == "required"` se invocan siempre; los `"available"` si son relevantes).*

---

## Flujos de Ejecución según Modo

### Express Mode
1. Autovalida todo.
2. Genera reportes (README, implementation summary).
3. Archiva la carpeta silenciosamente.

### Standard Mode
1. Corre todos los validadores y los presenta.
2. Extrae aprendizajes (Learnings) y sugiere enviarlos a PATTERNS.
3. Resuelve ítems de Backlog si están linkeados en `meta.md` o mencionados.
4. Requiere confirmación humana explícita antes de archivar.
5. Archiva y muestra resultado.

---

## Verificaciones Individuales Bloqueantes

### 0. Detección Determinística de Fase
Usa el script bash `detect-phase.sh` (obligatorio para ahorrar tokens).
```bash
# Verify implementation phase
phase_result=$(bash ~/.dev-sdd-kit/tools/detection/detect-phase.sh dev/wip/[feature] --json)
current_stage=$(echo "$phase_result" | grep -o '"stage":"[^"]*"' | cut -d'"' -f4)

# Stop si no es "implementation"
```

Detección Mobile:
```bash
stack_result=$(bash ~/.dev-sdd-kit/tools/detection/detect-stack.sh . --json 2>/dev/null)
platform=$(echo "$stack_result" | grep -o '"platform":"[^"]*"' | cut -d'"' -f4)
```

### 0.5 y 1. CI Validation & Estado de Tareas
- Mobile (iOS/Android): `./gradlew test` o `xcodebuild test`.
- Web/Backend: Ejecución de `gary-release-process-beta` skill.
- Todas las tareas en `tasks.json` deben figurar como `completed`. Usa script `validate-complete.sh`.

### 2. Cumplimiento de Plataforma & Seguridad (MANDATORY)
- **Mobile**: Corre `dev-validator` para assembleDebug/build nativo, tests, e inspección Andes/Everest.
- **Backend/Web**: Usa `validate-gary-compliance.sh`. (Requiere Dockerfile(s), /ping).
- **Seguridad**:
  - Delega en `dev-security-expert`.
  - Escaneo **crítico** de Secretos: prohibidos passwords/tokens harcodeados. Busca regex en código fuente.

### 3 y 4. Tests y Calidad del Código
- Coverage ≥ 80% (vía `validate-tests.sh`).
- Cero errores de linting / Typescript, y sin comentarios TODO sueltos.

### 5, 6 y 7. Análisis de Específicaciones (Spec Conflict Re-Scan)
- Usa `validate-spec-conflicts.sh` para detectar nuevas desviaciones tras `/dev.build`.
- Valida que todos los `<!-- overrides/extends/deprecates: path#section -->` apuntan a archivos existentes y secciones reales.
- Se actualiza `meta.md` con los `affected_specs`.
- Finalmente se invoca `/dev.check --sync`. Si el veredicto es `CANNOT_PROCEED`, no se puede archivar.

---

## Manejo de Proyectos Brownfield
Antes de archivar, detecta si es `brownfield`. Si lo es:
- Usa `validate-brownfield-merge.sh`.
- Ofrece al usuario fusionar los cambios de regreso a los specs generales del sistema (ej. en `dev/specs/`).

---

## Documentación Generada (Generación Final)

Antes de archivar, se generarán dentro de `wip/`:
- `README.md`: Resumen del feature, APIs.
- `implementation-summary.md`: Esfuerzo, commits, etc.

---

## Proceso de Archivado (CRITICAL WORKFLOW)

> **ADVERTENCIA DE INTEGRIDAD DE DATOS**: El orden de operaciones debe respetarse milimétricamente para evitar huérfanos.

**PASOS CORRECTOS**:
1. Limpiar temporales (`rm -rf dev/wip/[feature]/verdicts/`).
2. Generar documentación EN LA CARPETA WIP (README, validation report, etc).
3. Verificar archivos (ls -la de wip).
4. Mover LA CARPETA COMPLETA. Es decir:
   `mv dev/wip/[feature] dev/features/[feature]`
   > NO USAR `mv dev/wip/[feature]/*`. Fallará con archivos ocultos.
5. Chequear éxito (Verificar que el source no existe y el destino sí, y que `meta.md` se trasladó intacto).

---

## Extracción de Aprendizajes (PATTERNS.md)

Tras validar y antes de archivar, lee `4-implementation/progress.md`. Extrae descubrimientos técnicos.
Filtra duplicados (similitud >70%) contra los existentes en `dev/PATTERNS.md`.
Pregunta al usuario (vía `AskUserQuestion`) si desea promover los aprendizajes generalizables (que evitan repetir errores y no están en la documentación oficial). Si acepta, anexa a `PATTERNS.md` con timestamp.

---

## Pasos Interactivos Post-Archivado

Al finalizar todo, si el modo es Standard, debes sugerir continuar con `AskUserQuestion`:
Opciones recomendadas:
- `/dev.start`
- `/dev.start --reopen`
- `/dev.backlog list`
- `/dev.list`
