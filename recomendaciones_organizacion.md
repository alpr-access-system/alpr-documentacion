# Lineamientos de Organización — ALPR Access System

> **Documento vivo.** Registra las decisiones de organización, estructura y flujo de trabajo
> tomadas para el proyecto de tesis. Actualizar cuando se tomen nuevas decisiones.
>
> **Última actualización:** 2026-08-13

---

## 1. Estructura de la Organización GitHub ✅

**Decisión:** Usar una **Organización de GitHub** (`alpr-access-system`) con repositorios
separados por componente.

**Justificación:**
- El sistema tiene dos plataformas con tecnologías, ciclos de vida y deploy completamente distintos.
- Aislar los repos evita mezclar historial de commits, CI/CD y dependencias.
- La organización permite invitar colaboradores con permisos granulares.
- Escala bien si en el futuro se agregan más componentes (ej. repo de datasets).

### Repositorios (todos creados ✅)

| Repositorio | Propósito | Estado |
|---|---|---|
| [`alpr-documentacion`](https://github.com/alpr-access-system/alpr-documentacion) | Documentación central, ADRs, specs, contextos de agente | 🟢 Activo |
| [`alpr-admin-panel`](https://github.com/alpr-access-system/alpr-admin-panel) | Nodo Central: React + FastAPI + PostgreSQL | 🟡 En setup |
| [`alpr-edge-node`](https://github.com/alpr-access-system/alpr-edge-node) | Nodo de Borde: Python + YOLOv12s + EasyOCR + SQLite | 🟡 En setup |
| `alpr-ml-training` *(futuro)* | Datasets y notebooks de entrenamiento del modelo | ⏳ Pendiente |

---

## 2. GitHub Projects — Tablero Kanban ✅

**Decisión:** Un único **GitHub Project a nivel de organización** que agrupa issues de todos los repos.

**Justificación:** Con un equipo de 2 personas y 2 repos activos, un tablero unificado evita
tener que revisar múltiples tableros por separado y facilita la priorización del sprint.

**Configuración del tablero:**
```
[ Backlog ] → [ Sprint Actual ] → [ En Progreso ] → [ En Revisión ] → [ Completado ]
```

**Acceso:** [GitHub Projects — Organización](https://github.com/orgs/alpr-access-system/projects)

---

## 3. Estructura del Repositorio de Documentación ✅

```
alpr-documentacion/
├── README.md                          # Overview del proyecto y links a repos
├── GEMINI.md                          # Contexto de agente IA (cargado automáticamente)
├── main.pdf                           # Tesis completa (Fase 1)
├── cronograma_general.pdf
├── cronograma_detallado.pdf
│
├── architecture/                      # Decisiones de arquitectura (ADRs)
│   ├── system-architecture.md         # ✅ Diagrama C4 completo en Mermaid
│   ├── _ADR-001-stack-central.md      # ⏳ Stack Nodo Central
│   ├── _ADR-002-stack-edge.md         # ⏳ Stack Nodo de Borde
│   └── _ADR-003-docker-infra.md       # ✅ Decisión Docker (documentada)
│
├── specs/                             # Especificaciones funcionales
│   ├── _admin-panel-spec.md           # ⏳ Spec panel administrativo
│   └── _edge-node-spec.md             # ⏳ Spec nodo de borde
│
├── contributing/                      # Guías para contribuidores
│   ├── labels-guide.md                # ✅ Labels y cuándo usar cada uno
│   └── _workflow-guide.md             # ⏳ Flujo de trabajo y convenciones
│
└── planning/                          # Planificación
    └── _cronograma-implementacion.md  # ⏳ Sprints (por definir en conjunto)
```

> **Convención:** Los archivos con prefijo `_` están **pendientes de completar**.

---

## 4. Contexto de Agente IA ✅

**Decisión:** Usar `GEMINI.md` en la raíz del repo de documentación como fuente de verdad
para el contexto de todos los agentes IA del proyecto.

**¿Cómo funciona?**
- Al abrir cualquier repo de la organización con Antigravity (u otro agente), el archivo
  `GEMINI.md` se carga automáticamente, dando al agente el contexto completo del proyecto
  sin necesidad de explicarlo cada vez.
- El compañero desarrollador puede empezar a trabajar con IA inmediatamente con el mismo
  contexto, sin necesidad de que el PM le pase el contexto manualmente.

**Archivos de contexto:**
- [`GEMINI.md`](./GEMINI.md) — Contexto completo del proyecto (stack, arquitectura, restricciones, equipo)

**Recomendación:** Copiar o hacer referencia al `GEMINI.md` de documentación en cada repo
de código (`alpr-admin-panel` y `alpr-edge-node`) para que el contexto viaje con cada repo.

---

## 5. Infraestructura con Docker ✅

**Decisión:** Todos los servicios del proyecto se contendrizan con **Docker**.

**Justificación:**
- El servidor de producción ya corre Ubuntu + Coolify (Docker nativo).
- Elimina el problema de "funciona en mi máquina" entre los dos desarrolladores.
- El dev principal puede levantar todo con `docker-compose up` sin configuración manual.
- La migración futura a NVIDIA Jetson Orin Nano solo requiere cambiar la imagen base.

**Detalle completo en:** [`architecture/_ADR-003-docker-infra.md`](./architecture/_ADR-003-docker-infra.md)

---

## 6. Labels de GitHub ✅

**Decisión:** Sistema de labels consistente aplicado en todos los repos de la organización.

### Labels de Tipo
| Label | Color | Cuándo usarlo |
|---|---|---|
| `feat` | 🔵 Azul | Nueva funcionalidad |
| `bug` | 🔴 Rojo | Comportamiento incorrecto |
| `docs` | 🔵 Azul oscuro | Solo documentación |
| `model` | 🟣 Violeta | IA / ML / datasets / entrenamiento |
| `infra` | 🟡 Amarillo | Docker, CI/CD, deploy, configuración |

### Labels de Contexto (siempre combinados con tipo)
| Label | Color | Cuándo usarlo |
|---|---|---|
| `edge` | 🟠 Salmón | Afecta al Nodo de Borde (pipeline IA, SQLite, cámara) |
| `central` | 🔵 Azul claro | Afecta al Nodo Central (React, FastAPI, PostgreSQL) |

### Labels de Prioridad
| Label | Color | Cuándo usarlo |
|---|---|---|
| `prioridad:alta` | 🔴 Rojo intenso | Bloquea el sprint |
| `prioridad:media` | 🟠 Naranja | Importante, no bloqueante |
| `prioridad:baja` | 🟢 Verde | Deseable, puede postergarse |

### Labels de Estado Especial
| Label | Color | Cuándo usarlo |
|---|---|---|
| `needs-discussion` | ⬜ Gris | Requiere decisión antes de implementar |
| `blocked` | ⬛ Negro | Bloqueado por dependencia externa |

**Guía detallada en:** [`contributing/labels-guide.md`](./contributing/labels-guide.md)

---

## 7. Cronograma de Implementación ⏳

**Estado:** Pendiente — Se definirá en conjunto analizando el `cronograma_detallado.pdf`.

**Placeholder en:** [`planning/_cronograma-implementacion.md`](./planning/_cronograma-implementacion.md)

---

## 8. Equipo y Roles

| Persona | Rol | Responsabilidad |
|---|---|---|
| PM (tú) | Product Manager | Gestión de producto, documentación, coordinación, trabajo con IA. Disponibilidad parcial (práctica profesional). |
| Dev Principal (compañero) | Developer | Implementación de código, entrenamiento del modelo, infraestructura. Disponibilidad principal. |

---

## Decisiones Pendientes

- [ ] Definir versiones exactas de stack (Python 3.11 o 3.12, React 18 o 19, etc.)
- [ ] Definir cronograma de implementación con sprints reales
- [ ] Completar ADR-001 (stack central) y ADR-002 (stack edge)
- [ ] Completar specs funcionales (`_admin-panel-spec.md`, `_edge-node-spec.md`)
- [ ] Definir convenciones de código y commits (`_workflow-guide.md`)
- [ ] Crear GitHub Project en la organización y cargar issues del Sprint 1
- [ ] Copiar/referenciar `GEMINI.md` en repos de código
