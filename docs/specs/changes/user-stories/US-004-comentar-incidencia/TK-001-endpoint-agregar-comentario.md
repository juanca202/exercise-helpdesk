# TK-001: Endpoint para agregar comentarios a una incidencia

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-004](./README.md)
**Repositorio:** helpdesk-backend

## Descripción

Exponer `POST /incidents/{id}/comments`: validar el cuerpo con un esquema Zod estricto que solo admite `content` (recorte de espacios, 1–500 caracteres, un valor solo con espacios cuenta como vacío), asignar en el sistema `id` (UUID v4), `incidentId`, `author` (valor literal «Agente demo») y `createdAt`, persistir el comentario y responder `201` con él. Responder `400` con errores de validación en español sin persistir nada, incluidas las peticiones que envían `id`, `incidentId`, `author` o `createdAt`; y `404` sin persistir cuando la incidencia no existe o su id no es un UUID v4, comprobado solo cuando el cuerpo es válido. La operación queda publicada en el contrato OpenAPI y el contrato no expone ninguna operación de edición ni de eliminación de comentarios.

## Dependencias

- `comments.repository.ts` y tipo `Comment` ([TK-001 de US-003](../US-003-detalle-incidencia/TK-001-persistencia-comentarios.md)) — inserción del comentario en la tabla `comments`.
- Obtención de incidencia por id con `404` común ([TK-002 de US-003](../US-003-detalle-incidencia/TK-002-endpoint-detalle-incidencia.md)) — comprobación de existencia (id inexistente o no UUID v4) y cuerpo `{statusCode, error: "Not Found", message}`.
- Esquemas y servicio del módulo `incidents` ([TK-002 de US-001](../US-001-registrar-incidencia/TK-002-endpoint-crear-incidencia.md)) — `incidents.schemas.ts`, `incidents.service.ts` e `incidents.routes.ts`, registrados con `prefix: '/incidents'`.
- `@fastify/type-provider-zod` / `zod` — esquemas del contrato y validación (`z.strictObject`).
- `node:crypto` (`randomUUID`) — generación del identificador del comentario.
- `src/plugins/error-handler.ts` — formato de errores `{statusCode, error, message, issues[]}` para el `400`.

## Referencias

- **Arquitectura:** [ADR-001: API First con Fastify, Zod y OpenAPI](../../../../../helpdesk-backend/docs/adr/ADR-001-api-first-fastify-zod-openapi.md) · [ADR-008: Contrato OpenAPI compartido](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md) · [ADR-006: Plugins encapsulados por feature](../../../../../helpdesk-backend/docs/adr/ADR-006-plugins-encapsulados-por-feature.md) · [ADR-004: Vitest y Fastify inject](../../../../../helpdesk-backend/docs/adr/ADR-004-vitest-fastify-inject.md) · [ADR-007: Persistencia local con SQLite](../../../../../helpdesk-backend/docs/adr/ADR-007-sqlite-persistencia-local.md)
- **Estándares:** [API Standards](../../../../../helpdesk-backend/docs/standards/api.md) · [Architecture Standards](../../../../../helpdesk-backend/docs/standards/architecture.md) · [Testing Standards](../../../../../helpdesk-backend/docs/standards/testing.md) · [Persistence Standards](../../../../../helpdesk-backend/docs/standards/persistence.md)
- **Documentación técnica:** [MD-002: Comentario](../../../../architecture/incidencias/models/MD-002-comentario.md) · [API-001: POST /incidents/{id}/comments](../../../../architecture/incidencias/apis/API-001-incidencias.md#post-incidents-id-comments) · [FL-004: Registro de un comentario](../../../../architecture/incidencias/flows/FL-004-registro-comentario.md)

## Archivos afectados

```text
helpdesk-backend/
├── src/modules/incidents/
│   ├── ~ incidents.schemas.ts                       # esquemas Zod: alta de comentario (estricto) y comentario
│   ├── ~ incidents.service.ts                       # agregar comentario: existencia, id, author, createdAt
│   └── ~ incidents.routes.ts                        # POST /:id/comments con respuestas 201, 400, 404 y 500
└── test/
    ├── + incidents.comments.test.ts                 # inject: alta válida, 400, 404 y orden de validación
    └── ~ openapi.test.ts                            # ruta publicada; sin PUT/PATCH/DELETE de comentarios
```

## Plan de implementación

- [ ] **IT-01** — Definir en `incidents.schemas.ts` el esquema de alta de comentario con `z.strictObject`: solo `content` (recorte y 1–500), con mensajes propios en español
  Mensajes de `content`: «El comentario no puede estar vacío» (ausente, vacío o solo espacios) y «El comentario no puede superar los 500 caracteres». Cualquier clave no declarada (`id`, `incidentId`, `author`, `createdAt` u otra) se rechaza con un mensaje propio en español que indique que el campo no está permitido.
- [ ] **IT-02** — Definir el esquema de respuesta del comentario (`id` UUID, `incidentId` UUID, `content`, `author`, `createdAt` ISO 8601) reutilizando el de [TK-001 de US-003](../US-003-detalle-incidencia/TK-001-persistencia-comentarios.md) si ya existe
- [ ] **IT-03** — Implementar en `incidents.service.ts` la operación de agregar comentario: comprobar la existencia de la incidencia con la obtención por id de US-003 (`404` si no existe o el id no es UUID v4), asignar `id` con `randomUUID()`, `incidentId` del parámetro de ruta, `author = 'Agente demo'` y `createdAt`, e insertar vía `comments.repository.ts`; sin importar tipos de Fastify
  El valor «Agente demo» queda como constante única del módulo; se persiste el `content` ya recortado.
- [ ] **IT-04** — Registrar `POST /:id/comments` en `incidents.routes.ts` con el parámetro `id` sin formato UUID en el esquema y las respuestas `201`, `400`, `404` y `500` declaradas en el contrato
  El body se valida antes de ejecutar el handler: un body inválido sobre una incidencia inexistente responde `400`; el `404` solo se evalúa con body válido.
- [ ] **IT-05** — Verificar que no se registra ninguna ruta de edición ni de eliminación de comentarios (`PUT`, `PATCH` o `DELETE` sobre `/:id/comments` o `/:id/comments/:commentId`)
- [ ] **IT-06** — Pruebas con `inject` del alta válida: `201` con `id` UUID, `incidentId` de la ruta, `author` «Agente demo», `createdAt` y `content` recortado; el comentario aparece después en `GET /incidents/{id}`
- [ ] **IT-07** — Pruebas con `inject` de los `400`: `content` ausente, vacío, solo espacios y de 501 caracteres; body ausente o no JSON; claves `id`, `incidentId`, `author` y `createdAt`; cada respuesta con `issues` (`instancePath` y mensaje en español) y ningún comentario persistido
  Incluye el caso límite de 500 caracteres aceptado y el de 1 carácter tras recortar.
- [ ] **IT-08** — Pruebas con `inject` de los `404`: UUID v4 inexistente e id no UUID con body válido, cuerpo `{statusCode: 404, error: "Not Found", message}` y nada persistido; body inválido sobre incidencia inexistente responde `400`
- [ ] **IT-09** — Pruebas del contrato OpenAPI: `POST /incidents/{id}/comments` figura con sus respuestas `201`, `400`, `404` y `500`, y no existen operaciones `put`, `patch` ni `delete` sobre rutas de comentarios
