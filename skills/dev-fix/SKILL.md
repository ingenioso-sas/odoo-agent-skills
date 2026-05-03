---
name: dev.fix
description: Diagnóstico, reproducción y corrección de errores (bugs) técnicos.
argument-hint: "[descripción-error|--log-path]"
---

# Skill: Corrección de Errores (/dev.fix)

Este skill proporciona un flujo estructurado y científico para resolver fallos técnicos. Se basa en el principio de "reproducción primero", asegurando que cada corrección esté validada por una prueba que anteriormente fallaba.

## Uso del Comando

- `/dev.fix "descripción del error"` → Inicia el proceso de diagnóstico basado en una descripción.
- `/dev.fix --log-path path/to/log` → Analiza un archivo de log para identificar la causa raíz.

---

## Flujo de Trabajo Científico

### 1. Diagnóstico y Triaje
El agente analiza la evidencia disponible (logs, stack traces, reportes de usuario) para localizar el componente afectado.
- **Acción**: Explorar el código relacionado y entender el flujo que genera el fallo.

### 2. Reproducción (MANDATORIO)
Antes de modificar el código, el agente **debe** demostrar la existencia del error.
- **Acción**: Crear un caso de prueba (unitario o integración) que falle exactamente debido al bug reportado.
- **Validación**: Ejecutar el test y confirmar que está en "Rojo".

### 3. Implementación de la Solución
Desarrollar la corrección técnica mínima necesaria para resolver el problema.
- **Acción**: Modificar el código siguiendo los estándares del proyecto.
- **Regla**: No aprovechar para añadir funcionalidades nuevas o refactorizaciones masivas fuera del alcance del bug.

### 4. Verificación y Regresiones
Asegurar que la solución es robusta y no rompe otras partes del sistema.
- **Acción**: Ejecutar el test de reproducción (ahora debe estar en "Verde") y la suite completa de pruebas del proyecto.
- **Puertas de Calidad**: Validar compilación, linter y seguridad básica.

### 5. Resolución y Cierre
Documentar lo aprendido para evitar que el error se repita.
- **Acción**: 
  - Explicar la causa raíz y la solución en el mensaje de commit.
  - Decidir si el test de reproducción debe integrarse permanentemente a la suite de pruebas (recomendado para evitar regresiones).
  - Limpiar artefactos temporales de debugging.

---

## Reglas de Oro

- **Sin Test, No hay Fix**: Nunca asumas que el error está corregido si no tienes un test que lo demuestre.
- **Causa Raíz, No Síntoma**: Busca solucionar el origen del problema, no solo "parchear" la consecuencia inmediata.
- **Atomicidad**: La corrección de un bug debe ser un cambio atómico y fácil de revertir si es necesario.
- **Documentación de Lecciones**: Si el bug fue complejo, sugiere añadir el patrón de prevención a `PATTERNS.md`.

---

## Comandos Útiles para Debugging

- **Logs**: `tail -f`, `grep`, `awk`.
- **Tests**: Comandos de ejecución filtrados por el test de reproducción (ej: `npm test -- -t "BugReproduction"`).
- **Inspección**: Uso de herramientas de profiling o logs detallados en modo debug.
