# ADR-001 — Stack Tecnológico del Nodo Central

**Fecha:** 17 Agosto 2026
**Estado:** Aceptado

---

## Contexto

El sistema ALPR requiere un **Nodo Central** (Nube/Servidor Homelab) que actuará como la fuente maestra de verdad y panel administrativo. Las necesidades específicas de este nodo son:
1. Ofrecer una interfaz gráfica moderna, rápida y amigable para el registro de funcionarios y vehículos por parte del equipo administrativo de la UTEM.
2. Proveer una API REST para recibir de forma asíncrona la sincronización de registros de acceso desde múltiples Nodos de Borde (Edge Nodes).
3. Asegurar la persistencia confiable e integridad referencial de los datos maestros.
4. Facilitar su despliegue y mantenimiento en la infraestructura actual (Servidor Ubuntu con Coolify).

## Decisión

El stack tecnológico seleccionado para el Nodo Central es: **React + TypeScript (Frontend) + FastAPI (Backend) + PostgreSQL (Base de datos) + Docker (Infraestructura)**.

## Justificación

1. **FastAPI (Python):** 
   - Soporte nativo para asincronía (`async/await`), crucial para manejar múltiples Nodos de Borde que sincronizan datos simultáneamente sin bloquear el hilo principal.
   - Generación automática de documentación (OpenAPI/Swagger), facilitando la integración con el frontend.
   - Estandariza el lenguaje del equipo, ya que el Nodo de Borde también utilizará Python (para ML e inferencia), reduciendo el cambio de contexto entre desarrolladores.
2. **React + TypeScript:**
   - Permite construir una Single Page Application (SPA) con alta reactividad, proporcionando una experiencia de usuario moderna ("Rich Aesthetics" y "Dynamic Design").
   - TypeScript previene errores en tiempo de ejecución al tipar estrictamente los contratos de datos (interfaces) provenientes de la API REST.
3. **PostgreSQL:**
   - Motor relacional robusto, de código abierto, con estricto cumplimiento ACID, esencial para mantener la integridad del modelo relacional (`Funcionario -> Vehiculo -> RegistroAcceso`).
4. **Docker:**
   - Garantiza consistencia entre el entorno de desarrollo (Windows) y el entorno de producción (Servidor Ubuntu autoalojado).
   - Se integra nativamente con Coolify, permitiendo despliegues automáticos limpios y aislados.

## Consecuencias

### Positivas
- Alto rendimiento en la gestión de I/O de red gracias a FastAPI.
- Código fuertemente tipado en el frontend, reduciendo bugs.
- Independencia total (desacoplamiento) entre Frontend y Backend, lo que facilita que trabajen diferentes desarrolladores en paralelo.

### Negativas / Riesgos
- La adopción de una arquitectura de SPA + API requiere gestionar el estado y la autenticación mediante JWT (añadiendo complejidad en comparación con aplicaciones monolíticas tipo SSR/Django/Laravel).
- Se requieren dos procesos corriendo separados en desarrollo y contenedores distintos en producción.
