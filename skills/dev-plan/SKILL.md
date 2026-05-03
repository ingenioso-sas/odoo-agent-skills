---
name: dev.plan
description: Transforma especificaciones técnicas aprobadas en tareas de implementación ejecutables.
argument-hint: "[--approve|--refine|--view|--resume]"
---

# Skill: Planificación de Tareas (/dev.plan)

Este skill permite descomponer una especificación técnica en un plan de trabajo estructurado y detallado. Organiza las tareas por capas lógicas, gestiona dependencias y establece criterios de aceptación claros para cada paso de la implementación.

## Uso del Comando

- `/dev.plan` → Genera automáticamente el plan de tareas basado en las especificaciones actuales.
- `/dev.plan --refine` → Permite ajustar, añadir o dividir tareas del plan existente.
- `/dev.plan --approve` → Aprueba formalmente el plan y selecciona la estrategia de ejecución.
- `/dev.plan --view` → Muestra una vista estructurada o interactiva de las tareas.
- `/dev.plan --resume` → Reanuda una sesión de planificación interrumpida.

---

## Requisitos Previos

| Requisito | Acción en caso de fallo |
|-----------|-------------------------|
| Especificación Técnica Aprobada | Ejecutar `/dev.spec technical --approve` |
| Conflictos de Spec Resueltos | Revisar y corregir en la fase de `/dev.spec` |

---

## Flujo de Trabajo

### 1. Lectura y Análisis
El agente lee las especificaciones funcionales y técnicas aprobadas para identificar componentes, cambios en el modelo de datos, nuevos endpoints y requerimientos de infraestructura.

### 2. Generación de Tareas
Se crean tareas siguiendo principios de modularidad y claridad. Cada tarea debe tener:
- **Título Conciso**: Acción clara a realizar.
- **Descripción**: 2-3 frases sobre el objetivo técnico.
- **Capa (Layer)**: Ubicación en el stack de ejecución.
- **Criterios de Aceptación (AC)**: Condiciones específicas para considerar la tarea como "Hecha".
- **Puertas de Calidad (GATEs)**: Validaciones automáticas (ej: tests que deben pasar).

### 3. Organización por Capas (Layers)

Para asegurar un desarrollo ordenado, las tareas se agrupan en las siguientes capas:

| Capa | Nombre | Propósito | Ejemplo de Tareas |
|------|--------|-----------|-------------------|
| **L1** | **Cimientos** | Configuración base e infraestructura. | Setup de BD, Dockerfile, Config de servicios. |
| **L2** | **Núcleo** | Lógica de negocio y persistencia. | Modelos, Repositorios, Servicios, APIs. |
| **L3** | **Interfaz** | Capa visual o integración externa. | Componentes UI, Integración con terceros. |
| **L4** | **Calidad** | Validaciones finales y pulido. | Pruebas E2E, Refactorización, Documentación. |

---

## Estructura de Salida (`tasks.json`)

El plan se almacena en un archivo `tasks.json` que actúa como la única fuente de verdad para la fase de construcción (`/dev.build`).

```json
{
  "feature": "nombre-funcionalidad",
  "stats": { "total": 10, "done": 0 },
  "tasks": [
    {
      "id": "TASK-001",
      "title": "Configuración de Base de Datos",
      "description": "Crear migraciones y definir esquemas iniciales.",
      "status": "pending",
      "layer": 1,
      "depends_on": [],
      "acceptance_criteria": [
        "AC-1: Tablas creadas según el diseño técnico",
        "GATE: La migración se aplica correctamente en el entorno local"
      ]
    }
  ]
}
```

---

## Estrategias de Ejecución

Al aprobar el plan (`--approve`), el usuario puede elegir cómo desea que el agente aborde las tareas:

1. **Secuencial**: Tarea por tarea, ideal para flujos lineales y simples.
2. **Por Lotes (Batched)**: Agrupa tareas relacionadas para optimizar el contexto y la velocidad. (Recomendado).
3. **Paralelo**: Aborda múltiples tareas de manera simultánea si no tienen dependencias entre sí.

---

## Reglas de Oro

- **Sin Tareas de Despliegue**: El agente se enfoca en código, pruebas y configuraciones. El despliegue a producción es una fase externa.
- **IDs Deterministas**: Las tareas deben seguir una numeración secuencial (TASK-001, TASK-002...).
- **Estimación de Complejidad**: Cada tarea debe marcarse como Baja, Media o Alta para ayudar en la priorización.
- **Interacción Proactiva**: Si una tarea parece demasiado grande, el agente debe sugerir dividirla durante la fase de `--refine`.
