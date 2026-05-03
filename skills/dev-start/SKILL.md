---
name: dev.spec
description: Define and manage functional and technical specifications for software development.
argument-hint: "[functional|technical] [--approve|--iterate|--summary|--audio]"
---

# Skill: Development Specifications (/dev.spec)

Este skill permite guiar al usuario a través del proceso de definición de requerimientos y diseño técnico de una funcionalidad o sistema. Se enfoca en la claridad, la simplicidad y la alineación con los objetivos del proyecto.

## Uso del Comando

- `/dev.spec` → Detecta la fase actual y continúa el proceso.
- `/dev.spec "contexto inicial"` → Inicia la especificación con una descripción base.
- `/dev.spec functional` → Se enfoca exclusivamente en la fase funcional.
- `/dev.spec technical` → Se enfoca exclusivamente en la fase técnica.
- `/dev.spec --approve` → Aprueba formalmente la especificación actual.
- `/dev.spec --iterate "cambio"` → Refina la especificación existente con nuevas instrucciones.
- `/dev.spec --summary` → Genera un resumen rápido del estado actual.
- `/dev.spec --audio` → Permite capturar la descripción inicial vía voz.

---

## Proceso de Trabajo

El flujo de trabajo se divide en dos fases principales que deben completarse y aprobarse secuencialmente.

### Fase 1: Especificación Funcional (¿QUÉ construir?)

El objetivo es definir claramente el problema que se intenta resolver y el comportamiento esperado desde la perspectiva del usuario.

**Entrevista Sugerida (3-5 preguntas):**
1. **Objetivo y Valor**: ¿Cuál es el problema principal y qué resultado esperas obtener?
2. **Historias de Usuario**: ¿Cuáles son las acciones principales que realizará el usuario y qué recibirá a cambio?
3. **Alcance y Exclusiones**: ¿Hay algo específico que NO deba incluirse en esta solución?
4. **Criterios de Aceptación**: ¿Qué condiciones deben cumplirse para considerar que la funcionalidad está terminada?

**Secciones del Documento Funcional:**
- Resumen del Problema.
- Objetivos de la Funcionalidad.
- Historias de Usuario con Criterios de Aceptación.
- Casos de Uso y Escenarios E2E (opcional para proyectos grandes).
- Exclusiones (Fuera de Alcance).

---

### Fase 2: Especificación Técnica (¿CÓMO construir?)

Una vez aprobado el "qué", se define el diseño técnico y la arquitectura necesaria para implementar la solución de manera eficiente.

**Áreas de Diseño:**
1. **Arquitectura del Sistema**: Diagramas de flujo y estructura de componentes (preferiblemente usando Mermaid).
2. **Modelo de Datos**: Definición de entidades, esquemas de base de datos y relaciones.
3. **Contratos de API**: Definición de endpoints (REST/GraphQL), parámetros de entrada y respuestas de error estándar.
4. **Seguridad y Cumplimiento**: Gestión de secretos, autenticación y validación de datos.
5. **Estrategia de Pruebas**: Definición de pruebas unitarias e integración.

**Secciones del Documento Técnico:**
- Resumen Ejecutivo.
- Diagramas de Arquitectura.
- Detalle del Modelo de Datos.
- Definición de Interfaces/APIs.
- Decisiones de Diseño y Racional (ADR).
- Consideraciones de Desempeño y Seguridad.

---

## Reglas y Directrices Generales

### Interacción con el Usuario
- **Sin Redundancias**: No preguntes dos veces lo mismo. Si la información puede inferirse del contexto o de respuestas previas, úsala para pre-completar secciones.
- **Estándares de la Industria**: Aplica patrones de diseño estándar (REST, SOLID, etc.) por defecto. No preguntes sobre convenciones obvias (ej: códigos de error HTTP).
- **Validación Proactiva**: Antes de solicitar aprobación, valida que no existan contradicciones entre la parte funcional y la técnica.

### Gestión de Iteraciones (`--iterate`)
- **Vista Previa Obligatoria**: Nunca apliques cambios directamente. Muestra un "diff" o resumen de los cambios propuestos y solicita confirmación explícita.
- **Evaluación de Impacto**: Si un cambio en la especificación afecta tareas ya iniciadas, infórmalo al usuario.

### Estilo de Documentación
- **Idioma**: El idioma del documento debe ser consistente (por defecto el idioma en que el usuario está interactuando).
- **Términos Técnicos**: Mantén los términos técnicos comunes en inglés cuando sea apropiado (API, Endpoint, CRUD, etc.).
- **Simplicidad**: Prefiere la solución más sencilla que cumpla con los requerimientos. Evita el "over-engineering".

---

## Salidas del Proceso

Las especificaciones se almacenan generalmente en la estructura del proyecto bajo rutas predecibles para facilitar su consulta y mantenimiento futuro:
- Funcional: `docs/specs/functional/feature-name.md`
- Técnica: `docs/specs/technical/feature-name.md`

---

## Ayuda Rápida (`/dev.spec help`)

Muestra una guía resumida sobre cómo iniciar y gestionar especificaciones, incluyendo ejemplos de comandos comunes.
