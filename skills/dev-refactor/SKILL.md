---
name: dev.refactor
description: Habilidad especializada en mitigar la deuda técnica y mejorar la calidad interna del código. Garantiza cero impacto en la funcionalidad de negocio mediante estrategias de línea base y verificación bidireccional.
argument-hint: "[archivo-o-directorio]"
---

### REGLAS ESTRICTAS DE LECTURA E INTERACCIÓN

> [!IMPORTANT]
> Si en algún momento requieres que el usuario apruebe una estrategia de refactorización o escoja un patrón, debes utilizar la herramienta `AskUserQuestion(questions=[{...}])`.
> NUNCA imprimas el JSON como texto, ni pidas por chat abierto que te escriban la respuesta.

# Comando: `/dev.refactor`

**Misión**: Mejorar la mantenibilidad, legibilidad y estructura del código, garantizando mediante validaciones cruzadas que el comportamiento externo (los tests y contratos) permanece 100% inalterado.

**Uso**:
- `/dev.refactor [path_a_archivo]` → Analiza y refactoriza un archivo específico.
- `/dev.refactor [path_a_directorio]` → Escanea el directorio buscando code smells y propone refactorizaciones.

---

## Restricción de Dominio (Boundary Rules)

1. **PROHIBIDO MODIFICAR FUNCIONALIDAD**: `/dev.refactor` tiene ESTRICTAMENTE PROHIBIDO alterar la lógica de negocio. Si detectas un bug durante la refactorización, NO lo arregles; deriva a `/dev.fix`.
2. **ESPECIFICACIONES INMUTABLES**: No puedes bajo ningún concepto modificar los archivos `1-functional/spec.md`, `2-technical/spec.md`, ni `tasks.json`. El refactor es un proceso técnico subyacente.
3. **MICRO-COMMITS**: La refactorización debe dividirse en los pasos más pequeños y atómicos posibles.

---

## Flujo de Trabajo (Refactoring Loop)

### Fase 1: Línea Base de Estabilidad (Pre-Flight Check)

> **Regla de Oro**: Jamás refactorices código roto.

Antes de tocar una sola línea de código, el agente DEBE ejecutar los tests asociados al componente objetivo.
```bash
# Detectar el framework y ejecutar pruebas (usa bash tools del SDD Kit)
test_status=$(bash ~/.dev-sdd-kit/tools/tests/run-target-test.sh [path] --json)
```

- Si los tests **Pasan** (Verde) → Proceder a Fase 2.
- Si los tests **Fallan** (Rojo) → DETENER FLUJO. Imprime alerta: "El código actual no es estable. Llama a `/dev.fix` antes de intentar refactorizar."

### Fase 2: Análisis Estático y Code Smells

Delega el análisis estructural al subagente experto en arquitectura (o escáner de deuda).
Evalúa:
1. **Complejidad Ciclomática**: ¿Hay condicionales anidados excesivos (`if/else/switch`)?
2. **Duplicación (DRY)**: ¿Existen bloques lógicos repetidos?
3. **Acoplamiento**: ¿El módulo expone dependencias que deberían inyectarse o aislarse?
4. **Legibilidad**: Nombramiento confuso (variables como `d`, `tmp1`, clases `Manager`).

Presenta los hallazgos en formato de tabla (NO interactiva) como resumen visual:
```markdown
| Archivo | Olor Detectado | Solución Propuesta (Patrón) | Complejidad |
|---------|----------------|-----------------------------|-------------|
| utils.go| Condicionales anidados| Extract Method / Guard Clauses | Baja |
```

A continuación, utiliza `AskUserQuestion` para solicitar confirmación de proceder con el plan propuesto o ajustar el enfoque.

### Fase 3: Ejecución Incremental (Micro-Refactoring)

Aplica el patrón seleccionado modificando el código.
Al hacerlo, aplica la estrategia de Micro-Pasos:
1. Aplica un cambio (ej. Renombrar variable).
2. Verifica si el proyecto compila.
3. Aplica otro cambio (ej. Extraer método).

*Nota de Seguridad*: Siempre asegúrate de que estás leyendo las librerías permitidas si estás en un proyecto con capa de infraestructura (ej. `dev-gary-discovery` si es un componente de gary).

### Fase 4: Verificación Bidireccional de Consistencia

Al terminar los cambios, debes demostrar matemáticamente que la funcionalidad es idéntica a la Línea Base.

1. **Re-ejecución Local**: Corre los tests específicos del componente.
   - *Si falla*: Revertir automáticamente la edición del paso actual y notificar al usuario.
2. **Propagación Horizontal (Igual que `/dev.fix`)**:
   Asegúrate de que los cambios de nombres de variables o métodos se hayan propagado a todos los lugares donde se consumen. Usa `grep_search` o la herramienta equivalente para encontrar referencias huérfanas en el proyecto antes de dar por terminado el refactor.

### Fase 5: Consolidación (Commit Automático)

Si todo está en "Verde" y la verificación cruzada es exitosa, prepara un mensaje explicativo y ofrece integrarlo:
```markdown
# ♻️ Refactorización Completada
- Reducción de complejidad ciclomática de 8 a 3.
- Extracción de método `calcularDescuento()`.
- Tests verificados (100% pasando).
```
Sugiere un comando de commit o realízalo usando herramientas bash seguras del kit si estás autorizado por el perfil del usuario.
