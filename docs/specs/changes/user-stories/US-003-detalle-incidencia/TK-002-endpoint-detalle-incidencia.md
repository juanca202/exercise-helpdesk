# TK-002: Endpoint de detalle de incidencia con comentarios

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-003](./README.md)
**Repositorio:** helpdesk-backend

## Descripción

Ampliar `GET /incidents/{id}` para que devuelva el detalle completo de la incidencia: todos sus datos más `comments`, ordenados por `createdAt` ascendente y `[]` cuando no tiene comentarios. Un identificador inexistente o que no sea un UUID v4 válido responde `404` con el mismo cuerpo `{ statusCode: 404, error: "Not Found", message }` y un mensaje en español, sin distinguir entre ambos casos. La respuesta de detalle queda publicada en el contrato OpenAPI.

## Dependencias

- `GET /:id` de `incidents.routes.ts` y obtención por id de `incidents.service.ts` ([TK-003 de US-001](../US-001-registrar-incidencia/TK-003-endpoint-recuperar-incidencia.md)) — operación que se amplía.
- `incidents.schemas.ts` ([TK-002 de US-001](../US-001-registrar-incidencia/TK-002-endpoint-crear-incidencia.md)) — esquema de la incidencia que se extiende con `comments`.
- `incidents.repository.ts` ([TK-001 de US-001](../US-001-registrar-incidencia/TK-001-persistencia-sqlite-incidencias.md)) — `findById`.
- `comments.repository.ts` ([TK-001](./TK-001-persistencia-comentarios.md)) — `findByIncidentId` ordenado e `insert` para preparar los datos de las pruebas.
- `src/plugins/error-handler.ts` — formato común de errores `{ statusCode, error, message }`.
- `@fastify/type-provider-zod` / `zod` — esquemas de parámetros y de respuesta.

## Referencias

- **Arquitectura:** [ADR-001: API First con Fastify, Zod y OpenAPI](../../../../../helpdesk-backend/docs/adr/ADR-001-api-first-fastify-zod-openapi.md) · [ADR-008: Contrato OpenAPI compartido](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md) · [ADR-005: Monolito modular](../../../../../helpdesk-backend/docs/adr/ADR-005-monolito-modular-adaptador-http.md) · [ADR-004: Vitest e inject de Fastify](../../../../../helpdesk-backend/docs/adr/ADR-004-vitest-fastify-inject.md)
- **Estándares:** [API Standards](../../../../../helpdesk-backend/docs/standards/api.md) · [Architecture Standards](../../../../../helpdesk-backend/docs/standards/architecture.md) · [Testing Standards](../../../../../helpdesk-backend/docs/standards/testing.md)
- **Documentación técnica:** [API-001: GET /incidents/{id}](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents-id) · [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md) · [MD-002: Comentario](../../../../architecture/incidencias/models/MD-002-comentario.md) · [FL-003: Consulta del detalle de una incidencia](../../../../architecture/incidencias/flows/FL-003-consulta-detalle-incidencia.md)

## Archivos afectados

```text
helpdesk-backend/
├── src/
│   ├── ~ plugins/error-handler.ts                   # error "Not Found" en los 404 (frase HTTP del código, no el nombre de la excepción)
│   └── modules/incidents/
│       ├── ~ incidents.schemas.ts                   # params id sin formato UUID; esquema de comentario y de detalle
│       ├── ~ incidents.service.ts                   # detalle: id UUID v4, incidencia y comentarios; 404 común
│       └── ~ incidents.routes.ts                    # GET /:id responde el detalle (200, 404)
└── test/
    ├── ~ incidents.get.test.ts                      # inject: detalle con y sin comentarios, orden y 404
    ├── ~ error-handler.test.ts                      # error "Not Found" en un 404
    └── ~ openapi.test.ts                            # respuesta de detalle con comments en el contrato
```

## Plan de implementación

- [x] **IT-01** — Cambiar el esquema de parámetros de `GET /:id` para que `id` sea un texto sin formato UUID, de modo que un id inválido no produzca `400`
- [x] **IT-02** — Definir en `incidents.schemas.ts` el esquema del comentario (MD-002) y el esquema de detalle: campos de la incidencia más `comments` como lista de comentarios
- [x] **IT-03** — Declarar en `GET /:id` las respuestas `200` (detalle) y `404` (`statusCode`, `error`, `message`)
- [x] **IT-04** — Implementar la obtención del detalle en el servicio: si el `id` no es un UUID v4 válido, lanzar el error `404` sin consultar la base de datos; si la incidencia no existe, el mismo error; si existe, devolverla con sus comentarios de `findByIncidentId`
  Un único error de dominio con `statusCode` 404 y mensaje en español que comunique que la incidencia no fue encontrada (p. ej. «La incidencia no fue encontrada»), compartido por ambos casos; sin importar tipos de Fastify en el servicio.
- [x] **IT-05** — Ajustar el manejador de errores común para que una respuesta `404` lleve `error: "Not Found"` en lugar del nombre de la excepción, conservando el `message` propio
  Sin alterar el comportamiento de los `400` de validación ni de los `500`.
- [x] **IT-06** — Pruebas con `inject`: detalle de una incidencia con varios comentarios devuelve todos los campos y los comentarios en orden ascendente; dos comentarios con el mismo `createdAt` salen en orden de inserción; una incidencia sin comentarios devuelve `comments: []`
- [x] **IT-07** — Pruebas con `inject` del `404`: un UUID v4 inexistente y un id que no es UUID v4 (p. ej. `abc` o un UUID de otra versión) responden `404` con cuerpo idéntico `{ statusCode: 404, error: "Not Found", message }`; prueba del manejador común para el `404`; la respuesta de detalle con `comments` figura en el contrato OpenAPI
