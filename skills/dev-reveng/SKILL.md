---
name: dev.reverse-eng
description: Realizar ingeniería inversa de una base de código existente para generar especificaciones SDD. Úsalo cuando el usuario quiera crear especificaciones a partir de código existente.
model: opus
argument-hint: "[scope]"
---

### CÓMO LEER ESTE SKILL

Cuando veas un bloque como este:

⛔ INVOKE TOOL (do not print this, CALL the tool):
AskUserQuestion(questions=[{...}])

Esta es una LLAMADA A UNA HERRAMIENTA (TOOL CALL) que debes ejecutar, no texto para mostrar.

| INCORRECTO | CORRECTO |
|-------|---------|
| Bash(echo "1. Opción A") | Llama directamente a la herramienta AskUserQuestion |
| Imprimir el JSON en la terminal | Pasa los parámetros mostrados a la herramienta |

# Comando: /dev.reverse-eng

**Versión**: 3.0.0
**Última Actualización**: 2026-05-04
**Descripción**: Realizar ingeniería inversa de una base de código existente para generar especificaciones que permitan una evolución guiada por especificaciones (Spec-Driven Development).

**Uso**:
- `/dev.reverse-eng` → Analizar directorio actual
- `/dev.reverse-eng [ruta]` → Analizar ruta específica
- `/dev.reverse-eng --focus api,database` → Enfocarse en áreas específicas

---

## Ayuda Rápida

> `/dev.reverse-eng help` → Muestra este resumen

**Sintaxis**: `/dev.reverse-eng [ruta] [flags]`

| Flag | Descripción |
|------|-------------|
| (ninguno) | Analizar directorio actual |
| `[ruta]` | Analizar ruta específica |
| `--focus <componente>` | Análisis profundo de un componente específico, enriqueciendo specs existentes |

**Nota sobre `--focus`**: Este flag enriquece las especificaciones existentes con más detalles sobre un componente.
NO crea archivos de especificación separados; actualiza `functional-spec.md` y `technical-spec.md` directamente.

Casos de uso:
- Extracción general primero, luego `--focus ServicioDePagos` para más detalle
- Re-extraer un componente específico que fue analizado superficialmente
- Añadir detalle a especificaciones existentes en proyectos legacy (brownfield)

**Ejemplos**:
```bash
/dev.reverse-eng                            # Analizar directorio actual
/dev.reverse-eng ./src                      # Analizar ruta específica
/dev.reverse-eng --focus PaymentService     # Análisis profundo de PaymentService
```

---

REGLAS CRÍTICAS DE INTERACCIÓN CON EL USUARIO
Cuando este skill muestre un JSON para AskUserQuestion, DEBES:
  1. LLAMAR a la HERRAMIENTA AskUserQuestion con ese JSON exacto
  2. NO imprimir opciones usando Bash (sin echo, cat, printf)
  3. NO preguntar "¿Qué opción eliges?" como texto
  4. Las tablas marcadas "SOLO REFERENCIA" son para documentación - NO las imprimas

## ⚡ Primer Paso: Selección de Modo (OBLIGATORIO)

> **🚨 CRÍTICO**: Antes de CUALQUIER trabajo de extracción, SIEMPRE presenta la selección de modo al usuario.

```
┌─────────────────────────────────────────────────────────────────┐
│  /dev.reverse-eng - Selección de Modo                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Estado detectado:                                               │
│  • dev/extracted/ existe: [SÍ/NO]                               │
│  • dev/specs/ existe: [SÍ/NO]                                   │
│                                                                  │
│  Selecciona modo:                                                │
│  1. EXTRACCIÓN COMPLETA - Análisis completo desde cero           │
│  2. MODO ACTUALIZACIÓN - Re-analizar y fusionar con specs       │
│  3. VER ESTADO - Mostrar resumen de la extracción actual         │
│                                                                  │
│  [Si dev/specs/ existe pero dev/extracted/ no]:                 │
│  4. MEJORAR SPECS - Añadir detalles faltantes a specs existentes │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**⛔ INVOKE TOOL (do not print this, CALL the tool):**

```json
{
  "questions": [{
    "question": "Selecciona el modo de ingeniería inversa para esta base de código.",
    "header": "Selección de Modo",
    "options": [
      {"label": "EXTRACCIÓN COMPLETA", "description": "Análisis completo desde cero"},
      {"label": "MODO ACTUALIZACIÓN", "description": "Re-analizar y fusionar con specs existentes"},
      {"label": "VER ESTADO", "description": "Mostrar resumen de la extracción actual"},
      {"label": "MEJORAR SPECS", "description": "Añadir detalles faltantes a specs (solo si dev/specs/ existe pero dev/extracted/ no)"}
    ],
    "multiSelect": false
  }]
}
```

### Comportamiento del Modo

| Modo | Condición | Comportamiento |
|------|-----------|----------|
| **EXTRACCIÓN COMPLETA** | Cualquier estado | Eliminar `dev/extracted/` existente, crear nuevo, ejecutar Fases 0-7 |
| **MODO ACTUALIZACIÓN** | `dev/extracted/` existe | Re-ejecutar extracción, comparar diferencias, actualizar TODOS los archivos |
| **MODO ACTUALIZACIÓN + --focus** | `dev/extracted/` existe + flag `--focus` | **Enriquecer** specs existentes con detalle del componente enfocado |
| **VER ESTADO** | `dev/extracted/` existe | Mostrar resumen de extracción actual, sin cambios |
| **MEJORAR SPECS** | `dev/specs/` existe, falta `dev/extracted/` | Analizar código para añadir detalles a specs existentes |

### Comportamiento de `--focus` (CRÍTICO)

> **REGLA**: `--focus` SIEMPRE actualiza los archivos base. NUNCA crea archivos con sufijos `-{componente}.md`.

**Lo que hace `--focus`**:
1. Extrae detalle profundo sobre el componente especificado
2. **Fusiona** ese detalle en las especificaciones existentes (o las crea si no existen)
3. Marca las secciones como `[Enfocado: NombreComponente]` para trazabilidad

**Lo que `--focus` NO hace**:
- Crear archivos `functional-spec-{component}.md`
- Crear versiones paralelas de especificaciones
- Eliminar contenido existente de otros componentes

### Archivos a Actualizar en Re-extracción

> **CRÍTICO**: Al re-ejecutar `/dev.reverse-eng`, TODOS estos archivos DEBEN ser **REEMPLAZADOS** (sin sufijos):

| Archivo | Qué Actualizar | Ubicación |
|------|----------------|----------|
| `functional-spec.md` | Casos de uso, actores, reglas de negocio | `dev/extracted/` Y `dev/specs/` |
| `technical-spec.md` | Arquitectura, APIs, integraciones | `dev/extracted/` Y `dev/specs/` |
| `DISCREPANCIES_REPORT.md` | Nuevas/resueltas discrepancias | `dev/extracted/` |
| `DOCUMENTATION_GAPS.md` | Análisis de cobertura | `dev/extracted/` |
| `raw/README.md` | Fecha de extracción, fuentes, metadatos | `dev/extracted/raw/` |
| `PATTERNS.md` | Patrones de proyecto descubiertos | `dev/extracted/` Y `dev/specs/` |

### ⚠️ ANTI-PATRÓN: Sin Sufijos `-UPDATED`

> **INCORRECTO**: Crear `functional-spec-UPDATED.md` junto al original
> **CORRECTO**: Reemplazar `functional-spec.md` directamente

```
❌ INCORRECTO (crea confusión):
dev/specs/
├── functional-spec.md           # Versión antigua
├── functional-spec-UPDATED.md   # Nueva versión - el usuario debe renombrar manualmente

✅ CORRECTO (actualización limpia):
dev/specs/
├── functional-spec.md           # Reemplazado con la nueva versión
```

**El Modo Actualización DEBE**:
1. Mostrar resumen de diferencias de los cambios
2. Pedir confirmación si se detecta >20% de cambios
3. **REEMPLAZAR archivos directamente** - sin sufijos, sin versiones en paralelo
4. Actualizar tanto `dev/extracted/` COMO `dev/specs/`

---

## Propósito

Realiza una ingeniería inversa exhaustiva en **ocho fases** (0-7):

| Fase | Nombre | Propósito |
|-------|------|---------|
| **0** | Detección de Estado | Identificar configuraciones y specs existentes antes de extraer |
| **1** | Análisis Estático de Código | Extraer datos mediante escaneo profundo del código fuente |
| **2** | Validación Cruzada Básica | Comparar código vs documentación existente (si la hay) |
| **3** | Validación Cruzada Profunda | Comparación a nivel de campo, detección de endpoints fantasma |
| **4** | Síntesis | Generar specs con indicadores de confianza de 6 niveles |
| **5** | Generar PATTERNS.md | Extraer patrones establecidos de la base de código |
| **6** | Verificación de Consistencia | Validar alineación funcional ↔ técnica |
| **7** | Promoción de Specs | **Copiar specs a `dev/specs/`** para el orquestador principal |

---

## Estructura de Salida (CANÓNICA)

> **⚠️ CRÍTICO**: Esta es la ÚNICA estructura de salida válida. Cualquier archivo fuera de estas ubicaciones es un error.

```text
dev/
├── extracted/                        # Directorio de TRABAJO (Fases 0-5)
│   ├── raw/                          # Fase 0-1: Datos fuente
│   │   ├── existing-specs/           # DEBE CREARSE si se detectan specs
│   │   │   └── DETECTION_REPORT.md   # Qué frameworks/specs se encontraron
│   │   ├── code-analysis/            # DEBE CREARSE siempre
│   │   │   ├── architecture/
│   │   │   ├── api-specs/
│   │   │   ├── database/
│   │   │   └── deployment/
│   │   └── README.md                 # Metadatos de extracción
│   │
│   ├── DOCUMENTATION_GAPS.md         # Fase 2: Reporte de validación cruzada
│   ├── DISCREPANCIES_REPORT.md       # Fase 3: Validación a nivel de campo
│   ├── functional-spec.md            # Fase 4: Spec sintetizada
│   ├── technical-spec.md             # Fase 4: Spec sintetizada
│   ├── PATTERNS.md                   # Fase 5: Patrones descubiertos
│   └── README.md                     # Índice y metadatos
│
├── specs/                            # Ubicación FINAL (Fase 7)
│   ├── functional-spec.md            # ← PROMOVIDO desde extracted/
│   └── technical-spec.md             # ← PROMOVIDO desde extracted/
│
└── PATTERNS.md                       # ← PROMOVIDO desde extracted/ (nivel raíz)
```

**PUNTOS CLAVE**:
- `dev/extracted/` = Directorio de trabajo con todos los artefactos
- `dev/specs/` = **Ubicación final** para specs globales (creado en Fase 7)
- La Fase 7 **PROMUEVE** las specs de `extracted/` a `specs/`
- **NUNCA escribas archivos en la raíz de `dev/` excepto `PATTERNS.md`**

---

## Reglas de Creación de Directorios (OBLIGATORIO)

> **🚨 CRÍTICO**: Antes de escribir CUALQUIER archivo, asegúrate de que los directorios padre existan.

**Fase 0 Pre-paso** - Crear estructura completa:

```bash
# SIEMPRE ejecutar al inicio de la extracción
mkdir -p dev/extracted/raw/existing-specs/
mkdir -p dev/extracted/raw/code-analysis/architecture/
mkdir -p dev/extracted/raw/code-analysis/api-specs/
mkdir -p dev/extracted/raw/code-analysis/database/
mkdir -p dev/extracted/raw/code-analysis/deployment/
mkdir -p dev/specs/
mkdir -p dev/wip/
```

---

## Flujo de Trabajo de Ocho Fases

### Fase 0 Pre-paso 2: Verificación de Configuración del Proyecto (OBLIGATORIO)

> **PROPÓSITO**: Asegurar que `PROJECT.md` y `CLAUDE.md` existan antes de la extracción.

**SIEMPRE ejecuta DESPUÉS de la creación de directorios**:

```pseudocode
# 1. Asegurar que PROJECT.md exista
IF NOT EXISTS dev/PROJECT.md:
    → Crea dev/PROJECT.md con lenguaje base ("es" por defecto) y convenciones detectadas
    → Ejemplo: language: es, stack: [Node, Express, etc.]

# 2. Asegurar que CLAUDE.md exista
IF .claude/ directory exists:
    spec_lang = leer language.specs de dev/PROJECT.md (fallback: "es")

    IF NOT EXISTS CLAUDE.md:
        → Generar CLAUDE.md con sección del SDD Kit
    ELSE IF CLAUDE.md existe pero le falta la sección "## SDD Kit":
        → Añadir sección (preservar contenido existente)
```

**Sección SDD Kit a inyectar**:
```markdown
## dev SDD Kit

Este proyecto utiliza **dev SDD Kit** para desarrollo basado en especificaciones.

### Idioma de Especificaciones
Todas las especificaciones DEBEN escribirse en **Español** (`es`).
No mezcles idiomas. Los términos técnicos (API, REST, CRUD) se mantienen en Inglés.

### Referencia Rápida
- Flujo de trabajo: `/dev` → `/dev.spec` → `/dev.plan` → `/dev.build` → `/dev.finish`
- Convenciones del proyecto: `dev/PROJECT.md`
- Patrones descubiertos: `dev/PATTERNS.md`
```

---

### Fase 0: Detección de Estado del Repositorio

> **PROPÓSITO**: Identificar documentación previa, APIs definidas (OpenAPI/Swagger) y dependencias principales.

**Salida**: `DETECTION_REPORT.md` en `dev/extracted/raw/existing-specs/` con los hallazgos y la estrategia seleccionada.

---

### Fase 1: Análisis Estático de Código

Escanea exhaustivamente el código para obtener datos verídicos. No inventes.

1. **Detección de Stack** - Node, Python, Java, Go, etc.
2. **Extracción de API** - Escanear controladores o archivos de rutas (express, fastapi, spring, etc.)
3. **Extracción de Base de Datos** - Escanear modelos de ORMs (Sequelize, SQLAlchemy, Hibernate, Prisma).
4. **Descubrimiento de Actores**:
   - Analizar configuraciones de API Gateway, Swagger, Webhooks.
   - Analizar suscripciones a colas de mensajes (RabbitMQ, Kafka, SQS).
   - Analizar trabajos programados (Cron, Schedulers).
   - Analizar integraciones externas (Clientes REST, URLs hardcodeadas).

---

### Fase 2 y 3: Validación Cruzada (Básica y Profunda)

> **CRÍTICO**: Esta fase detecta los errores más peligrosos: endpoints fantasma, valores enumerados faltantes.

Comparar la documentación existente (si se detectó en Fase 0) con el código escaneado (Fase 1).

**Chequeos**:
| Tipo de Chequeo | Qué Comparar | Salida |
|------------|-----------------|--------|
| **Existencia de Endpoints** | Rutas en README/Swagger vs Rutas en Código | Endpoints fantasma |
| **Campos de Entidad** | Modelos descritos vs ORM real | Diferencia de campos |

**Salidas**: `DOCUMENTATION_GAPS.md` y `DISCREPANCIES_REPORT.md`.

---

### Fase 4: Síntesis con Indicadores de Confianza

Transformar datos en especificaciones, marcando el **origen** de cada pieza de información.

#### Sistema de Confianza de Seis Niveles (CRÍTICO)

| Nivel | Ícono | Significado | Acción |
|-------|------|---------|--------|
| **THREE_WAY** | ✅✅✅ | Encontrado en múltiples fuentes y TODO COINCIDE | Alta confianza |
| **VERIFIED** | ✅✅ | Encontrado en documentación y código | Alta confianza |
| **PARTIAL** | ✅⚠️ | Encontrado en ambas, pero DIFIERE | Revisar diferencia |
| **CODE_ONLY** | 🔸 | Solo en código (confiable, indocumentado) | Documentar |
| **DOCS_ONLY** | ⚠️ | Solo en docs (NO en código) | ¡VERIFICAR! Es un fantasma |
| **UNKNOWN** | ❓ | Información insuficiente | NO USAR sin verificar |

**Reglas de Sobrescritura**:
> **CRÍTICO**: Cuando hay discrepancias entre Docs (README) y CÓDIGO,
> las especificaciones generadas DEBEN reflejar la realidad del CÓDIGO, NO las afirmaciones del README.

**Genera**:
- `functional-spec.md` - Casos de uso, capacidades, actores (con confianza)
- `technical-spec.md` - Arquitectura, APIs, BD (con confianza)

---

### Fase 5: Generar PATTERNS.md

> **PROPÓSITO**: Extraer convenciones establecidas del código base para acelerar el desarrollo futuro.

**Formato Requerido para cada patrón**:

```markdown
### [Nombre del Patrón]

**Categoría**: [HTTP/API | Database | Messaging | Error Handling | Testing | Security]

**Evidencia**: Usado en:
- `path/to/file1.ts:42`
- `path/to/file2.ts:87`

**Ejemplo**:
```typescript
// Máximo 20 líneas de código mostrando el patrón
```

**Cuándo usar**: [1-2 oraciones explicando cuándo aplicar este patrón]
```

---

### Fase 6: Verificación de Consistencia Funcional ↔ Técnica

> **🚨 BLOQUEANTE**: Valida que todo endpoint documentado en el `technical-spec` tenga una razón de ser en el `functional-spec`, y viceversa.

---

### Fase 7: Promoción de Especificaciones (OBLIGATORIO)

> **PROPÓSITO**: Copiar las especificaciones sintetizadas a `dev/specs/` - la ubicación canónica para las specs globales.

#### Paso 1: Ejecutar Promoción

```bash
mkdir -p dev/specs
# Copiar Specs
cp dev/extracted/functional-spec.md dev/specs/functional-spec.md
cp dev/extracted/technical-spec.md dev/specs/technical-spec.md
# Copiar Patrones
cp dev/extracted/PATTERNS.md dev/PATTERNS.md
```

#### Comportamiento en Modo Actualización
Si `dev/specs/` ya existe:
- **REEMPLAZAR** archivos directamente (sin sufijo `-UPDATED`).

---

## Protocolo Anti-Truncamiento (CRÍTICO)

> **NUNCA TRUNQUES campos de entidades, valores enum o listas de endpoints.**

```
┌─────────────────────────────────────────────────────────────────────┐
│  REGLA ANTI-TRUNCAMIENTO                                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Si una entidad tiene 48 campos → Documenta TODOS los 48 campos     │
│  Si un enum tiene 15 valores → Lista TODOS los 15 valores           │
│  Si hay 44 endpoints → Documenta TODOS los 44 endpoints             │
│                                                                     │
│  NO DEBES:                                                          │
│  - Usar "..." para truncar                                          │
│  - Decir "y X más"                                                  │
│  - Resumir en lugar de listar exhaustivamente                       │
│                                                                     │
│  POR QUÉ: Specs truncadas causan fallos de integración.             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Instrucciones del Agente

1. **SIEMPRE** muestra la pregunta de AskUserQuestion para el Modo antes de iniciar.
2. **NUNCA** inventes endpoints, APIs o configuraciones que no existan en el código. (Protocolo Anti-Invención).
3. **MANTÉN** la estructura de salida estrictamente como se define.
4. Aplica el **Protocolo Anti-Truncamiento** rigurosamente durante la escritura de `technical-spec.md`.
