# Guía de Gestión de Proyecto — ALPR Access System

> Esta guía define cómo clasificar, organizar y gestionar los issues en todos los repositorios
> de la organización `alpr-access-system`. Está diseñada para que cualquier miembro del equipo
> pueda entender y aplicar la metodología en menos de 10 minutos.
>
> **Última actualización:** 15 de agosto de 2026

---

## Índice

1. [Filosofía: Cero Labels](#filosofía-cero-labels)
2. [Arquitectura de 3 Capas](#arquitectura-de-3-capas)
3. [Capa 1: Tipos de Issue (Issue Types)](#capa-1-tipos-de-issue)
4. [Capa 1: Campos del Issue (Issue Fields)](#capa-1-campos-del-issue)
5. [Capa 2: Campos del Proyecto (Project Fields)](#capa-2-campos-del-proyecto)
6. [Capa 3: Milestones y Relaciones](#capa-3-milestones-y-relaciones)
7. [Tutorial: Cómo crear un Issue correctamente](#tutorial-cómo-crear-un-issue-correctamente)
8. [Referencia rápida](#referencia-rápida)

---

## Filosofía: Cero Labels

Este proyecto **no usa labels tradicionales de GitHub**. En su lugar, aprovechamos al máximo
las herramientas nativas de GitHub (2025–2026):

- **Issue Types** → Clasifican el tipo de trabajo (reemplazan labels como `feat`, `bug`, `docs`)
- **Issue Fields** → Metadatos estructurados (reemplazan labels como `prioridad:alta`, `edge`, `infra`)
- **Project Fields** → Gestión ágil del flujo de trabajo (Status, Iteración, Tamaño)
- **Relationships** → Bloqueos y dependencias (reemplazan labels como `blocked`)

### ¿Por qué?

| Labels (antes) | Campos nativos (ahora) |
|---|---|
| Se olvidan fácilmente | Los campos se pueden marcar como obligatorios |
| No se pueden filtrar por categoría | Cada campo tiene su propio filtro |
| Todo mezclado en una sola lista | Cada campo tiene su propia sección |
| Sin reportes | GitHub genera gráficos automáticos |
| No se pueden agrupar | Puedes agrupar columnas del tablero por campo |

---

## Arquitectura de 3 Capas

```
┌──────────────────────────────────────────────────────────────────┐
│  CAPA 1: ORGANIZACIÓN  (Settings → Planning)                    │
│  Aplica automáticamente a TODOS los repos y proyectos           │
│                                                                  │
│  Issue Types   → ¿QUÉ tipo de trabajo es?                       │
│  Issue Fields  → Metadatos estructurados del issue               │
├──────────────────────────────────────────────────────────────────┤
│  CAPA 2: PROYECTO  (Project Settings → Fields)                  │
│  Aplica solo dentro del tablero del Project                     │
│                                                                  │
│  Project Fields → Gestión ágil del flujo de trabajo              │
├──────────────────────────────────────────────────────────────────┤
│  CAPA 3: REPOSITORIO                                            │
│  Configuración específica de cada repo                          │
│                                                                  │
│  Milestones    → Entregas concretas con fecha límite             │
│  Relationships → Bloqueos, dependencias, jerarquías              │
└──────────────────────────────────────────────────────────────────┘
```

**Regla de oro:** Si un dato tiene un campo nativo en GitHub, úsalo. No crees labels.

---

## Capa 1: Tipos de Issue

> **Dónde se configuran:** Settings de la organización → Planning → Issue types
>
> **Qué son:** Clasifican la *naturaleza* del trabajo. Son **mutuamente excluyentes** — un issue
> solo puede tener UN tipo. Se ven en la barra lateral de cada issue.

### Tipos configurados

| Tipo | Descripción | Cuándo usar |
|---|---|---|
| 🔵 **Funcionalidad** | Nueva funcionalidad o mejora | El sistema **no hace algo** y debería hacerlo |
| 🔴 **Error** | Problema o comportamiento inesperado | Algo que existe **no funciona como debería** |
| 🟡 **Tarea** | Trabajo técnico o de mantenimiento | Trabajo que **no es feature nueva ni bug** |
| 🟣 **Documentación** | Redacción de specs, ADRs, README, guías | El entregable es **solo texto** |
| 🟢 **Modelo IA** | Entrenamiento, dataset, evaluación del modelo | Todo el **ciclo de vida del modelo** |
| 🟠 **Testing** | Pruebas, validaciones, verificaciones | Escribir o ejecutar **tests del sistema** |

### Ejemplos por tipo

**Funcionalidad:**
- Implementar CRUD de funcionarios en el panel web
- Agregar endpoint de consulta de historial de accesos
- Vista de dashboard con últimos 10 registros de acceso

**Error:**
- El pipeline falla cuando la cámara no detecta ningún vehículo por > 5 s
- El token JWT expira sin renovación automática

**Tarea:**
- Configurar Docker Compose para desarrollo local
- Configurar GitHub Actions para lint automático en PRs
- Refactorizar módulo de validación de patentes

**Documentación:**
- Completar ADR-001 con la justificación de FastAPI
- Actualizar README con instrucciones de setup
- Escribir la especificación funcional del Admin Panel

**Modelo IA:**
- Preparar dataset con 500 imágenes de patentes con sombra
- Evaluar YOLOv12s vs YOLOv10 en dataset de validación
- Ajustar umbral de confianza mínimo para reducir falsos positivos

**Testing:**
- Crear tests de integración para el endpoint de accesos
- Validar pipeline de OCR con imágenes en condiciones adversas
- Verificar comportamiento offline del nodo edge

---

## Capa 1: Campos del Issue

> **Dónde se configuran:** Settings de la organización → Planning → Issue fields
>
> **Qué son:** Metadatos estructurados que **viven en el issue** y se ven en cualquier lugar
> donde se abra (repositorio, proyecto, búsqueda global). Son la "fuente de verdad".

### Campos configurados

#### Prioridad (Priority)

Define la urgencia del issue.

| Opción | Cuándo usar |
|---|---|
| **Urgent** | Bloquea todo. Resolver HOY |
| **High** | Debe resolverse en el sprint actual, sin excepción |
| **Medium** | Importante pero no bloquea. Este sprint o el siguiente |
| **Low** | Deseable. Se trabaja cuando no hay nada de mayor prioridad |

#### Módulo

Identifica **qué parte del sistema** afecta el issue.

| Opción | Descripción |
|---|---|
| **Nodo Central** | Admin Panel: React SPA, FastAPI, PostgreSQL, Coolify |
| **Nodo Edge** | Nodo de Borde: Pipeline IA, YOLOv12, OpenCV, SQLite |
| **Ambos** | Integración o sincronización entre ambos nodos |
| **General** | No es específico de un nodo (documentación general, tesis) |

#### Área

Identifica el **área técnica transversal** del issue.

| Opción | Descripción |
|---|---|
| **General** | Trabajo estándar de desarrollo (default) |
| **Infraestructura** | Docker, CI/CD, deploy, Coolify, configuración de entornos |
| **Seguridad** | Vulnerabilidades, mitigaciones, hardening, análisis de riesgos |

---

## Capa 2: Campos del Proyecto

> **Dónde se configuran:** Project → Settings → Fields
>
> **Qué son:** Campos específicos del tablero que sirven para la **gestión ágil del flujo**.
> Solo se ven dentro del proyecto, no en la vista del repositorio.

### Campos configurados

#### Status

Define el estado del issue dentro del flujo de trabajo. Se traduce en las **columnas del tablero**.

| Opción | Descripción |
|---|---|
| **Backlog** | Identificado pero no planificado para ningún sprint |
| **Por Hacer** | Planificado para el sprint actual, listo para ser tomado |
| **En Progreso** | Alguien lo está trabajando activamente |
| **En Revisión** | Tiene un PR abierto esperando revisión |
| **Hecho** | Completado y cerrado |

#### Iteración (Sprints)

Bloques de trabajo de **2 semanas**. Cada issue se asigna al sprint en que se trabaja.

| Sprint | Fechas | Foco principal |
|---|---|---|
| Sprint 0 | Ago 13 – Ago 26 | Documentación, specs, setup inicial |
| Sprint 1 | Ago 27 – Sep 09 | Base técnica, primeros CRUDs, pipeline base |
| Sprint 2 | Sep 10 – Sep 23 | Entrenamiento IA, CRUDs avanzados |

#### Tamaño (Size)

Estimación ágil del tamaño de la tarea.

| Opción | Equivale a | Ejemplo |
|---|---|---|
| **XS** | < 1 hora | Corregir un typo, actualizar README |
| **S** | 1–4 horas | Agregar un endpoint simple, escribir un ADR |
| **M** | 1–2 días | CRUD completo, configurar Docker Compose |
| **L** | 3–5 días | Pipeline de captura de video completo |
| **XL** | > 1 semana | Entrenamiento completo del modelo YOLOv12 |

#### Progreso de Sub-issues

Campo automático que muestra el porcentaje de sub-issues completados. Útil para épicas.

---

## Capa 3: Milestones y Relaciones

### Milestones

> **Dónde se configuran:** Repositorio → Issues → Milestones
>
> **Qué son:** Representan **entregas concretas con fecha límite**. Son diferentes a los Sprints:
> un Sprint es un bloque de tiempo, un Milestone es un **objetivo de entrega** (versión, demo, etc.).

#### Milestones de `alpr-admin-panel`

| Milestone | Descripción | Fecha |
|---|---|---|
| v0.1 - Scaffold | Proyecto base: Docker + FastAPI + React funcionando | 02 Sep 2026 |
| v0.2 - CRUD Base | ABM de Funcionarios y Vehículos completo | 23 Sep 2026 |
| v1.0 - MVP | Panel completo con auth, sync y dashboard | Oct 2026 |

#### Milestones de `alpr-edge-node`

| Milestone | Descripción | Fecha |
|---|---|---|
| v0.1 - Pipeline Base | Captura de video + detección YOLOv12s funcionando | 09 Sep 2026 |
| v0.2 - OCR + Validación | Pipeline completo: detección + OCR + validación | 23 Sep 2026 |
| v1.0 - Prototipo Demo | Prototipo funcional para evaluadores | Oct 2026 |

#### Milestones de `alpr-documentacion`

| Milestone | Descripción | Fecha |
|---|---|---|
| Documentación Base | ADRs, Specs, Cronograma, Guías completados | 26 Ago 2026 |

### Relaciones (Relationships)

GitHub permite establecer relaciones nativas entre issues:

| Relación | Cuándo usar | Ejemplo |
|---|---|---|
| **Bloqueado por** | No puedo avanzar hasta que se cierre otro issue | "Setup Docker" bloquea a "Implementar CRUD" |
| **Bloquea a** | Mi issue impide que otro avance | — |
| **Issue padre** | Crear jerarquía épica → sub-issues | Pipeline IA (padre) → Captura video (hijo) |
| **Relacionado con** | Vincular issues que comparten contexto | — |

---

## Tutorial: Cómo crear un Issue correctamente

### Paso a paso

1. **Ve al repositorio** correspondiente y haz clic en **"New issue"**
2. **Escribe el título** de forma clara y descriptiva
3. **Selecciona el Type** (Funcionalidad, Error, Tarea, Documentación, Modelo IA, Testing)
4. **En la barra lateral derecha**, completa:
   - **Assignees**: ¿Quién lo va a hacer? (@ezekim-dev o @BenjaminValdesValdes)
   - **Prioridad**: ¿Qué tan urgente es? (Urgent / High / Medium / Low)
   - **Módulo**: ¿Qué parte del sistema afecta? (Nodo Central / Nodo Edge / Ambos / General)
   - **Área**: ¿Es infraestructura o seguridad? Si no, dejar en General
   - **Milestone**: ¿A qué entrega pertenece? (v0.1, v0.2, v1.0)
5. **Agrega al Project** "ALPR Access System — Sprints & Roadmap" y configura:
   - **Status**: ¿En qué estado empieza? (Backlog o Por Hacer)
   - **Iteración**: ¿En qué sprint se trabajará?
   - **Tamaño**: ¿Qué tan grande es la tarea? (XS, S, M, L, XL)
6. **Establece relaciones** si aplica:
   - ¿Depende de otro issue? → "Bloqueado por #X"
   - ¿Es parte de algo más grande? → "Add parent"

### Ejemplo completo

```
Título:  Implementar captura de video RTSP con OpenCV
Type:    Funcionalidad

── Barra lateral ──
Assignees:    @BenjaminValdesValdes
Prioridad:    High
Módulo:       Nodo Edge
Área:         General
Milestone:    v0.1 - Pipeline Base

── Dentro del Project ──
Status:       Por Hacer
Iteración:    Sprint 1
Tamaño:       L

── Relaciones ──
Bloqueado por: alpr-edge-node#1 (Setup Docker Compose)
```

---

## Referencia rápida

### ¿Dónde se configura cada cosa?

| Dato | Herramienta | Dónde se configura |
|---|---|---|
| Tipo de trabajo | Issue Type | Org → Settings → Planning → Issue types |
| Prioridad | Issue Field | Org → Settings → Planning → Issue fields |
| Módulo | Issue Field | Org → Settings → Planning → Issue fields |
| Área | Issue Field | Org → Settings → Planning → Issue fields |
| Estado del flujo | Project Field | Project → Settings → Fields |
| Sprint actual | Project Field | Project → Settings → Fields |
| Tamaño estimado | Project Field | Project → Settings → Fields |
| Fecha de entrega | Milestone | Repo → Issues → Milestones |
| Dependencias | Relationship | Dentro del issue → Relationships |
| Asignación | Assignees | Dentro del issue → Assignees |
| Labels | ~~No se usan~~ | — |

### Distribución del equipo

| Persona | Rol | Módulo principal | Repositorios |
|---|---|---|---|
| @ezekim-dev (Ezequiel) | PM + Desarrollo Panel | Nodo Central | `alpr-admin-panel`, `alpr-documentacion`, `Tesis` |
| @BenjaminValdesValdes (Benjamin) | Desarrollo IA + Edge | Nodo Edge | `alpr-edge-node`, `alpr-documentacion` |

---

> **¿Dudas?** Consulta al PM (Ezequiel) o revisa el contexto general del proyecto en
> [`GEMINI.md`](../GEMINI.md).
