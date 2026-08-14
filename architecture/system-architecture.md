# Arquitectura del Sistema ALPR — UTEM Campus Ñuñoa

> **Estado:** Diseño finalizado (Fase 1). Base para la implementación (Fase 2).  
> **Fuente:** `main.pdf` — Capítulo 3: Desarrollo de la Solución

---

## Visión General

El sistema implementa una **Topología Distribuida** con dos nodos físicamente separados
que se comunican de forma asíncrona. Esta separación es estratégica: aísla la carga
de procesamiento de IA (continua e intensiva) de las tareas administrativas cotidianas.

```mermaid
C4Context
  title Sistema ALPR — Diagrama de Contexto (C4 Nivel 1)
  
  Person(admin, "Administrador / Conserje", "Gestiona usuarios y visualiza registros")
  Person(funcionario, "Funcionario UTEM", "Se aproxima con su vehículo")
  
  System(alpr, "Sistema ALPR", "Reconoce patentes y controla acceso vehicular automáticamente")
  
  System_Ext(porton, "Portón Vehicular", "Hardware de apertura/cierre")
  
  Rel(admin, alpr, "Gestiona usuarios y registros", "HTTPS")
  Rel(funcionario, alpr, "Se aproxima (video capturado)", "Cámara IP")
  Rel(alpr, porton, "Abre/Cierra", "Señal eléctrica (relé)")
```

---

## Diagrama de Contenedores (C4 Nivel 2)

```mermaid
C4Container
  title Sistema ALPR — Contenedores (Nube y Borde)

  Container_Boundary(central, "Nodo Central (Nube)") {
    Container(web, "Panel Web", "React + TypeScript", "SPA de administración. Gestión de funcionarios, vehículos y registros históricos.")
    Container(api, "API Central", "Python + FastAPI", "API RESTful. Lógica de negocio, autenticación y sincronización.")
    ContainerDb(db, "Base de Datos Maestra", "PostgreSQL", "Registro maestro de funcionarios, vehículos y accesos.")
  }

  Container_Boundary(edge, "Nodo de Borde (Portería / PC Prototipo)") {
    Container(pipeline, "Pipeline de IA", "Python + YOLOv12s + EasyOCR", "Captura video, detecta patentes, extrae texto, valida acceso.")
    Container(dashboard, "Dashboard Local", "React (lite)", "Visualización en tiempo real para demostración y validación técnica.")
    ContainerDb(cache, "Caché Local", "SQLite", "Lista de patentes autorizadas. Registro offline de accesos.")
    Container(camera, "Cámara IP", "Stream RTSP", "Captura video del acceso vehicular.")
  }

  Rel(web, api, "Peticiones REST", "HTTPS/JSON")
  Rel(api, db, "Consultas", "SQL/TCP")
  Rel(pipeline, cache, "Lee patentes / Escribe registros", "SQL local")
  Rel(pipeline, camera, "Captura frames", "RTSP/OpenCV")
  Rel(dashboard, pipeline, "Estado en tiempo real", "HTTP local")
  Rel(api, pipeline, "Sincronización asíncrona", "HTTP/REST (cuando hay red)")
```

---

## Modelo de Datos (Entidad-Relación)

```mermaid
erDiagram
    FUNCIONARIO {
        int id PK
        string nombre
        string rut
        string email
        bool activo
        datetime created_at
    }
    
    VEHICULO {
        int id PK
        string patente
        string tipo
        int funcionario_id FK
        bool activo
        datetime created_at
    }
    
    REGISTRO_ACCESO {
        int id PK
        int vehiculo_id FK
        datetime timestamp
        string resultado
        float confianza_modelo
        string origen_nodo
        bool sincronizado
        string imagen_captura
    }

    FUNCIONARIO ||--o{ VEHICULO : "posee"
    VEHICULO ||--o{ REGISTRO_ACCESO : "genera"
```

---

## Flujo de Procesamiento del Pipeline (Nodo Edge)

```mermaid
flowchart TD
    A[📷 Cámara IP\nStream RTSP] --> B[OpenCV\nCaptura de frame]
    B --> C[Preprocesamiento\nFiltros de mejora de imagen]
    C --> D[YOLOv12s\nDetección de placa]
    D --> E{¿Placa\ndetectada?}
    E -->|No| B
    E -->|Sí| F[EasyOCR\nExtracción de texto]
    F --> G[Validación Lógica\nConsulta SQLite local]
    G --> H{¿Patente\nAutorizada?}
    H -->|Sí| I[✅ Apertura\nIndicador Visual]
    H -->|No| J[❌ Denegación\nIndicador Visual]
    I --> K[Registrar en SQLite]
    J --> K
    K --> L{¿Hay conexión\na red?}
    L -->|Sí| M[Sincronizar con\nAPI Central]
    L -->|No| N[Registro pendiente\nse sincroniza luego]
    M --> B
    N --> B
```

---

## Restricciones de Latencia

| Etapa | Tiempo objetivo |
|---|---|
| Captura + Preprocesamiento | < 10 ms |
| Inferencia YOLOv12s | < 30 ms |
| OCR EasyOCR | < 40 ms |
| Validación en SQLite | < 5 ms |
| **Total pipeline (end-to-end)** | **< 100 ms** |

---

## Componentes por Repositorio

### [`alpr-admin-panel`](https://github.com/alpr-access-system/alpr-admin-panel)
- Frontend React SPA
- Backend FastAPI
- Base de datos PostgreSQL
- Docker Compose para desarrollo y producción
- Deploy en Coolify (homelab Ubuntu)

### [`alpr-edge-node`](https://github.com/alpr-access-system/alpr-edge-node)
- Pipeline Python (YOLO + EasyOCR + OpenCV)
- Dashboard React local (visualización en tiempo real)
- SQLite para caché local
- Docker Compose para ejecutar en PC/Jetson

---

## Infraestructura de Deploy (Prototipo)

```
┌─────────────────────────────────────────────────────┐
│                   Internet / Nube                   │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │         Servidor Ubuntu (Homelab)            │   │
│  │   Coolify + Traefik + Docker                 │   │
│  │                                             │   │
│  │  [alpr-admin-panel]                         │   │
│  │   └── FastAPI container (puerto 8000)       │   │
│  │   └── React build (servido por nginx)       │   │
│  │   └── PostgreSQL container                  │   │
│  └─────────────────────────────────────────────┘   │
│                      ▲                              │
│                      │ Sync HTTP                    │
└──────────────────────┼──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│              PC de Simulación (Local)               │
│         Docker Desktop + docker-compose             │
│                                                     │
│  [alpr-edge-node]                                   │
│   └── Pipeline IA container (Python)                │
│   └── Dashboard React (localhost:3000)              │
│   └── SQLite (volumen local)                        │
│   └── Cámara: webcam USB (simulación)               │
└─────────────────────────────────────────────────────┘
```

---

## Decisiones de Arquitectura (ADRs)

| ADR | Título | Estado |
|---|---|---|
| [ADR-001](./ADR-001-stack-central.md) | Stack tecnológico del Nodo Central | ⏳ Por documentar |
| [ADR-002](./ADR-002-stack-edge.md) | Stack tecnológico del Nodo de Borde | ⏳ Por documentar |
| [ADR-003](./ADR-003-docker-infra.md) | Uso de Docker como infraestructura base | ⏳ Por documentar |
