# Labels de GitHub — Guía de Uso

> Aplica estos labels en todos los repositorios de la organización `alpr-access-system`.
> El objetivo es mantener consistencia entre `alpr-admin-panel` y `alpr-edge-node`.

---

## Configuración de Labels

Crea estos labels en cada repositorio (Settings → Labels → New label).

---

## 🏷️ Labels por Tipo de Trabajo

### `feat` — Feature / Funcionalidad
**Color:** `#0075ca` (azul)

Se usa cuando el issue describe una **nueva funcionalidad** que debe implementarse.
El issue debe describir qué debe hacer el sistema, no cómo implementarlo.

**Ejemplos:**
- `feat` | Implementar módulo de registro de funcionarios
- `feat` | Agregar endpoint de consulta de historial de accesos
- `feat` | Vista de dashboard con últimos 10 registros de acceso

---

### `bug` — Error / Defecto
**Color:** `#d73a4a` (rojo)

Se usa cuando el issue reporta un **comportamiento incorrecto** del sistema.
Debe incluir pasos para reproducir, comportamiento esperado y comportamiento actual.

**Ejemplos:**
- `bug` | El pipeline falla cuando la cámara no detecta ningún vehículo por > 5 s
- `bug` | El token JWT expira sin renovación automática
- `bug` | La sincronización offline no persiste registros con acentos en el nombre

---

### `docs` — Documentación
**Color:** `#0052cc` (azul oscuro)

Se usa cuando el issue es **exclusivamente de documentación**, sin cambios de código.
Puede ser documentación técnica (README, ADRs, specs) o comentarios de código.

**Ejemplos:**
- `docs` | Completar ADR-001 con la justificación de FastAPI
- `docs` | Agregar docstrings al módulo de validación de patentes
- `docs` | Actualizar README con instrucciones de setup con Docker

---

### `model` — Modelo de IA / ML
**Color:** `#7057ff` (violeta)

Se usa para issues relacionados exclusivamente con el **modelo de inteligencia artificial**:
entrenamiento, métricas, datasets, preprocesamiento, evaluación o ajuste fino.

**Ejemplos:**
- `model` | Preparar dataset con 500 imágenes de patentes con sombra
- `model` | Evaluar YOLOv12s vs YOLOv10 en dataset de validación
- `model` | Ajustar umbral de confianza mínimo para reducir falsos positivos

---

### `infra` — Infraestructura / DevOps
**Color:** `#e4e669` (amarillo)

Se usa para issues de **infraestructura, configuración de entornos, CI/CD, Docker,
despliegue o cualquier cosa relacionada con la operación del sistema**, no con su lógica.

**Ejemplos:**
- `infra` | Crear `docker-compose.yml` para desarrollo local del Nodo Central
- `infra` | Configurar GitHub Actions para lint automático en PRs
- `infra` | Configurar Coolify para deploy automático desde rama `main`

---

### `edge` — Nodo de Borde (Edge)
**Color:** `#f9d0c4` (salmón)

**Label de contexto.** Se usa para identificar que el issue pertenece o afecta al
**Nodo de Borde** (pipeline de IA, SQLite, dashboard local, cámara). Siempre se combina
con otro label de tipo.

**Uso:** Siempre combinado. Ejemplos:
- `feat` + `edge` | Implementar votación temporal para estabilizar lecturas de OCR
- `bug` + `edge` | El pipeline no libera el recurso de la cámara al cerrar
- `infra` + `edge` | Dockerizar el pipeline de inferencia para PC Windows

---

### `central` — Nodo Central (Cloud)
**Color:** `#bfd4f2` (azul claro)

**Label de contexto.** Se usa para identificar que el issue pertenece o afecta al
**Nodo Central** (React SPA, FastAPI, PostgreSQL, deploy en Coolify). Siempre combinado.

**Uso:** Siempre combinado. Ejemplos:
- `feat` + `central` | Implementar autenticación con JWT en la API
- `bug` + `central` | La paginación del historial devuelve resultados duplicados
- `infra` + `central` | Configurar volumen persistente de PostgreSQL en Docker

---

## 🏷️ Labels de Prioridad

### `prioridad:alta`
**Color:** `#e11d48` (rojo intenso)

El issue **bloquea el sprint** o es dependencia directa de otros issues críticos.
Debe resolverse en el sprint actual, sin excepción.

---

### `prioridad:media`
**Color:** `#f97316` (naranja)

El issue es importante pero **no bloquea** el avance inmediato. Debe resolverse en el
sprint actual o a más tardar en el siguiente.

---

### `prioridad:baja`
**Color:** `#84cc16` (verde)

El issue es deseable pero puede postergarse. Se trabaja cuando no hay issues de mayor
prioridad asignados.

---

## 🏷️ Labels de Estado Especial

### `needs-discussion`
**Color:** `#cfd3d7` (gris)

El issue tiene ambigüedad en los requerimientos o requiere una decisión de diseño
antes de poder implementarse. **No iniciar implementación** sin resolver la discusión.

---

### `blocked`
**Color:** `#000000` (negro)

El issue **no puede avanzar** porque depende de otro issue o factor externo sin resolver.
Siempre agregar un comentario en el issue explicando qué lo bloquea y con referencia al
issue bloqueante (ej. `Bloqueado por #23`).

---

## Combinaciones Más Comunes

| Situación | Labels a usar |
|---|---|
| Nueva funcionalidad en el panel web | `feat` + `central` + `prioridad:alta/media` |
| Error en el pipeline de IA | `bug` + `edge` + `prioridad:alta` |
| Configurar Docker para desarrollo | `infra` + `edge` o `central` |
| Entrenar o ajustar el modelo | `model` + `prioridad:alta/media` |
| Documentar una decisión de arquitectura | `docs` |
| Issue que no está claro | cualquier tipo + `needs-discussion` |

---

## Regla General

> **Todo issue activo debe tener al menos:**
> 1. Un label de **tipo** (`feat`, `bug`, `docs`, `model`, `infra`)
> 2. Un label de **contexto** (`edge`, `central`) — si aplica
> 3. Un label de **prioridad** (`prioridad:alta`, `prioridad:media`, `prioridad:baja`)
