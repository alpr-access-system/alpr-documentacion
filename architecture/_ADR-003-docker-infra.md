# ADR-003 — Uso de Docker como Infraestructura Base

> ✅ **DECISIÓN TOMADA** — Docker se usa en todos los servicios del proyecto.

---

## Contexto

El proyecto necesita:
1. **Paridad entre entornos**: evitar el clásico "funciona en mi máquina".
2. **Facilidad de deploy**: el servidor de producción ya corre Coolify (Docker).
3. **Equipo pequeño en Windows**: ambos desarrolladores usan Windows como SO principal.
4. **Dos nodos independientes**: el Nodo Central (nube) y el Nodo Edge (local) deben poder
   desplegarse de forma independiente con sus dependencias aisladas.

## Decisión

**Todos los servicios se contendrizarán con Docker.** Cada nodo tendrá su propio
`docker-compose.yml` para orquestar sus contenedores.

## Detalle por Nodo

### Nodo Central (`alpr-admin-panel`)

```yaml
# Servicios en docker-compose.yml
services:
  api:        # FastAPI (Python)
  frontend:   # React build servida por nginx (producción) o Vite dev server (desarrollo)
  db:         # PostgreSQL
```

- **Desarrollo local (Windows):** `docker-compose up` en Docker Desktop.
- **Producción:** Deploy automático desde Coolify apuntando a rama `main`.
- **Persistencia:** Volumen Docker para PostgreSQL data.

### Nodo de Borde (`alpr-edge-node`)

```yaml
# Servicios en docker-compose.yml
services:
  pipeline:   # Python + YOLOv12s + EasyOCR
  dashboard:  # React lite (visualización en tiempo real)
```

- **Consideración especial:** El acceso a la cámara (webcam USB / RTSP) requiere
  pasar el dispositivo al contenedor (`devices:` en docker-compose).
- **SQLite:** Persistido en un volumen local para sobrevivir reinicios del contenedor.
- **GPU (futuro Jetson):** Cuando se migre a NVIDIA Jetson, se usará la imagen base
  `nvcr.io/nvidia/l4t-pytorch` que incluye soporte CUDA nativo.

## Justificación

| Factor | Sin Docker | Con Docker |
|---|---|---|
| Setup para nuevo dev | Horas (instalar Python, deps, configurar) | Minutos (`docker-compose up`) |
| Consistencia con producción | ❌ Diferencias de OS/versiones | ✅ Mismo contenedor en dev y prod |
| Deploy en Coolify | Requiere configuración manual de servidor | ✅ Coolify soporta Docker Compose nativamente |
| Aislamiento de dependencias | ❌ Conflictos entre YOLO, EasyOCR y otros | ✅ Cada servicio en su propio contenedor |
| Portabilidad a Jetson (futuro) | Requiere reinstalar todo en ARM | ✅ Cambiar imagen base es suficiente |

## Consecuencias

- **Positivas:**
  - El compañero dev puede levantar el entorno completo con un solo comando.
  - El deploy a producción es trivial (Coolify ya sabe trabajar con `docker-compose.yml`).
  - La migración futura a NVIDIA Jetson Orin Nano solo requiere cambiar la imagen base.

- **Negativas / Consideraciones:**
  - El acceso a la GPU para inferencia local (en prototipo PC) requiere
    instalar [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)
    si la PC tiene GPU NVIDIA.
  - El acceso a la cámara USB dentro de Docker en Windows puede requerir
    configuración adicional (pasar dispositivo `/dev/video0` o equivalente WSL2).
  - Para desarrollo rápido del frontend, el equipo puede optar por correr
    `npm run dev` directamente sin Docker (Vite HMR es más rápido fuera de contenedor).

## Referencias

- [Coolify Documentation — Docker Compose](https://coolify.io/docs/resources/services/docker-compose)
- [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/)
- [Docker Desktop para Windows](https://docs.docker.com/desktop/windows/)
