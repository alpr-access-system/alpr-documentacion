# 📋 ALPR Access System — Documentación

> **Repositorio de documentación central** de la organización [alpr-access-system](https://github.com/alpr-access-system).  
> Tesis de Título — Universidad Tecnológica Metropolitana (UTEM) — Campus Ñuñoa.

---

## ¿Qué es este proyecto?

Sistema de control de acceso vehicular automatizado para el Campus Ñuñoa de la UTEM, basado en reconocimiento de placas patentes chilenas mediante Inteligencia Artificial (ALPR — Automatic License Plate Recognition).

El sistema automatiza el ingreso de funcionarios eliminando la dependencia del personal de conserjería para tareas repetitivas de apertura de portón, mediante un pipeline de visión computacional en tiempo real con latencia < 100 ms.

---

## Repositorios de la Organización

| Repositorio | Descripción | Estado |
|---|---|---|
| [`alpr-documentacion`](https://github.com/alpr-access-system/alpr-documentacion) | Este repo. Documentación central, ADRs, especificaciones y lineamientos. | 🟢 Activo |
| [`alpr-admin-panel`](https://github.com/alpr-access-system/alpr-admin-panel) | Nodo Central: Panel administrativo web (React + FastAPI + PostgreSQL) | 🟡 En setup |
| [`alpr-edge-node`](https://github.com/alpr-access-system/alpr-edge-node) | Nodo de Borde: Pipeline de IA local (Python + YOLOv12s + EasyOCR + SQLite) | 🟡 En setup |
| [`alpr-ml-training`](https://github.com/alpr-access-system/alpr-ml-training) | Datasets y notebooks de entrenamiento del modelo *(futuro)* | ⏳ Pendiente |

---

## Estructura del Repositorio

```
alpr-documentacion/
├── README.md                        # Este archivo
├── GEMINI.md                        # Contexto de agente IA (para todos los colaboradores)
├── main.pdf                         # Tesis completa (Fase 1 — Investigación)
├── cronograma_general.pdf
├── cronograma_detallado.pdf
│
├── architecture/                    # Decisiones de arquitectura (ADRs)
│   ├── system-architecture.md       # Diagrama C4 y descripción general
│   ├── ADR-001-stack-central.md     # Stack del Nodo Central
│   ├── ADR-002-stack-edge.md        # Stack del Nodo de Borde
│   └── _ADR-003-docker-infra.md     # Decisión de infraestructura con Docker
│
├── specs/                           # Especificaciones funcionales
│   ├── _admin-panel-spec.md         # Spec del panel administrativo
│   └── _edge-node-spec.md           # Spec del nodo de borde
│
├── contributing/                    # Guías para contribuidores
│   ├── labels-guide.md              # Labels de GitHub y cuándo usar cada uno
│   └── _workflow-guide.md           # Flujo de trabajo y convenciones
│
└── planning/                        # Planificación de sprints
    └── _cronograma-implementacion.md # Cronograma de implementación (⏳ Pendiente)
```

> **Convención:** Los archivos con prefijo `_` están **pendientes de completar**.

---

## Equipo

| Rol | Responsabilidad Principal |
|---|---|
| Product Manager | Definición de requerimientos, documentación, coordinación y gestión con IA |
| Dev Principal | Implementación de código, entrenamiento del modelo, infraestructura |

---

## Guía Rápida para Colaboradores

1. Leer [`GEMINI.md`](./GEMINI.md) — contexto del proyecto para agentes IA.
2. Revisar [`architecture/system-architecture.md`](./architecture/system-architecture.md) — visión técnica general.
3. Ver el [**GitHub Project**](https://github.com/orgs/alpr-access-system/projects) de la organización — tablero de sprints.
4. Consultar [`contributing/labels-guide.md`](./contributing/labels-guide.md) — cómo etiquetar issues.

---

## Estado del Proyecto

```
[Fase 1] Investigación y Diseño    ████████████████████ 100% ✅
[Fase 2] Implementación Software   ░░░░░░░░░░░░░░░░░░░░   0% 🔄 ACTUAL
[Fase 3] Entrenamiento Modelo IA   ░░░░░░░░░░░░░░░░░░░░   0% ⏳
[Fase 4] Evaluación y Pruebas      ░░░░░░░░░░░░░░░░░░░░   0% ⏳
[Fase 5] Despliegue y Entrega      ░░░░░░░░░░░░░░░░░░░░   0% ⏳
```
