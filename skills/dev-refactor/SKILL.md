---
name: dev.refactor
description: Mejora de la estructura y legibilidad del código sin alterar su comportamiento funcional.
argument-hint: "[archivo-o-directorio]"
---

# Skill: Refactorización Estructural (/dev.refactor)

Este skill está diseñado para combatir la deuda técnica. Permite mejorar la calidad interna del código asegurando que la funcionalidad externa permanezca idéntica a través de un proceso riguroso de verificación.

## Uso del Comando

- `/dev.refactor [path]` → Inicia un análisis y propuesta de refactorización para un archivo o carpeta específica.

---

## Flujo de Trabajo de Alta Confianza

### 1. Línea Base (Baseline)
Antes de tocar el código, el agente **debe** verificar que el estado actual es estable.
- **Acción**: Ejecutar las pruebas existentes relacionadas con el código a refactorizar.
- **Regla**: Si las pruebas fallan inicialmente, el agente debe derivar a `/dev.fix` antes de intentar refactorizar.

### 2. Análisis de Deuda
Identificación de "olores de código" (code smells) y oportunidades de mejora:
- **Puntos a evaluar**: Complejidad ciclomática, duplicación de código, nombres poco claros, acoplamiento excesivo.

### 3. Implementación (Paso a Paso)
Aplicar patrones de refactorización de forma incremental.
- **Patrones Comunes**: Extraer método, renombrar variables, simplificar condicionales, reemplazar constantes mágicas.
- **Regla**: Prohibido añadir lógica de negocio o cambiar el comportamiento esperado.

### 4. Verificación de Consistencia
Demostrar que el comportamiento funcional es idéntico.
- **Acción**: Ejecutar nuevamente las pruebas de la línea base.
- **Resultado**: Todas las pruebas deben pasar en "Verde" sin modificaciones en los casos de prueba.

### 5. Validación de Calidad
Confirmar que el cambio realmente aportó valor.
- **Métricas**: ¿Mejoró la legibilidad? ¿Se redujo la complejidad? ¿Es más fácil de testear?

---

## Reglas de Oro

- **Refactorización ≠ Funcionalidad Nueva**: Si necesitas cambiar qué hace el código, usa `/dev.build`. Si solo quieres cambiar cómo está escrito, usa `/dev.refactor`.
- **Pequeños Pasos**: Es mejor realizar 10 refactorizaciones pequeñas y probadas que una grande y arriesgada.
- **Cobertura de Tests**: No se recomienda refactorizar código que no tenga una suite de pruebas mínima que garantice su comportamiento.
- **Legibilidad sobre Astucia**: El objetivo es que el código sea más fácil de entender para humanos, no necesariamente más "corto" o "ingenioso".
