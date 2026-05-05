---
name: dev
description: Orquestador maestro del ciclo de vida de desarrollo de software (SDD Kit). Centraliza la navegación, detecta el estado del proyecto y deriva inteligentemente a la siguiente fase del workflow.
argument-hint: "[status|list|config]"
---

### REGLAS ESTRICTAS DE LECTURA E INTERACCIÓN

> [!IMPORTANT]
> Si en algún momento de la orquestación encuentras:
> 
> ⛔ INVOKE TOOL (do not print this, CALL the tool):
> `AskUserQuestion(questions=[{...}])`
> 
> ESTO ES UNA LLAMADA A LA HERRAMIENTA. DEBES LLAMARLA. NUNCA imprimas opciones JSON con `echo` ni solicites ingreso de datos por texto libre si la interfaz de la herramienta está disponible.

# Comando Base: `/dev`

**Misión**: Actuar como el "Router Inteligente" del proyecto. Analiza el entorno actual, lee los metadatos y recomienda o lanza automáticamente la fase que corresponde. Evita que el usuario tenga que recordar la secuencia de comandos.

## Modos de Operación

- `/dev` → (Piloto Automático) Ejecuta el heurístico de detección y sugiere qué hacer a continuación.
- `/dev status` → Muestra un panel (dashboard) holístico con el progreso, las puertas de calidad (gates) y métricas de completitud.
- `/dev list` → Lista de todo el inventario de features en `dev/wip/` y `dev/features/`.
- `/dev config` → Gestor de configuración del entorno (lee/escribe `user-profile.yaml` y `.gary`).

---

## Flujo Heurístico de Detección (Piloto Automático)

Cuando se ejecuta `/dev` sin argumentos, el agente DEBE realizar esta evaluación secuencial:

### 1. Detección de Directorio y Contexto
```bash
# Revisar si existe carpeta wip/
wip_count=$(ls -1 dev/wip/ 2>/dev/null | wc -l | tr -d ' ')
# Verificar si el repo tiene historial pero cero docs
has_code=$(bash ~/.dev-sdd-kit/tools/detection/detect-stack.sh . --has-code-only)
```

### 2. Árbol de Decisión (Enrutamiento)

Evalúa secuencialmente y detente en la primera condición que se cumpla:

1. **Repo heredado sin documentación**:
   - *Condición*: `has_code == true` y no existe `dev/wip/` ni `dev/features/`.
   - *Acción*: Mostrar mensaje "Se detectó código existente sin especificaciones".
   - *Derivación*: Llama a `AskUserQuestion` ofreciendo lanzar `/dev.reveng` para aplicar ingeniería inversa.

2. **Inicio (Arranque de Cero)**:
   - *Condición*: Directorio `dev/wip/` está vacío o no existe.
   - *Acción*: Sugerir crear una funcionalidad nueva.
   - *Derivación*: Lanza la sugerencia de iniciar con `/dev.start`.

3. **Análisis del WIP Actual** (Si hay exactamente 1 feature activa en `wip/`):
   - Lee el archivo `meta.md` de esa feature.
   - Determina el `status` de cada fase:
     - `stages.functional == draft` → Deriva a `/dev.spec functional`
     - `stages.technical == draft` → Deriva a `/dev.spec technical`
     - `stages.technical == approved` y no hay `tasks.json` → Advierte que el contexto es denso. Ejecuta compactación (`/dev.check --compact`) y luego deriva a `/dev.plan`.
     - Hay `tasks.json` con tareas `pending` o `in_progress` → Muestra resumen rápido y deriva a `/dev.build`.
     - Todo `tasks.json` está en "done" → Indica que el desarrollo culminó y deriva a `/dev.finish`.

4. **Multi-WIP (Múltiples features activas)**:
   - *Condición*: `dev/wip/` tiene > 1 feature.
   - *Acción*: Presenta un `AskUserQuestion` listando los features activos para que el usuario escoja en cuál enfocar el radar, antes de evaluar la fase.

---

## Dashboard de Progreso (`/dev status`)

Cuando se invoca `/dev status`, genera un panel consolidado, pero **DEBES usar subagentes y scripts** para recolectar la data. No inventes los porcentajes.

```bash
# Recolecta métricas reales
stats=$(bash ~/.dev-sdd-kit/tools/metrics/get-status.sh dev/wip/$(ls dev/wip | head -1) --json)
```

Formato esperado de visualización:

```markdown
# 📡 Radar del Proyecto: [Nombre de la Feature]

> FASE ACTUAL: **[Detectada dinámicamente]**

### 📊 Avance del Ciclo (Stages)
- [✅] Especificación Funcional (Aprobado por: @user)
- [✅] Especificación Técnica (Aprobado por: @user)
- [🔄] Planificación y Tareas (En Curso - 5/12 completadas)
- [⏳] Finalización y QA (Pendiente)

### 🛡️ Escáner de Calidad (Gates)
| Métrica | Estado | Detalle |
|---------|--------|---------|
| Compilación | 🟢 PASA | Build exitoso en último commit |
| Unit Tests | 🟡 ALERTA | Cobertura 68% (Mínimo: 80%) |
| Linting | 🟢 PASA | 0 Errores críticos |
```

> **NOTA DE PERFIL**: Si el perfil en `user-profile.yaml` es `non-technical`, oculta la tabla de "Escáner de Calidad" y muestra simplemente: "Estado Técnico: Saludable / Requiere Atención".

---

## Protocolo de Pase de Batón (Handoff Protocol)

El Orquestador base transfiere el control a otras skills mediante delegaciones de `Skill()`. 
**Regla de Compactación de Contexto**:
Antes de delegar de `/dev.spec` hacia `/dev.plan`, o de `/dev.plan` hacia `/dev.build`, el orquestador DEBE sugerir la liberación de contexto en memoria.
Muestra este mensaje fijo:
```
🧹 CONTEXT HANDOFF:
Estás cruzando una frontera de fase. Para maximizar la precisión del subagente destino, se recomienda ejecutar /clear en tu entorno y luego continuar con la siguiente skill propuesta.
```

---

## Delegaciones Directas de Soporte

El orquestador también maneja llamadas para escenarios de mantenimiento rápido:
- Si el usuario dice "ayudame a refactorizar esto" → Deriva de inmediato a `/dev.refactor`.
- Si el usuario reporta un bug → Usa `Skill("dev.fix")`.
- Si pide ver tareas descartadas o por hacer a futuro → Usa `Skill("dev.backlog")`.
