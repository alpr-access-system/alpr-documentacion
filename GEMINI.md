# ALPR Access System — Contexto del Proyecto para Agentes IA

> **Lee esto primero.** Este archivo es el contexto base que deben cargar todos los agentes IA
> que trabajen en cualquier repositorio de la organización `alpr-access-system`.

---

## ¿Qué estamos construyendo?

Un sistema de **control de acceso vehicular automatizado** para el Campus Ñuñoa de la
**Universidad Tecnológica Metropolitana (UTEM)** en Santiago, Chile.

El sistema reconoce automáticamente las placas patentes de los funcionarios de la universidad
mediante Inteligencia Artificial (ALPR), eliminando la necesidad de que el personal de conserjería
abra manualmente el portón vehicular. El objetivo técnico es lograr una **latencia de apertura
inferior a 100 ms** con funcionamiento **Offline-First** (sin depender de internet).

---

## Arquitectura General (Topología Distribuida)

El sistema se divide en **dos nodos independientes** con sincronización asíncrona:

```
┌─────────────────────────────────────┐     Sync Asíncrona     ┌──────────────────────────────────────┐
│         NODO CENTRAL (Nube)         │ ◄──────────────────────► │       NODO DE BORDE (Edge/Local)      │
│                                     │                          │                                      │
│  ┌────────────┐  ┌────────────────┐ │                          │ ┌──────────────┐  ┌───────────────┐ │
│  │ React SPA  │  │  FastAPI REST  │ │                          │ │ Pipeline IA  │  │  React (local)│ │
│  │ (Frontend) │  │   (Backend)    │ │                          │ │ Python+YOLO  │  │  (Dashboard)  │ │
│  └────────────┘  └───────┬────────┘ │                          │ └──────┬───────┘  └───────────────┘ │
│                          │          │                          │        │                             │
│              ┌───────────▼────────┐ │                          │  ┌─────▼──────┐  ┌───────────────┐ │
│              │    PostgreSQL      │ │                          │  │   SQLite   │  │  Cámara IP    │ │
│              │  (Base de datos    │ │                          │  │ (Caché     │  │  (RTSP Feed)  │ │
│              │    maestra)        │ │                          │  │  local)    │  └───────────────┘ │
│              └───────────────────┘ │                          │  └────────────┘                     │
└─────────────────────────────────────┘                          └──────────────────────────────────────┘
```

---

## Stack Tecnológico Definitivo

### Nodo Central (repo: `alpr-admin-panel`)
| Capa | Tecnología | Notas |
|---|---|---|
| Frontend | React + TypeScript | SPA, comunicación con API REST |
| Backend | Python + FastAPI | API RESTful, operaciones async |
| Base de datos | PostgreSQL | Motor relacional maestro |
| Contenedores | Docker + Docker Compose | Deploy en servidor Ubuntu (Coolify) |
| Deploy | Coolify (autoalojado) + Traefik | Infraestructura homelab existente |

### Nodo de Borde (repo: `alpr-edge-node`)
| Capa | Tecnología | Notas |
|---|---|---|
| Pipeline IA | Python 3.11+ | Servicio principal de inferencia |
| Detección | YOLOv12s | Modelo de detección de placas patentes |
| OCR | EasyOCR | Extracción de texto de patentes |
| Captura de video | OpenCV | Procesamiento del stream RTSP |
| Caché local | SQLite | Operación Offline-First |
| Dashboard local | React (lite) | Visualización en tiempo real para demos |
| Hardware target | PC (prototipo) → NVIDIA Jetson Orin Nano (producción) |

---

## Decisiones de Infraestructura

### Docker: SÍ, para todo lo posible
- El servidor de producción ya corre Ubuntu con Docker vía **Coolify**.
- Tanto el backend del Nodo Central como el pipeline del Nodo de Borde se contendrizarán.
- El Nodo de Borde (en PC para prototipo) correrá sus servicios via `docker-compose`.
- Esto garantiza paridad entre entorno de desarrollo y producción.

### Entorno de desarrollo
- El equipo usa **Windows** como SO de desarrollo.
- Los contenedores Docker corren via **Docker Desktop** localmente.
- El deploy de producción va al servidor **Ubuntu del homelab** (via Coolify).

---

## Alcances y Restricciones Clave

> Estas restricciones son definitorias. No las ignores al generar código o arquitectura.

- **Solo patentes chilenas** — 5 formatos (1985 a hoy). No incluye motos ni vehículos especiales.
- **Prototipo en PC** — El nodo edge corre en PC normal, no en Jetson (eso es trabajo futuro).
- **Latencia objetivo** — < 100 ms en el nodo edge (validación local, no depende de red).
- **Offline-First** — El nodo edge debe funcionar aunque caiga internet (usa SQLite local).
- **Sincronización asíncrona** — Los registros locales del edge se sincronizan al central cuando hay red.
- **Sin actuadores físicos** — El prototipo muestra indicadores visuales de "Apertura/Cierre" (no controla motor real).

---

## Modelo de Datos (Conceptual)

```
Funcionario (id, nombre, rut, email, activo)
    │
    └─── Vehiculo (id, patente, tipo, funcionario_id, activo)
                │
                └─── RegistroAcceso (id, vehiculo_id, timestamp, resultado, 
                                     confianza_modelo, origen_nodo, sincronizado)
```

---

## Repositorios

| Repo | Stack | Deploy |
|---|---|---|
| `alpr-documentacion` | Markdown | GitHub (solo docs) |
| `alpr-admin-panel` | React + FastAPI + PostgreSQL | Docker en Coolify |
| `alpr-edge-node` | Python + YOLOv12s + SQLite + React | Docker en PC/Jetson |
| `alpr-ml-training` | Python + Jupyter | Local (GPU) |

---

## Metodología de Desarrollo

- **Gestión:** Scrum (sprints de 2 semanas) + **GitHub Projects** (tablero organización)
- **ML:** CRISP-DM para el ciclo de vida del modelo de IA
- **Equipo:** 2 personas
  - **PM:** Gestión de producto, documentación, coordinación, trabajo con IA
  - **Dev Principal:** Implementación de código, entrenamiento del modelo, infraestructura
- **Rama principal:** `main` (protegida). Desarrollo en ramas por feature: `feat/nombre-feature`

---

## Contexto Institucional

- **Institución:** Universidad Tecnológica Metropolitana (UTEM)
- **Campus:** Ñuñoa, Santiago, Chile
- **Problema:** El acceso vehicular de funcionarios depende de apertura manual por conserjería vía llamadas telefónicas.
- **Impacto económico del proyecto:** VAN $5.898.595 CLP, TIR 59.47%, Payback 1.55 años.

---

## Links Útiles

- [Tesis completa (main.pdf)](./main.pdf)
- [Cronograma General](./cronograma_general.pdf)
- [Cronograma Detallado](./cronograma_detallado.pdf)
- [GitHub Project (tablero)](https://github.com/orgs/alpr-access-system/projects)
- [Guía de Gestión de Proyecto](./contributing/project-management-guide.md)
- [Arquitectura del Sistema](./architecture/system-architecture.md)
