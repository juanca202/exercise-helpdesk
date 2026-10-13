# TK-001: Endpoint de cambio de estado de una incidencia

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-005](./README.md)
**Repositorio:** helpdesk-backend

## Descripción

Exponer `PATCH /incidents/{id}/status`, que aplica sobre una incidencia existente únicamente las transiciones `Open → In Progress`, `In Progress → Resolved` y `Resolved → In Progress`, y responde `200` con la incidencia actualizada (sin comentarios). Cualquier otra transición, incluida la que mantiene el mismo estado, responde `409` con `{ statusCode: 409, error: "Conflict", message }` y un mensaje en español que nombra el estado actual y el destino. Un `status` ausente o no permitido, o claves no declaradas en el cuerpo, responden `400` con `issues` en español; una incidencia inexistente o un id que no es UUID v4 responde `404`. Ningún rechazo modifica datos: la transición se aplica con una única actualización condicionada al estado vigente, y si no afecta filas responde `409`. El nuevo estado queda persistido y visible en el detalle y en el listado, y la ruta queda publicada en el contrato OpenAPI.

## Dependencias

- `incidents.repository.ts` ([TK-001 de US-001](../US-001-registrar-incidencia/TK-001-persistencia-sqlite-incidencias.md)) — se amplía con la actualización condicionada del estado.
- `incidents.schemas.ts` ([TK-002 de US-001](../US-001-registrar-incidencia/TK-002-endpoint-crear-incidencia.md)) — enum de estado y esquema de respuesta de la incidencia, reutilizados en el cuerpo y en la respuesta `200`.
- `incidents.service.ts` e `incidents.routes.ts` ([TK-002](../US-001-registrar-incidencia/TK-002-endpoint-crear-incidencia.md) y [TK-003 de US-001](../US-001-registrar-incidencia/TK-003-endpoint-recuperar-incidencia.md)) — módulo `incidents` que recibe la nueva operación.
- Error de dominio `404` común y tratamiento del id no UUID v4 de [TK-002 de US-003](../US-003-detalle-incidencia/TK-002-endpoint-detalle-incidencia.md) — reutilizados para la incidencia inexistente.
- `GET /incidents` de [TK-001 de US-002](../US-002-listar-filtrar-incidencias/TK-001-endpoint-listar-incidencias.md) y `GET /incidents/{id}` de [TK-002 de US-003](../US-003-detalle-incidencia/TK-002-endpoint-detalle-incidencia.md) — usados en las pruebas para comprobar que el nuevo estado se refleja.
- `src/plugins/error-handler.ts` — formato común de errores; se asegura `error: "Conflict"` en los `409`.
- `@fastify/type-provider-zod` / `zod` — esquemas estrictos del cuerpo y declaración de respuestas.

## Referencias

- **Arquitectura:** [ADR-001: API First con Fastify, Zod y OpenAPI](../../../../../helpdesk-backend/docs/adr/ADR-001-api-first-fastify-zod-openapi.md) · [ADR-008: Contrato OpenAPI compartido](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md) · [ADR-007: Persistencia local con SQLite](../../../../../helpdesk-backend/docs/adr/ADR-007-sqlite-persistencia-local.md) · [ADR-005: Monolito modular](../../../../../helpdesk-backend/docs/adr/ADR-005-monolito-modular-adaptador-http.md) · [ADR-004: Vitest e inject de Fastify](../../../../../helpdesk-backend/docs/adr/ADR-004-vitest-fastify-inject.md)
- **Estándares:** [Persistence Standards](../../../../../helpdesk-backend/docs/standards/persistence.md) (`consistencia-transaccional`) · [API Standards](../../../../../helpdesk-backend/docs/standards/api.md) · [Architecture Standards](../../../../../helpdesk-backend/docs/standards/architecture.md) · [Testing Standards](../../../../../helpdesk-backend/docs/standards/testing.md)
- **Documentación técnica:** [API-001: PATCH /incidents/{id}/status](../../../../architecture/incidencias/apis/API-001-incidencias.md#patch-incidents-id-status) · [DG-001: Estados de la incidencia](../../../../architecture/incidencias/diagrams/DG-001-estados-incidencia.md) · [FL-005: Cambio de estado de una incidencia](../../../../architecture/incidencias/flows/FL-005-cambio-estado-incidencia.md) · [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md)

## Archivos afectados

```text
helpdesk-backend/
├── src/
│   ├── ~ plugins/error-handler.ts                   # error "Conflict" en los 409 (frase HTTP del código)
│   └── modules/incidents/
│       ├── ~ incidents.schemas.ts                   # cuerpo estricto { status }; respuestas 200, 400, 404 y 409
│       ├── ~ incidents.repository.ts                # actualización de status condicionada al estado vigente
│       ├── ~ incidents.service.ts                   # tabla de transiciones, 404, 409 y aplicación del cambio
│       └── ~ incidents.routes.ts                    # PATCH /:id/status
└── test/
    ├── + incidents.status.test.ts                   # inject: transiciones válidas, 409, 400, 404 y reflejo en detalle/listado
    ├── ~ incidents.repository.test.ts               # actualización condicionada: 1 fila y 0 filas
    ├── ~ error-handler.test.ts                      # error "Conflict" en un 409
    └── ~ openapi.test.ts                            # la ruta y sus respuestas figuran en el contrato
```

## Plan de implementación

- [ ] **IT-01** — Definir en `incidents.schemas.ts` el cuerpo de cambio de estado con `z.strictObject` y una sola clave `status` (enum `Open` | `In Progress` | `Resolved`), con mensajes propios en español
  Un cuerpo ausente o no JSON, un `status` ausente o fuera del enum y cualquier clave no declarada producen `400` con `issues` (`instancePath` `/status` para el campo).
- [ ] **IT-02** — Declarar en `PATCH /:id/status` los parámetros (mismo `id` sin formato UUID que el detalle) y las respuestas `200` (incidencia sin comentarios), `400`, `404` y `409` (`statusCode`, `error`, `message`)
- [ ] **IT-03** — Añadir al repositorio la actualización de `status` condicionada al estado vigente en una sola sentencia (`id` y estado origen en la condición), devolviendo el número de filas afectadas; sin importar tipos de Fastify
- [ ] **IT-04** — Definir en el servicio la tabla de transiciones permitidas de DG-001: `Open → In Progress`, `In Progress → Resolved`, `Resolved → In Progress`
- [ ] **IT-05** — Implementar el cambio de estado en el servicio: id no UUID v4 o incidencia inexistente → error `404` común; transición fuera de la tabla (incluido el mismo estado) → error `409`; transición permitida → actualización condicionada y `409` si afecta 0 filas; con 1 fila, devolver la incidencia actualizada
  El error `409` es un error de dominio con `statusCode` 409 y mensaje en español que nombra el estado actual y el destino (p. ej. «No se puede cambiar el estado de Open a Resolved»); ningún camino de rechazo escribe en la base de datos.
- [ ] **IT-06** — Registrar `PATCH /:id/status` en `incidents.routes.ts` y verificar que figura en el contrato OpenAPI con sus respuestas
- [ ] **IT-07** — Asegurar en el manejador de errores común que una respuesta `409` lleve `error: "Conflict"` y conserve el `message` propio, con el mismo criterio aplicado al `404`
  Sin alterar los `400` de validación ni los `500`.
- [ ] **IT-08** — Pruebas del repositorio: la actualización condicionada afecta 1 fila cuando el estado vigente coincide y 0 filas, sin cambios, cuando no coincide
- [ ] **IT-09** — Pruebas con `inject` de las transiciones válidas: `Open → In Progress`, `In Progress → Resolved` y `Resolved → In Progress` responden `200` con la incidencia actualizada sin `comments`, y el nuevo estado se lee después en `GET /incidents/{id}` y en `GET /incidents`
- [ ] **IT-10** — Pruebas con `inject` de los rechazos: `Open → Resolved`, `Resolved → Open` y el mismo estado responden `409` con `error: "Conflict"` y mensaje en español con estado actual y destino; `status` ausente, inválido o con claves extra responde `400` con `issues` en español; UUID inexistente e id no UUID v4 responden `404`; en todos los casos el estado persistido no cambia
- [ ] **IT-11** — Prueba de la actualización concurrente: si el estado vigente cambia entre la lectura y la escritura, la operación responde `409` y no modifica la incidencia
