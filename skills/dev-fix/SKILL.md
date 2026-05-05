---
name: dev.fix
description: Corrige errores de validación e inconsistencias horizontales entre las capas de especificación. Utilízalo ante fallos en pruebas o cuando /dev.check detecte discrepancias entre el código, las tareas y las specs.
model: opus
argument-hint: "[feature-name]"
---

### GUÍA DE LECTURA E INTERACCIÓN

> [!IMPORTANT]
> Obligación Estricta: Si observas el siguiente texto:
> 
> ⛔ INVOKE TOOL (do not print this, CALL the tool):
> `AskUserQuestion(questions=[{...}])`
> 
> DEBES llamar directamente a la herramienta. Nunca uses `echo` o imprimas el JSON en la respuesta de texto.

# Módulo Principal: `/dev.fix`

**Misión**: Resolver errores analizando su salida y propagar las correcciones transversalmente por TODOS los artefactos (especificaciones, tareas, código) para salvaguardar la coherencia del feature.

---

## Modos de Uso (Sintaxis)

- `/dev.fix` → Inicia de forma interactiva pidiendo que pegues el error.
- `/dev.fix "mensaje"` → Le pasa el error directamente.
- `/dev.fix --file <ruta>` → Lee el error desde un archivo (ej. un log).
- `/dev.fix --audio` → Permite describir el error por voz.
- `--layer <N>` → Restringe el fix a una capa (PELIGROSO).
- `--auto` → Aplica arreglos automáticamente sin pedir permiso.

---

## Árbol de Decisión: ¿Cuándo Usar `/dev.fix`?

> **Regla de Oro**: Se usa ante errores reales (ej. crash, tests fallando) que sugieren un "hueco" en la especificación, no para corregir un simple punto y coma.

| Escenario | Tool Correcta |
|-----------|---------------|
| ¿Es una sospecha de inconsistencia sin error de test? | Usa `/dev.check --sync` |
| ¿Es un error puro de código (typo, sintaxis)? | Arréglalo manualmente |
| ¿Test falla porque la API pide un dato no documentado? | **Usa `/dev.fix`** |

---

## Principio Básico: Consistencia Horizontal Estricta

`/dev.fix` NO es un parche de código. Es una actualización atómica de todo el sistema. Si cambias el código para corregir un error funcional, DEBES actualizar las Tareas, el Technical Spec, y el Functional Spec (hacia atrás).

> **Aviso de Fase (v1.6.0)**: Sólo se pueden reparar capas que ya existan en la fase actual.

| Término | Significado | Momento |
|---------|-------------|---------|
| **Propagación Horizontal** | Actualizar capas afectadas en cadena | Al aplicar arreglos |
| **Consistencia Bidireccional**| Chequear que todo el código esté documentado y toda la doc implementada | Validación final |

---

## Subagentes

Para bugs complejos, delega la investigación:
```python
Task(subagent_type="dev-debugger", prompt="Analizar: [detalles del error]")
```

---

## Flujo del Comando (El Motor de `/dev.fix`)

### Paso 0: Detección de Fase (OBLIGATORIO)

> No puedes proponer cambios a código si estás en fase "functional".
Lee `meta.md` para saber en qué etapa estás y qué capas están disponibles.

### Paso 1: Recepción del Error
Pide el error (vía parámetro, portapapeles, archivo o audio).

### Paso 1.5: Clasificación (OBLIGATORIO)
Clasifica el problema antes de analizar: `FEATURE_GAP`, `DESIGN_FLAW`, `MISSING_TASK` o `IMPL_BUG`. Esto rige qué capas van a actualizarse obligatoriamente.

### Paso 2: Análisis (Plan Mode)
El "Plan Mode" está activo por defecto (`fix_complex_bugs: true`) para bugs serios. Encuentra la causa raíz.

### Paso 3: Análisis de Impacto
Verifica qué capas sufren el impacto de este bug.

### Paso 3.5: El Protocolo Anti-Atajos (OBLIGATORIO)

> **CRÍTICO**: Está prohibido declarar "Status: No Change" en una capa sin pruebas.

Por cada capa en la que decidas "No hay cambios", debes proveer evidencia textual de los specs:

```markdown
### 📋 Functional Spec
**Status**: No Change
**Evidence**: La sección "Manejo de Errores" ya lo dice:
> "El sistema muestra un error amigable ante emails inválidos"
```

**Banderas Rojas**: Si el fix añade parámetros de API, respuestas nuevas, comportamientos visibles, NO puedes usar "No Change".

### Paso 4: Propuesta Horizontal
Muestra los diffs propuestos para los 4 niveles (Specs y Código) y pide aprobación interactiva.

### Paso 5: Ejecución Horizontal
Aplica las modificaciones a los archivos, manteniendo su sincronía.

### Paso 6 y 6.5: Tests y Code Review (BLOQUEANTES)
1. Corre los tests para confirmar.
2. Llama obligatoriamente al `CodeReviewerMCP` en los archivos tocados. Resuelve **todos** los hallazgos (incluso menores).

### Paso 7: Validación Bidireccional
Ejecuta `/dev.check --sync`.
Si reporta `CANNOT_PROCEED` (inconsistencias entre specs y código), debes arreglarlas de inmediato y reintentar.

---

## Patrones Comunes de Falla a Evitar (Antipatrones de Agentes)

- ❌ **"El test pasó, así que los specs están bien"**.
  *Por qué está mal*: El fix de código pudo introducir una validación que no estaba documentada. Los specs quedaron obsoletos.
- ❌ **"Como solo toqué código, es un IMPL_BUG"**.
  *Por qué está mal*: Si agregaste manejo de errores, es al menos un `MISSING_TASK` o `DESIGN_FLAW`.
- ❌ **Usar flags `--code-only` o `--layer` a la ligera**.
  *Por qué está mal*: Causan desviación estructural de specs (Spec Drift). Solo úsalas si el fix es un typo cosmético sin impacto funcional.

---

## Instrucciones del Agente

1. **Audio Catch**: Si se usa `--audio`, llama a `Skill("audio-capture")`. Pasa el resultado como descripción de error.
2. **Help Flag**: Solo imprime el resumen.
3. **Clasificación Primero**: NUNCA empieces a arreglar sin determinar el tipo de error.
4. **Verificación Anti-Atajo**: SIEMPRE añade el bloque `Evidence` si no alteras una capa.
5. **Completitud del Fix**: El fix NO termina hasta que el CodeReviewerMCP dé cero hallazgos y el chequeo bidireccional confirme que no hay discrepancias.

> El progreso se enviará a la telemetría automáticamente.
