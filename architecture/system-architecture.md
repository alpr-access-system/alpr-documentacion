# Arquitectura del Sistema ALPR — UTEM Campus Ñuñoa

> **Estado:** Diseño finalizado (Fase 1). Base para la implementación (Fase 2).  
> **Fuente:** `main.pdf` — Capítulo 3: Desarrollo de la Solución

---

## Visión General

El sistema implementa una **Topología Distribuida** con dos nodos físicamente separados
que se comunican de forma asíncrona. Esta separación es estratégica: aísla la carga
de procesamiento de IA (continua e intensiva) de las tareas administrativas cotidianas.

---

## Diagrama de Contexto (Nivel 1)

> ¿Quién usa el sistema y cómo interactúa con él?

```mermaid
flowchart TD
    Admin["👤 Administrador / Conserje\n(Gestiona usuarios y registros)"]
    Funcionario["🚗 Funcionario UTEM\n(Se aproxima con su vehículo)"]
    ALPR["🧠 Sistema ALPR\n(Reconoce patentes y controla acceso)"]
    Porton["🚪 Portón Vehicular\n(Hardware de apertura)"]

    Admin -->|"Gestiona usuarios y registros (HTTPS)"| ALPR
    Funcionario -->|"Se aproxima — video capturado por cámara"| ALPR
    ALPR -->|"Abre / Cierra (señal eléctrica)"| Porton
```

---

## Diagrama de Contenedores (Nivel 2)

> ¿Qué componentes de software componen el sistema?

```mermaid
flowchart LR
    subgraph Central["☁️ Nodo Central — Nube"]
        Web["🖥️ Panel Web\nReact + TypeScript\n(SPA de administración)"]
        API["⚙️ API Central\nPython + FastAPI\n(Lógica de negocio)"]
        DB[("🗄️ Base de Datos Maestra\nPostgreSQL")]
    end

    subgraph Edge["🏠 Nodo de Borde — Portería / PC Prototipo"]
        Pipeline["🤖 Pipeline de IA\nPython + YOLOv12s + EasyOCR\n(Detección + OCR + Validación)"]
        Dashboard["📊 Dashboard Local\nReact lite\n(Visualización en tiempo real)"]
        Cache[("💾 Caché Local\nSQLite\n(Patentes + Registros offline)")]
        Camera["📷 Cámara IP\nStream RTSP"]
    end

    Web -->|"REST / HTTPS"| API
    API -->|"SQL"| DB
    Camera -->|"RTSP / OpenCV"| Pipeline
    Pipeline -->|"Lee / Escribe"| Cache
    Dashboard -->|"HTTP local"| Pipeline
    API <-->|"Sincronización asíncrona\n(cuando hay red)"| Pipeline
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

## Flujo del Pipeline de IA (Nodo Edge)

```mermaid
flowchart TD
    A["📷 Cámara IP\nStream RTSP"] --> B["OpenCV\nCaptura de frame"]
    B --> C["Preprocesamiento\nFiltros de mejora de imagen"]
    C --> D["YOLOv12s\nDetección de placa"]
    D --> E{¿Placa detectada?}
    E -->|No| B
    E -->|Sí| F["EasyOCR\nExtracción de texto"]
    F --> G["Validación Lógica\nConsulta SQLite local"]
    G --> H{¿Patente Autorizada?}
    H -->|Sí| I["✅ Apertura\nIndicador Visual"]
    H -->|No| J["❌ Denegación\nIndicador Visual"]
    I --> K["Registrar en SQLite"]
    J --> K
    K --> L{¿Hay conexión\na red?}
    L -->|Sí| M["Sincronizar con\nAPI Central"]
    L -->|No| N["Registro pendiente\nse sincroniza luego"]
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

## Estructura Interna del Nodo de Borde

El Nodo de Borde vive en **un solo repositorio** (`alpr-edge-node`) con dos módulos bien separados:

```
alpr-edge-node/
├── pipeline/          # 🤖 Backend IA (Python)
│   ├── detector.py    # YOLOv12s — detección de placa
│   ├── ocr.py         # EasyOCR — extracción de texto
│   ├── validator.py   # Validación contra SQLite
│   ├── sync.py        # Sincronización con API Central
│   └── main.py        # Loop principal
│
├── dashboard/         # 📊 Frontend local (React)
│   └── src/           # Visualización en tiempo real (bounding boxes, estado)
│
├── data/
│   └── local.db       # SQLite (volumen Docker persistente)
│
└── docker-compose.yml # Levanta pipeline + dashboard juntos
```

**Razón:** Son dos procesos del mismo nodo físico (la portería), comparten recursos de red local y se despliegan juntos con `docker-compose`. Un repositorio separado agregaría complejidad de sincronización innecesaria.

---

## Infraestructura de Deploy (Prototipo)

```mermaid
flowchart TB
    subgraph Nube["☁️ Internet — Servidor Ubuntu Homelab"]
        Coolify["Coolify + Traefik"]
        subgraph AdminPanel["alpr-admin-panel"]
            FastAPI["FastAPI\n:8000"]
            ReactBuild["React (nginx)"]
            Postgres[("PostgreSQL")]
        end
        Coolify --> AdminPanel
    end

    subgraph PC["💻 PC de Simulación — Local"]
        DockerDesktop["Docker Desktop"]
        subgraph EdgeNode["alpr-edge-node"]
            PipelineContainer["Pipeline IA\n(Python + YOLO)"]
            DashboardContainer["Dashboard React\nlocalhost:3000"]
            SQLiteVol[("SQLite\n(volumen local)")]
            WebcamUSB["📷 Webcam USB\n(simulación cámara)"]
        end
        DockerDesktop --> EdgeNode
    end

    PipelineContainer <-->|"Sync HTTP\n(cuando hay red)"| FastAPI
```

---

## Decisiones de Arquitectura (ADRs)

| ADR | Título | Estado |
|---|---|---|
| [ADR-001](./_ADR-001-stack-central.md) | Stack tecnológico del Nodo Central | ⏳ Por documentar |
| [ADR-002](./_ADR-002-stack-edge.md) | Stack tecnológico del Nodo de Borde | ⏳ Por documentar |
| [ADR-003](./_ADR-003-docker-infra.md) | Uso de Docker como infraestructura base | ✅ Documentado |
