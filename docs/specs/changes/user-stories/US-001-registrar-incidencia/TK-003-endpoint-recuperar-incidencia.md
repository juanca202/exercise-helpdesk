# TK-003: Endpoint de recuperación de incidencia por id

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-001](./README.md)
**Repositorio:** helpdesk-backend

## Descripción

Exponer `GET /incidents/{id}` con lo mínimo que exige la recuperación posterior de una incidencia registrada (AC-003): devolver `200` con los datos con que se registró, o `404` si no existe. El detalle completo (comentarios y cuerpo exacto del `404`) lo cierra la historia de detalle (RF-03).

## Dependencias

- `incidents.repository.ts` ([TK-001](./TK-001-persistencia-sqlite-incidencias.md)) — `findById`.
- Esquemas de incidencia de [TK-002](./TK-002-endpoint-crear-incidencia.md) — respuesta `200`.
- `src/plugins/error-handler.ts` — respuesta `404` con `{statusCode, error, message}`.

## Referencias

- **Arquitectura:** [ADR-001: API First con Fastify, Zod y OpenAPI](../../../../../helpdesk-backend/docs/adr/ADR-001-api-first-fastify-zod-openapi.md) · [ADR-008: Contrato OpenAPI compartido](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md)
- **Estándares:** [API Standards](../../../../../helpdesk-backend/docs/standards/api.md)
- **Documentación técnica:** [API-001: GET /incidents/{id}](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents-id) · [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md)

## Archivos afectados

```text
helpdesk-backend/
├── src/modules/incidents/
│   ├── ~ incidents.schemas.ts                       # esquema de params (id UUID)
│   ├── ~ incidents.service.ts                       # obtener por id; error 404 si no existe
│   └── ~ incidents.routes.ts                        # GET /:id con respuestas 200 y 404
└── test/
    └── + incidents.get.test.ts                      # inject: crear y recuperar; id inexistente
```

## Plan de implementación

- [x] **IT-01** — Añadir el esquema de parámetros (`id` como UUID) y las respuestas `200` y `404` de `GET /:id`
- [x] **IT-02** — Implementar la obtención por id en el servicio, lanzando un error con `statusCode` 404 cuando no existe
- [x] **IT-03** — Registrar `GET /:id` en `incidents.routes.ts` y verificar que figura en el contrato OpenAPI
- [x] **IT-04** — Pruebas con `inject`: registrar y recuperar devuelve los mismos datos (AC-003); un UUID inexistente devuelve `404`
