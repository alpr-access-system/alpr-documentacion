# Especificación — Panel Administrativo (Nodo Central)

**Estado:** Aprobado
**Stack:** React + TypeScript (Frontend) / FastAPI + PostgreSQL (Backend)

---

## Descripción

El Panel Administrativo es una aplicación web desplegada en la nube que actúa como la consola central del sistema ALPR. 

*(Nota técnica sobre SPA: Es una Single Page Application. Esto no significa que tenga "una sola pestaña", sino que tiene múltiples vistas y pestañas —Dashboard, Funcionarios, Historial— pero navega entre ellas instantáneamente de forma fluida sin tener que recargar la página web por completo).*

## Módulos Funcionales

1. **Autenticación y Sesión:**
   - Login seguro con JWT para el personal administrativo y conserjes.
   - Protección de rutas en el frontend.
   - Obtención del perfil activo (`/me`).

2. **Gestión de Usuarios del Sistema (RBAC):**
   - Control de Acceso Basado en Roles (Role-Based Access Control).
   - **Rol Admin:** Permiso total. Puede crear/eliminar cuentas de otros administradores o conserjes.
   - **Rol Conserje:** Permiso operativo. Puede registrar y modificar profesores (funcionarios) y sus vehículos, además de ver el historial. No puede gestionar usuarios del sistema.

3. **Gestión de Funcionarios y Vehículos (Módulo Unificado):**
   - CRUD completo de funcionarios de la UTEM.
   - La gestión de vehículos se hace **directamente dentro de la edición del funcionario**. Todo vehículo pertenece a un solo funcionario (relación Padre-Hijo). Si dos profesores comparten auto, basta con asociarlo a uno solo para que el portón abra.
   - **Atributos Funcionario:** ID, Nombre Completo, RUT, Correo, Estado (Activo/Inactivo).
   - **Atributos Vehículo:** Patente (única), Tipo (Auto, Moto, etc.).

4. **Historial de Accesos (Logs):**
   - Visualización de los registros consolidados provenientes de los Nodos de Borde.
   - **Importante:** El sistema ignorará activamente las patentes no reconocidas o que pasen por la calle. No se gastarán recursos de base de datos en autos ajenos al sistema.
   - Filtros: Rango de fechas, búsqueda por patente, estado (Acceso Permitido, Acceso Denegado).
   - Datos visualizados: Foto (si aplica), patente, fecha/hora, nodo de origen.

5. **Dashboard y Estadísticas:**
   - Visualización de métricas de resumen (ej. cantidad estimada de autos actualmente dentro del estacionamiento).
   - Gráficos estadísticos (ej. demanda de estacionamiento por franjas horarias).

## Especificación de API REST (FastAPI)

Los siguientes endpoints serán implementados en el backend:

### Autenticación
- `POST /api/auth/login`: Genera token JWT.
- `GET /api/auth/me`: Retorna los datos y el rol del usuario actualmente autenticado (Enterprise best practice).

### Gestión de Usuarios (Requiere Rol: Admin)
- `GET, POST /api/usuarios/`: Listar y crear cuentas de sistema (conserjes/admins).
- `GET, PUT, DELETE /api/usuarios/{id}`: Detalle, edición y borrado de cuentas.

### CRUD Funcionarios y Vehículos (Anidado)
- `GET, POST /api/funcionarios/`: Listar y crear funcionarios.
- `GET, PUT, DELETE /api/funcionarios/{id}`: Detalle, edición y borrado (lógico) del funcionario.
- `POST /api/funcionarios/{id}/vehiculos`: Agregar un nuevo vehículo (patente) a un funcionario específico.
- `DELETE /api/funcionarios/{id}/vehiculos/{patente}`: Eliminar el permiso de un vehículo específico.

### Sincronización (Comunicación Nube-Borde)
- `GET /api/sync/whitelist`: Devuelve la lista maestra optimizada de patentes activas para que el Nodo de Borde la guarde en su SQLite local.
- `POST /api/sync/accesos`: Recibe los registros asíncronos creados en el Nodo de Borde para guardarlos en PostgreSQL.

### Reportes
- `GET /api/accesos/`: Obtiene el historial de accesos con paginación.
- `GET /api/metrics/dashboard`: Retorna métricas agregadas para el dashboard.

## Pantallas (Frontend React)

El diseño de la UI seguirá la estética moderna ("Rich Aesthetics" y "Glassmorphism") para un aspecto premium:

1. **Pantalla de Login:** Minimalista, con validación de credenciales.
2. **Layout Principal:** Menú lateral (Sidebar) de navegación y barra superior (Header) con perfil del usuario logueado.
3. **Pantalla Dashboard:** Tarjetas (Cards) con métricas en tiempo real y gráficos simples de accesos diarios.
4. **Gestión de Accesos (Solo Admin):** Tabla exclusiva para que el administrador gestione a los conserjes.
5. **Vista de Funcionarios:** Tabla de datos con buscador. Botón flotante para "Añadir Funcionario" (abre modal).
6. **Vista de Vehículos:** Modal dentro de Funcionarios para gestionar la lista de patentes.
7. **Vista de Historial:** Tabla detallada. Incluye un indicador visual (badge) de si el acceso fue otorgado o rechazado.
