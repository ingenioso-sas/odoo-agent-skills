---
name: dev.build
description: Ejecuta y valida las tareas de implementación definidas en el plan del proyecto.
argument-hint: "[task-id|--next|--all|--resume]"
---

# Skill: Construcción e Implementación (/dev.build)

Este skill es responsable de la ejecución técnica de las tareas. Guía al agente en el proceso de escribir código, realizar pruebas y validar la calidad antes de marcar cada tarea como completada.

## Uso del Comando

- `/dev.build` → Ejecuta todas las tareas pendientes siguiendo la estrategia aprobada.
- `/dev.build task TASK-XXX` → Se enfoca exclusivamente en una tarea específica.
- `/dev.build --next` → Identifica y ejecuta la siguiente tarea pendiente en el orden lógico.
- `/dev.build --resume` → Reanuda la implementación desde el último punto guardado.

---

## Ciclo de Ejecución de Tarea

Cada tarea se procesa siguiendo un ciclo riguroso de calidad:

1. **Análisis**: El agente lee la especificación técnica y las decisiones de diseño relevantes para la tarea.
2. **Implementación**: Se escribe el código siguiendo principios de *Clean Code* y los estándares del proyecto.
3. **Pruebas (TDD/Unitarias)**: Se ejecutan o crean pruebas para validar la lógica. (MANDATORIO: No se permite saltar pruebas en proyectos de producción).
4. **Validación de Calidad (Gates)**:
   - **Compilación**: El código debe compilar sin errores.
   - **Linter**: Cumplimiento de las reglas de estilo del lenguaje.
   - **Seguridad**: Verificación de patrones inseguros (ej: secretos en código, inyección SQL).
5. **Persistencia**: Se actualiza el estado en `tasks.json` y se realiza un commit atómico.

---

## Puertas de Calidad (Quality Gates)

Antes de dar por terminada una tarea o una capa (layer), deben cumplirse las siguientes condiciones:

| Control | Descripción | Herramienta Común |
|---------|-------------|-------------------|
| **Build** | El proyecto debe construir/compilar correctamente. | `npm build`, `mvn compile`, `go build` |
| **Tests** | Todas las pruebas (unitarias e integración) deben estar en verde. | `npm test`, `pytest`, `go test` |
| **Lint** | El código debe seguir las guías de estilo. | `eslint`, `pylint`, `checkstyle` |
| **Seguridad** | Sin vulnerabilidades críticas o exposición de secretos. | Escaneo de estático básico |

---

## Comandos de Construcción por Tecnología

| Stack | Comando de Build | Comando de Test |
|-------|------------------|-----------------|
| **Node.js / TS** | `npm run build` | `npm test` |
| **Java (Maven)** | `mvn compile` | `mvn test` |
| **Java (Gradle)** | `./gradlew build` | `./gradlew test` |
| **Go** | `go build ./...` | `go test ./...` |
| **Python** | N/A | `pytest` |

---

## Gestión de Contexto e Interrupciones

La implementación de múltiples tareas puede agotar el contexto del modelo. El agente debe seguir estas directrices:

- **Avisos de Contexto**: Cuando el uso del contexto supera el 50%, el agente sugerirá realizar un `/clear` o compactar el estado.
- **Persistencia en Disco**: El estado de cada tarea se guarda inmediatamente en `tasks.json` para que el progreso sobreviva a reinicios de sesión.
- **Resumen de Progreso**: Al finalizar una capa (Layer), se muestra un resumen de lo logrado y lo pendiente.

---

## Manejo de Errores y Brechas en la Spec

- **Auto-Fix**: En caso de error de compilación o test, el agente intentará corregirlo automáticamente hasta 2 veces antes de pedir ayuda al usuario.
- **Detección de Brechas (Spec Gaps)**: Si durante la implementación se descubre que la especificación es ambigua o incompleta, el agente sugerirá usar `/dev.spec --iterate` para actualizar los requerimientos antes de continuar.

---

## Buenas Prácticas Profesionales

- **Atomicidad**: Un commit por tarea o por funcionalidad lógica pequeña.
- **Trazabilidad**: Incluir referencias a la especificación en los comentarios de código cuando sea necesario.
- **Simplicidad**: Seguir el principio KISS (*Keep It Simple, Stupid*). No añadir complejidad innecesaria fuera del alcance de la tarea.
- **Documentación**: Actualizar archivos README o JSDoc/Docstrings según corresponda.
