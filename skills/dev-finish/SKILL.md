---
name: dev.finish
description: Finaliza la implementación de una funcionalidad, realiza validaciones finales y archiva el proyecto.
argument-hint: "[--force|--skip-tests]"
---

# Skill: Finalización y Archivados (/dev.finish)

Este skill marca el final del ciclo de vida de una funcionalidad (feature). Se encarga de asegurar que todo esté en orden, documentar los resultados y mover los archivos del área de trabajo activa al histórico del proyecto.

## Uso del Comando

- `/dev.finish` → Valida el estado final y archiva la funcionalidad (comportamiento estándar).
- `/dev.finish --force` → Omite algunas validaciones no críticas (no recomendado).
- `/dev.finish --skip-tests` → Salta la ejecución final de pruebas.

---

## Requisitos Previos

Antes de ejecutar este comando, se debe haber completado la fase de `/dev.build` con todas sus validaciones en verde. `/dev.finish` actúa como un control de calidad final.

---

## Flujo de Trabajo

### 1. Validación Final (Blocking)
El agente realiza una última verificación de seguridad:
- **Estado de Tareas**: Todas las tareas en `tasks.json` deben estar marcadas como `completed`.
- **Integridad del Código**: Re-ejecución de pruebas unitarias y verificación de compilación.
- **Calidad y Estilo**: Verificación de que no queden errores de linter o TODOs pendientes en el código.
- **Seguridad**: Escaneo rápido para asegurar que no se hayan filtrado secretos o credenciales.

### 2. Generación de Documentación
Se crean o actualizan archivos que resumen el trabajo realizado:
- **README.md**: Un resumen de alto nivel de la funcionalidad, componentes afectados y cómo probarla.
- **implementation-summary.md**: Un reporte con métricas de esfuerzo, archivos modificados y decisiones técnicas clave.

### 3. Gestión del Conocimiento (PATTERNS.md)
El agente analiza el proceso de implementación en busca de aprendizajes que puedan ser útiles para futuros desarrollos (ej: un patrón de configuración específico o una solución a un error común) y sugiere agregarlos al archivo `PATTERNS.md` del proyecto.

### 4. Proceso de Archivado
Una vez aprobado el cierre, el agente realiza las siguientes acciones:
1. **Limpieza**: Elimina archivos temporales, logs de ejecución y reportes de validación intermedios.
2. **Movimiento Atómico**: Mueve la carpeta de la funcionalidad desde el directorio de trabajo activo (`wip/`) al directorio histórico (`features/` o `history/`).
3. **Sincronización Final**: Verifica que los archivos se hayan movido correctamente y que no queden archivos huérfanos.

---

## Estructura de un Archivado Estándar

Al finalizar, la carpeta de la funcionalidad en `features/[nombre-feature]/` debe contener:
- `README.md` (Resumen del proyecto)
- `meta.md` (Metadatos y trazabilidad)
- `functional-spec.md` (Especificación funcional final)
- `technical-spec.md` (Diseño técnico final)
- `tasks.json` (Plan de trabajo ejecutado)
- `implementation-summary.md` (Resumen de métricas y cierre)

---

## Reglas de Oro

- **Nunca archivar con fallos**: Si una prueba falla o hay tareas pendientes, el archivado se bloquea.
- **Preservar el Historial**: El archivo `meta.md` es sagrado; contiene la trazabilidad de quién, cuándo y cómo se hizo la funcionalidad.
- **Limpieza Rigurosa**: No deben quedar carpetas de "verdictos" o archivos temporales en el histórico.
- **Confirmación del Usuario**: En modo estándar, el agente siempre pedirá confirmación antes de mover los archivos físicamente.
