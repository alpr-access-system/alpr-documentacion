# Plan de Implementación — ALPR Access System
## Fase 2: Desarrollo e Implementación

> **Última actualización:** 2026-08-14  
> **Estado:** En ejecución  
> **Fecha de entrega final:** 2026-12-18

---

## Contexto: Retroalimentación del Profesor (2026-08-10)

El profesor entregó las siguientes observaciones que **impactan directamente el plan de desarrollo:**

| Observación | Impacto |
|---|---|
| *"Quisiera ver los prototipos en el documento"* | Cada sprint debe entregar un prototipo documentable y demostrable |
| *"¿Cómo entrenarán el modelo, cuántas patentes, cuáles son las medidas de éxito?"* | Requiere definir formalmente la estrategia de entrenamiento (ver Sección 4) |
| *"¿Qué sucede si la patente no es leída correctamente?"* | Gap en los casos de uso — requiere caso de uso de fallback/contingencia |
| *"El terreno requerirá adaptación para no entrar más autos de los que caben"* | Fuera del alcance del prototipo, pero debe documentarse como limitación |
| *"¿Qué tal la flexibilidad para otros estacionamientos?"* | Documentar capacidad multi-sitio de la arquitectura |

---

## Alineación Metodológica

### Cómo se integran CRISP-DM + Scrum + Prototipado Evolutivo

```
CRISP-DM Phase          Sprints              Prototipos Evolutivos
─────────────────────────────────────────────────────────────────────
Fase 1: Comprensión     [COMPLETADO]         Tesis Fase 1 (main.pdf)
del negocio             Sprint 0

Fase 2: Comprensión     Sprint 1             Prototipo v0.1: Dataset base
y preparación datos     Sprint 2             anotado + pipeline OpenCV

Fase 3: Modelado        Sprint 2             Prototipo v0.2: YOLO+OCR
(Entrenamiento IA)      Sprint 3             funcionando con dataset

Fase 4: Evaluación      Sprint 4             Prototipo v0.3: Sistema
y Pruebas               Sprint 5             integrado + métricas

Fase 5: Despliegue      Sprint 5             Prototipo v1.0: DEMO FINAL
                        Sprint 6             Deploy + documentación
```

**Regla de oro:** Los sprints de Scrum organizan *quién y cuándo*. CRISP-DM define *qué fases técnicas* ejecutar. En cada sprint habrá tareas de ambos mundos ejecutándose en paralelo.

---

## División de Trabajo por Repositorio

| Repositorio | Responsable | Rol en el sistema |
|---|---|---|
| `alpr-admin-panel` | **Ezequiel** | Panel web + API REST + PostgreSQL |
| `alpr-edge-node` | **Benjamin** | Pipeline IA + Dataset + Docker + Dashboard |
| `alpr-documentacion` | **Ezequiel** (PM + IA) | Specs, ADRs, cronograma, documentación |

> ⚠️ **Dependencia clave:** El Admin Panel es la base de prueba del modelo. Cuando Benjamin tenga el pipeline funcionando, necesita patentes registradas en el Admin Panel para validar. Por esto, el Admin Panel tiene **prioridad máxima en los primeros sprints**.

---

## Cronograma de Sprints

> Basado en el cronograma detallado de la tesis y ajustado con la retroalimentación del profesor.  
> **Periodicidad de entrega:** Semanal (para auditoría continua del equipo y avances al profesor).

---

### Sprint 0 — Planificación y Fundación (10/08 – 19/08/2026)

**CRISP-DM:** Transición Fase 1 → Fase 2  
**Prototipo Evolutivo:** No aplica (sprint de preparación)

#### Ezequiel
- [x] Crear estructura de documentación (`alpr-documentacion`)
- [x] Crear GitHub Project y 20 issues base
- [ ] Completar spec funcional del admin-panel (issue #4 docs)
- [ ] Definir estrategia del dataset (ver Sección 4 de este documento)
- [ ] Documentar caso de uso fallback (patente no legible) en spec

#### Benjamin
- [ ] Evaluar herramientas de anotación para YOLOv12 (issue: balanceo de clases)
- [ ] Setup Docker Compose base del nodo edge (issue #1 edge)
- [ ] Comenzar captura fotográfica de patentes en terreno UTEM

**Entregable de sprint:**
- Documento de estrategia de dataset completo
- Repos con estructura base inicializada
- 50+ imágenes iniciales de patentes capturadas en terreno

---

### Sprint 1 — Fundación Técnica + Dataset Base (19/08 – 02/09/2026)

**CRISP-DM:** Fase 2 — Comprensión y Preparación de Datos  
**Prototipo Evolutivo:** v0.1

#### Ezequiel (Admin Panel)
- [ ] FastAPI + PostgreSQL + Docker Compose funcionando
- [ ] React + TypeScript + Vite scaffold
- [ ] Autenticación JWT básica
- [ ] Modelo de datos + migraciones Alembic

#### Benjamin (Edge Node + Dataset)
- [ ] OpenCV capturando video de webcam en Docker
- [ ] Pipeline CLAHE + redimensión implementado
- [ ] 300+ imágenes de patentes capturadas (terreno + web)
- [ ] Dataset anotado con bounding boxes en formato YOLO
- [ ] Script de partición train/val/test (80/10/10)

**Entregable de sprint (Prototipo v0.1):**
> Admin Panel con login funcional accesible en localhost.  
> Video de webcam capturado y preprocesado mostrando frames en consola.  
> Dataset base anotado y particionado listo para entrenamiento.

---

### Sprint 2 — CRUD Core + Entrenamiento v1 (02/09 – 24/09/2026)

**CRISP-DM:** Fase 2 completa + Fase 3 inicio  
**Prototipo Evolutivo:** v0.2

#### Ezequiel (Admin Panel)
- [ ] CRUD Funcionarios completo (API + React)
- [ ] CRUD Vehículos completo (API + React) con validación de formatos de patente
- [ ] Primer funcionario y vehículo registrado en el sistema

#### Benjamin (Edge Node + IA)
- [ ] Script de entrenamiento YOLOv12s configurado en GPU
- [ ] Entrenamiento modelo base v1.0 ejecutado
- [ ] Módulo de inferencia sobre imágenes estáticas funcionando
- [ ] SQLite local inicializado con esquema base
- [ ] Integración inicial YOLO + EasyOCR (texto extraído de foto)

**Entregable de sprint (Prototipo v0.2):**
> Panel administrativo con CRUD funcionando: se pueden registrar funcionarios y sus vehículos.  
> Modelo YOLOv12s v1.0 capaz de detectar placas en imágenes estáticas (aunque baja precisión).  
> Primera lectura exitosa de texto de patente con EasyOCR.

---

### Sprint 3 — Pipeline en Tiempo Real + API REST (24/09 – 15/10/2026)

**CRISP-DM:** Fase 3 — Modelado completo  
**Prototipo Evolutivo:** v0.3

#### Ezequiel (Admin Panel)
- [ ] Historial de accesos con filtros (API + React)
- [ ] Endpoint `POST /accesos/sync` para recibir registros del edge
- [ ] Deploy del backend en Coolify (homelab)
- [ ] API accesible desde internet (no solo localhost)

#### Benjamin (Edge Node + IA)
- [ ] Pipeline completo en tiempo real: Cámara → YOLO → OCR → SQLite
- [ ] Validación de acceso contra lista blanca en SQLite
- [ ] Lógica de fallback: si patente no es legible en 3 segundos → alerta conserje
- [ ] Worker de sincronización edge → API Central

**Entregable de sprint (Prototipo v0.3):**
> **PRIMER PROTOTIPO INTEGRADO:** El edge node detecta una patente en tiempo real y la valida contra las registradas en el Admin Panel.  
> El resultado (autorizado/denegado/no legible) aparece en consola.  
> Los registros de acceso aparecen en el historial del panel web.

---

### Sprint 4 — Dashboard Local + Evaluación del Modelo (15/10 – 05/11/2026)

**CRISP-DM:** Fase 4 — Evaluación y Pruebas  
**Prototipo Evolutivo:** v0.4 → v0.5

#### Ezequiel (Admin Panel)
- [ ] Dashboard con métricas (últimos accesos, estadísticas del día)
- [ ] Vista de login y UX completa
- [ ] Módulo de gestión de vehículos con virtualización de listas React
- [ ] Pruebas de carga en la API (concurrencia)

#### Benjamin (Edge Node + Evaluación)
- [ ] Dashboard React local: video en tiempo real + bounding boxes
- [ ] Panel lateral: patente detectada, resultado, confianza del modelo
- [ ] **Evaluación formal del modelo:**
  - Calcular mAP@0.5, Precision, Recall, F1-Score
  - Medir latencia de cada etapa del pipeline
  - Análisis de errores OCR en caracteres similares (0/O, 1/I)
- [ ] Fine-tuning del modelo según resultados de evaluación

**Entregable de sprint (Prototipo v0.5):**
> Dashboard local funcionando con video en tiempo real y detecciones visuales.  
> Reporte de métricas del modelo documentado (mAP, latencia, errores OCR).  
> Sistema visible y demostrable end-to-end.

---

### Sprint 5 — Pruebas de Estrés + Optimización (05/11 – 23/11/2026)

**CRISP-DM:** Fase 4 completa  
**Prototipo Evolutivo:** v0.9

#### Ezequiel
- [ ] Pruebas de corte de internet (modo offline)
- [ ] Sincronización automática al reconectar

#### Benjamin
- [ ] Análisis de latencia total del pipeline (objetivo < 100ms)
- [ ] Conversión del modelo a ONNX/TensorRT para optimización
- [ ] Pruebas con múltiples vehículos simultáneos
- [ ] Corrección de bugs críticos

**Entregable de sprint (Prototipo v0.9):**
> Sistema completo funcionando con todas las métricas documentadas.  
> Prueba de corte de red exitosa: el edge sigue operando y sincroniza al reconectar.  
> Latencia medida y documentada.

---

### Sprint 6 — Deploy Final + Documentación (23/11 – 18/12/2026)

**CRISP-DM:** Fase 5 — Despliegue  
**Prototipo Evolutivo:** v1.0 (entrega final)

#### Ezequiel
- [ ] Deploy definitivo del Admin Panel en Coolify
- [ ] Documentación capítulo de resultados en la tesis
- [ ] Manual técnico de despliegue (Anexo)
- [ ] Revisión APA/IEEE

#### Benjamin
- [ ] Documentar arquitectura de migración a NVIDIA Jetson (trabajo futuro)
- [ ] Documentación capítulo de evaluación
- [ ] Video de demostración del prototipo

**Entregable final (2026-12-18):**
> Prototipo v1.0 funcionando. Documento de tesis actualizado con resultados reales.  
> Video de demostración. Deploy accesible.

---

## Estrategia de Entrenamiento del Modelo (Respuesta al Profesor)

### ¿Cuántas imágenes necesita el dataset?

| Subconjunto | Cantidad | Descripción |
|---|---|---|
| Captura en terreno UTEM | 300–400 | Fotos reales de patentes en la portería del campus |
| Captura web (datasets públicos chilenos) | 400–600 | Imágenes diversas de patentes de internet |
| **Total imágenes base** | **700–1000** | Antes de data augmentation |
| **Con augmentation (x4)** | **2800–4000** | Dataset final de entrenamiento |

### Técnicas de Data Augmentation
- Rotación ±15°
- Variación de brillo y contraste
- Blur gaussiano (simula movimiento o baja resolución)
- Ruido aleatorio
- Recorte y zoom

### Distribución del Dataset
```
train/   80%  → 560–800 imágenes originales (~2500 con augmentation)
val/     10%  → 70–100 imágenes
test/    10%  → 70–100 imágenes (nunca vistas durante entrenamiento)
```

### Criterios de Calidad de Imagen (ya definidos en tesis)
- Resolución mínima: 640×480 px
- Placa visible y legible por humano
- Ángulo de captura máximo: 30° horizontal
- Condiciones variadas: día/noche, sol, sombra, lluvia

### Métricas de Éxito del Modelo (Criterios de Aceptación)

| Métrica | Mínimo Aceptable | Objetivo |
|---|---|---|
| mAP@0.5 (detección de placa) | ≥ 80% | ≥ 90% |
| Precision | ≥ 85% | ≥ 92% |
| Recall | ≥ 80% | ≥ 88% |
| F1-Score | ≥ 82% | ≥ 90% |
| Latencia inferencia YOLO | < 40ms | < 30ms |
| Latencia OCR EasyOCR | < 50ms | < 40ms |
| **Latencia total pipeline** | **< 100ms** | **< 80ms** |
| Tasa de lectura correcta patente | ≥ 85% | ≥ 90% |

---

## Caso de Uso Faltante: Patente No Legible (Fallback)

> **Gap identificado por el profesor.** Este caso de uso debe agregarse al documento de tesis.

### Flujo de contingencia

```
1. Pipeline detecta vehículo en cámara
2. YOLOv12s no detecta placa con confianza suficiente (< umbral) 
   O EasyOCR no puede extraer texto válido
3. Sistema espera 3 segundos (configurable) y reintenta
4. Si persiste:
   → Registra evento "LECTURA_FALLIDA" en SQLite local
   → Dashboard local muestra alerta visual al operador
   → El conserje interviene usando el control remoto manual preexistente
   → El sistema registra que la apertura fue "manual/contingencia"
5. El registro se sincroniza al panel central con resultado = "manual"
```

**Resultado:** El funcionario nunca queda bloqueado. El sistema degradado funciona igual que el sistema manual actual.

---

## Respuestas a Observaciones del Profesor

### ¿El terreno requiere adaptación para delimitar espacios?
**Respuesta para el documento:** El sistema ALPR controla el acceso de entrada/salida pero no gestiona la ocupación de espacios de estacionamiento. Este control (detección de espacios libres) requeriría sensores adicionales (cámaras cenital o sensores ultrasónicos por plaza) y configura un sistema de guiado de parking distinto al alcance de este prototipo. Se documenta como **extensión futura de alto valor** dada la arquitectura modular del sistema.

### ¿Flexibilidad para otros estacionamientos?
**Respuesta para el documento:** La arquitectura distribuida permite escalar a múltiples accesos. Cada acceso físico adicional requeriere:
- Un nuevo Nodo de Borde (`alpr-edge-node` desplegado en otra PC/Jetson)
- Configuración del `CENTRAL_API_URL` apuntando al mismo Admin Panel
- Registro de los nuevos funcionarios/vehículos en el mismo panel central

No requiere cambios de código. Es una característica de diseño de la arquitectura.

---

## Cadencia de Entrega y Auditoría

- **Frecuencia de entrega:** Semanal (cada viernes)
- **Formato de avance:** Screenshot/video del prototipo actual + issues completados esa semana
- **Auditoría PM→Dev:** Ezequiel revisa en el GitHub Project los issues completados por Benjamin cada viernes
- **Comunicación al profesor:** Resumen quincenal del prototipo con evidencia (capturas, videos, métricas)
