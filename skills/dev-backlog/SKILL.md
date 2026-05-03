---
name: dev.backlog
description: Gestión y priorización de tareas pendientes, errores y deuda técnica.
argument-hint: "[add|list|pick|refine]"
---

# Skill: Gestión de Backlog (/dev.backlog)

Este skill actúa como la memoria a largo plazo del proyecto. Permite capturar, organizar y priorizar el trabajo futuro, asegurando que ninguna idea o problema técnico se pierda.

## Uso del Comando

- `/dev.backlog add` → Captura una nueva tarea, bug o mejora.
- `/dev.backlog list` → Muestra la lista de tareas pendientes organizada por prioridad.
- `/dev.backlog pick [ID]` → Selecciona una tarea para comenzar el ciclo de desarrollo (`/dev.start`).
- `/dev.backlog refine [ID]` → Añade detalles técnicos o desglosa una tarea del backlog.

---

## Tipos de Elementos en el Backlog

Para mantener el orden, cada elemento debe clasificarse en una de estas categorías:
- **Feature**: Nueva funcionalidad o mejora de negocio.
- **Bug**: Comportamiento incorrecto reportado.
- **Debt**: Deuda técnica o refactorización necesaria.
- **Chore**: Tareas de mantenimiento (ej: actualización de dependencias).

---

## Flujo de Trabajo

### 1. Captura (`add`)
El agente solicita información básica: título, descripción, tipo y una estimación inicial de esfuerzo (S, M, L, XL). 
- **Persistencia**: Se guarda en el archivo central `dev/backlog.md`.

### 2. Organización (`list`)
El agente presenta una vista clara del backlog:
```markdown
| ID | Tipo | Título | Esfuerzo | Prioridad |
|----|------|--------|----------|-----------|
| US-001 | Feature | Implementar Login | M | Alta |
| BUG-042 | Bug | Error en validación de email | S | Crítica |
| DB-005 | Debt | Refactorizar capa de persistencia | L | Media |
```

### 3. Selección (`pick`)
Al elegir una tarea, el orquestador maestro realiza el puente hacia `/dev.start`, utilizando la descripción del backlog como base para la fase de especificación.

---

## Reglas de Oro

- **Fuente Única de Verdad**: El archivo `dev/backlog.md` debe ser el único lugar donde se gestiona el trabajo pendiente.
- **Descripción Accionable**: Evita entradas vagas. Cada elemento debe tener suficiente contexto para que un agente pueda empezar a trabajar en él.
- **Limpieza Periódica**: El agente debe sugerir eliminar o archivar tareas que ya no sean relevantes para el proyecto.
