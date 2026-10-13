# Capability: Incidencias

**Fecha de creación:** 2026-10-12
**Última actualización:** 2026-10-12

## Propósito

Especificación técnica de la incidencia de soporte de HelpDesk Lite: sus modelos, el contrato REST que comparten [helpdesk-backend](../../../helpdesk-backend/README.md) y [helpdesk-frontend](../../../helpdesk-frontend/README.md), sus flujos y su ciclo de estados. Cubre registro ([US-001](../../specs/changes/user-stories/US-001-registrar-incidencia/README.md), RF-01), listado y filtros ([US-002](../../specs/changes/user-stories/US-002-listar-filtrar-incidencias/README.md), RF-02), detalle ([US-003](../../specs/changes/user-stories/US-003-detalle-incidencia/README.md), RF-03), comentarios ([US-004](../../specs/changes/user-stories/US-004-comentar-incidencia/README.md), RF-04), cambio de estado ([US-005](../../specs/changes/user-stories/US-005-cambiar-estado-incidencia/README.md), RF-05) e indicadores del dashboard ([US-006](../../specs/changes/user-stories/US-006-dashboard-indicadores/README.md), RF-06). Fuera: autenticación y roles, edición o borrado de incidencias y comentarios.

## Modelos de datos

| Id | Modelo | Descripción |
| -- | ------ | ----------- |
| [MD-001](models/MD-001-incidencia.md) | Incidencia | Solicitud de soporte persistida: datos del solicitante más `id`, `status` y `createdAt` asignados por el sistema |
| [MD-002](models/MD-002-comentario.md) | Comentario | Nota de seguimiento persistida de una incidencia (N—1), con autor de demostración asignado por el sistema; inmutable |
| [MD-003](models/MD-003-indicadores-incidencias.md) | Indicadores de incidencias | Proyección no persistida: total y recuento por estado, calculados al consultar |

## APIs / Endpoints

| Id | Grupo | Operaciones | Descripción |
| -- | ----- | ----------- | ----------- |
| [API-001](apis/API-001-incidencias.md) | Incidencias | `POST /incidents`, `GET /incidents`, `GET /incidents/summary`, `GET /incidents/{id}`, `POST /incidents/{id}/comments`, `PATCH /incidents/{id}/status` | Registro, listado filtrado y paginado, indicadores, detalle con comentarios, alta de comentarios y cambio de estado |

## Flujos / Procesos

| Id | Flujo | Descripción |
| -- | ----- | ----------- |
| [FL-001](flows/FL-001-registro-incidencia.md) | Registro de una incidencia | Formulario → contrato OpenAPI → validación Zod → persistencia SQLite → confirmación o errores por campo |
| [FL-002](flows/FL-002-consulta-listado-incidencias.md) | Consulta del listado de incidencias | Filtros, orden y página → `GET /incidents` → página de 20 con totales, estado vacío o error |
| [FL-003](flows/FL-003-consulta-detalle-incidencia.md) | Consulta del detalle de una incidencia | Id → `GET /incidents/{id}` → datos y comentarios ascendentes, o «no encontrada» (404, también para id no UUID) |
| [FL-004](flows/FL-004-registro-comentario.md) | Registro de un comentario | Formulario en el detalle → `POST /incidents/{id}/comments` → comentario añadido sin recargar, o error con texto conservado |
| [FL-005](flows/FL-005-cambio-estado-incidencia.md) | Cambio de estado de una incidencia | Acción válida → `PATCH /incidents/{id}/status` → nuevo estado, o rechazo (409) sin modificar nada |

## Diagramas

| Id | Diagrama | Tipo | Alcance |
| -- | -------- | ---- | ------- |
| [DG-001](diagrams/DG-001-estados-incidencia.md) | Estados de la incidencia | Estados | Estado inicial `Open` y transiciones permitidas de `status` |

## Observaciones

- **MD-002 — valor de `author`:** el usuario de demostración es fijo y su valor literal es `Agente demo`, fijado en el wireframe aprobado del formulario de comentario de [US-004](../../specs/changes/user-stories/US-004-comentar-incidencia/README.md). Cerrado.
- **API-001 (`GET /incidents`) — un valor por filtro:** se especifica un único valor por parámetro (`status`, `priority`, `category`); la combinación es entre parámetros distintos (BR-01 de [US-002](../../specs/changes/user-stories/US-002-listar-filtrar-incidencias/README.md)). Seleccionar varios valores del mismo filtro no está contemplado; confirmar si el wireframe del listado lo requiere.
- **API-001 (`GET /incidents`) — parámetros de query no declarados:** se ignoran (comportamiento por defecto de Zod), a diferencia de los bodies, que son estrictos. Confirmar en la planificación técnica de US-002 si se prefiere rechazarlos con `400`.
- **API-001 (`GET /incidents/{id}`, `POST /incidents/{id}/comments`, `PATCH /incidents/{id}/status`) — cuerpo de `404` y `409`:** estructura genérica `{ statusCode, error, message }` con `error: "Not Found"` / `"Conflict"` y mensaje en español; el manejador común ([`src/plugins/error-handler.ts`](../../../helpdesk-backend/src/plugins/error-handler.ts)) toma hoy `error` del nombre de la excepción, por lo que la implementación debe ajustarlo ([US-003](../../specs/changes/user-stories/US-003-detalle-incidencia/README.md), [US-005](../../specs/changes/user-stories/US-005-cambiar-estado-incidencia/README.md)). Redacción exacta de los mensajes en la implementación.
- **API-001 (`POST /incidents/{id}/comments`, `PATCH /incidents/{id}/status`) — id no UUID v4:** se trata como `404`, igual que en el detalle (BR-01 de US-003), por coherencia del recurso; US-004 y US-005 solo exigen `404` para la incidencia inexistente.
- **FL-003 — dirección del detalle en el frontend:** la ruta exacta de Angular se fija en la planificación técnica o con el wireframe del detalle ([US-003](../../specs/changes/user-stories/US-003-detalle-incidencia/README.md)).
- **Persistencia (MD-001, MD-002):** nombres de tabla/columnas, representación de `createdAt` en SQLite e índices (filtros/orden del listado, `incidentId` de comentarios) se deciden en la planificación técnica junto con la librería de acceso a datos ([persistence.md](../../../helpdesk-backend/docs/standards/persistence.md)).
- **Diseño del PRD no cubierto aquí:** diagrama de componentes frontend/backend; se documentará como `DG-XXX` cuando una historia lo pida.

## Historial de cambios

| Fecha | Cambio | Origen |
| ----- | ------ | ------ |
| 2026-10-12 | API-001: añadida `GET /incidents`; añadido FL-002 | [US-002](../../specs/changes/user-stories/US-002-listar-filtrar-incidencias/README.md) |
| 2026-10-12 | API-001: `GET /incidents/{id}` ampliada con comentarios y `404` cerrado (incluido id no UUID v4); MD-001 actualizado; añadido FL-003. Cerrado el pendiente de `GET /incidents/{id}` | [US-003](../../specs/changes/user-stories/US-003-detalle-incidencia/README.md) |
| 2026-10-12 | Añadido MD-002; API-001: añadida `POST /incidents/{id}/comments`; añadido FL-004 | [US-004](../../specs/changes/user-stories/US-004-comentar-incidencia/README.md) |
| 2026-10-12 | Añadido DG-001; MD-001: `status` literal confirmado y transiciones; API-001: añadida `PATCH /incidents/{id}/status` (409); añadido FL-005. Cerrados los pendientes de estilo de `status` y del diagrama de estados | [US-005](../../specs/changes/user-stories/US-005-cambiar-estado-incidencia/README.md) |
| 2026-10-12 | Añadido MD-003; API-001: añadida `GET /incidents/summary` | [US-006](../../specs/changes/user-stories/US-006-dashboard-indicadores/README.md) |
| 2026-10-12 | FL-001: enlaces a los flujos nuevos y respuesta de detalle con `comments` | US-002..US-005 |
