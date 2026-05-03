---
name: dev
description: Orquestador central del ciclo de vida de desarrollo de software. Gestiona el flujo desde la idea hasta el archivado.
---

# Orquestador de Desarrollo (/dev)

Este es el comando central para gestionar el ciclo de vida de tus funcionalidades. Detecta automáticamente en qué etapa te encuentras y te guía hacia el siguiente paso lógico, integrando todas las skills de desarrollo.

## Comandos Principales

- `/dev` → **Piloto Automático**: Analiza el proyecto y sugiere o ejecuta la siguiente acción necesaria.
- `/dev status` → **Panel de Control**: Muestra un resumen visual del estado de la funcionalidad actual.
- `/dev list` → **Explorador**: Lista todas las funcionalidades activas (`wip/`) y terminadas (`features/`).
- `/dev config` → **Ajustes**: Configura las reglas, lenguajes y estándares del proyecto.

---

## Flujo de Trabajo Inteligente

Al ejecutar `/dev`, el agente realiza una detección de fase basada en el estado de los archivos:

| Estado Detectado | Acción Sugerida | Skill Delegada |
|------------------|-----------------|----------------|
| Sin área de trabajo activa | Iniciar nueva funcionalidad | `/dev.spec` |
| Especificación incompleta | Refinar requerimientos | `/dev.spec` |
| Spec aprobada, sin tareas | Crear plan de ejecución | `/dev.plan` |
| Tareas pendientes | Continuar implementación | `/dev.build` |
| Error detectado o Bug reportado | Corregir fallo técnico | `/dev.fix` |
| Deuda técnica identificada | Mejorar estructura de código | `/dev.refactor` |
| Tareas completadas | Validar y archivar | `/dev.finish` |

---

## Panel de Control (`/dev status`)

Muestra una visualización premium del progreso actual:

```markdown
# Estado del Proyecto: [Nombre de la Funcionalidad]
> Etapa Actual: 🔄 **Construcción** (Capa 2: Núcleo)

### 📊 Progreso General
| Fase | Estado | Resultado |
|------|--------|-----------|
| **1. Especificación** | ✅ Completo | Aprobada |
| **2. Planificación**  | ✅ Completo | 12 tareas generadas |
| **3. Construcción**   | 🔄 En Curso | 5/12 completadas (42%) |
| **4. Finalización**   | ⏳ Pendiente | - |

### 🛠️ Calidad y Puertas (Gates)
- **Compilación**: ✅ Verde
- **Tests Unitarios**: 🔄 85% Cobertura
- **Linter**: ✅ Sin advertencias
- **Seguridad**: ✅ Sin hallazgos
```

---

## Gestión de Estado

El orquestador utiliza el archivo `meta.md` en la carpeta `wip/` de cada funcionalidad para mantener la trazabilidad. No es necesario que el usuario edite este archivo; el orquestador lo mantiene actualizado automáticamente.

---

## Beneficios del Orquestador

1. **Simplicidad**: No necesitas recordar nombres de comandos complejos; `/dev` sabe qué hacer.
2. **Consistencia**: Asegura que se sigan todos los pasos del ciclo de vida sin saltarse validaciones críticas.
3. **Visibilidad**: Proporciona una fuente única de verdad sobre el avance del equipo y la calidad del código.
4. **Agilidad**: Reduce la fricción entre fases (especificación -> planificación -> construcción).

---

## Integraciones

El orquestador delega el trabajo pesado a las skills especializadas:
- [dev.spec](file:///Users/andersonbuitron/proyectos/ia-agent-skills/skills/dev-start/SKILL.md)
- [dev.plan](file:///Users/andersonbuitron/proyectos/ia-agent-skills/skills/dev-plan/SKILL.md)
- [dev.build](file:///Users/andersonbuitron/proyectos/ia-agent-skills/skills/dev-build/SKILL.md)
- [dev.fix](file:///Users/andersonbuitron/proyectos/ia-agent-skills/skills/dev-fix/SKILL.md)
- [dev.refactor](file:///Users/andersonbuitron/proyectos/ia-agent-skills/skills/dev-refactor/SKILL.md)
- [dev.backlog](file:///Users/andersonbuitron/proyectos/ia-agent-skills/skills/dev-backlog/SKILL.md)
- [dev.finish](file:///Users/andersonbuitron/proyectos/ia-agent-skills/skills/dev-finish/SKILL.md)
