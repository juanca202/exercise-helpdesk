# TK-002: Endpoint de creación de incidencias

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-001](./README.md)
**Repositorio:** helpdesk-backend

## Descripción

Exponer `POST /incidents`: validar el cuerpo con esquemas Zod estrictos (recorte de espacios, longitudes, enums, claves no declaradas), asignar `id` (UUID v4), `status = Open` y `createdAt`, persistir la incidencia y responder `201` con ella, o `400` con errores de validación en español sin persistir nada. La ruta queda publicada en el contrato OpenAPI.

## Dependencias

- `incidents.repository.ts` ([TK-001](./TK-001-persistencia-sqlite-incidencias.md)) — inserción de la incidencia.
- `@fastify/type-provider-zod` / `zod` — esquemas del contrato y validación (`z.strictObject`).
- `node:crypto` (`randomUUID`) — generación del identificador.
- `src/plugins/error-handler.ts` — formato de errores `{statusCode, error, message, issues[]}`.

## Referencias

- **Arquitectura:** [ADR-001: API First con Fastify, Zod y OpenAPI](../../../../../helpdesk-backend/docs/adr/ADR-001-api-first-fastify-zod-openapi.md) · [ADR-008: Contrato OpenAPI compartido](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md) · [ADR-006: Plugins encapsulados por feature](../../../../../helpdesk-backend/docs/adr/ADR-006-plugins-encapsulados-por-feature.md)
- **Estándares:** [API Standards](../../../../../helpdesk-backend/docs/standards/api.md) · [Architecture Standards](../../../../../helpdesk-backend/docs/standards/architecture.md)
- **Documentación técnica:** [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md) · [API-001: POST /incidents](../../../../architecture/incidencias/apis/API-001-incidencias.md#post-incidents) · [FL-001: Registro de una incidencia](../../../../architecture/incidencias/flows/FL-001-registro-incidencia.md)

## Archivos afectados

```text
helpdesk-backend/
├── src/
│   ├── ~ app.ts                                     # registra el módulo incidents con prefix /incidents
│   ├── ~ plugins/error-handler.ts                   # asegura que issues exponga los mensajes propios en español
│   └── modules/incidents/
│       ├── + incidents.schemas.ts                   # esquemas Zod: creación (estricto), incidencia, enums
│       ├── + incidents.service.ts                   # crea la incidencia: id, status Open, createdAt
│       └── + incidents.routes.ts                    # POST / con respuestas 201 y 400
└── test/
    ├── + incidents.create.test.ts                   # inject: casos válidos e inválidos
    └── ~ openapi.test.ts                            # la ruta aparece en el contrato
```

## Plan de implementación

- [ ] **IT-01** — Definir en `incidents.schemas.ts` los enums de categoría (`HARDWARE`, `SOFTWARE`, `ACCESS_PERMISSIONS`, `NETWORK_CONNECTIVITY`, `EMAIL_COMMUNICATION`, `OTHER`) y prioridad (`LOW`, `MEDIUM`, `HIGH`)
- [ ] **IT-02** — Definir el esquema de creación con `z.strictObject`: `title` (recorte y 5–100), `description` (recorte y 10–2000), `category`, `priority`; con mensajes de validación propios en español
  Un valor solo con espacios cuenta como vacío; `id`, `status` y `createdAt` enviados por el cliente (o cualquier clave no declarada) se rechazan.
- [ ] **IT-03** — Definir el esquema de respuesta de la incidencia (`id` UUID, campos, `status`, `createdAt` ISO 8601)
- [ ] **IT-04** — Implementar `incidents.service.ts`: asignar `id` con `randomUUID()`, `status = 'Open'` y `createdAt`, y persistir vía el repositorio; sin importar tipos de Fastify
- [ ] **IT-05** — Implementar `POST /` en `incidents.routes.ts` con respuesta `201` y `400`, y registrar el módulo en `app.ts` con `prefix: '/incidents'`
- [ ] **IT-06** — Verificar que el manejador de errores común devuelve en `issues` el campo (`instancePath`) y el mensaje propio en español; ajustarlo solo si hace falta
- [ ] **IT-07** — Pruebas con `inject`: `201` con id, `Open` y `createdAt`; `400` por título/descripción/categoría/prioridad inválidos, por espacios y por claves `id`/`status`/`createdAt`; nada persistido en los `400`; la ruta figura en el OpenAPI
