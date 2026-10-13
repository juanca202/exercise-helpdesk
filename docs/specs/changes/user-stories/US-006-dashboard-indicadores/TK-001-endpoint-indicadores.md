# TK-001: Endpoint de indicadores de incidencias

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-006](./README.md)
**Repositorio:** helpdesk-backend

## Descripción

Exponer `GET /incidents/summary`: sin parámetros ni body, responder `200` con `{ total, open, inProgress, resolved }` calculado en cada petición mediante un recuento agrupado por `status` sobre las incidencias persistidas, con `total` igual a la suma de los tres recuentos y los cuatro valores en `0` cuando no hay incidencias; ante un fallo inesperado, responder `500` con el error genérico del proyecto. La ruta queda publicada en el contrato OpenAPI.

## Dependencias

- `incidents.repository.ts` y plugin `database` ([TK-001 de US-001](../US-001-registrar-incidencia/TK-001-persistencia-sqlite-incidencias.md)) — acceso a la tabla `incidents`; aquí se añade el recuento por estado.
- Módulo `incidents` (`incidents.schemas.ts`, `incidents.service.ts`, `incidents.routes.ts`) registrado con `prefix: '/incidents'` ([TK-002 de US-001](../US-001-registrar-incidencia/TK-002-endpoint-crear-incidencia.md)) — aloja el esquema, el servicio y la ruta nuevos.
- Ruta `GET /:id` ([TK-003 de US-001](../US-001-registrar-incidencia/TK-003-endpoint-recuperar-incidencia.md)) — ruta paramétrica junto a la que se declara la estática `/summary`.
- `@fastify/type-provider-zod` / `zod` — esquema de respuesta del contrato.
- `src/plugins/error-handler.ts` — respuesta `500` con `{statusCode, error, message}`.

## Referencias

- **Arquitectura:** [ADR-001: API First con Fastify, Zod y OpenAPI](../../../../../helpdesk-backend/docs/adr/ADR-001-api-first-fastify-zod-openapi.md) · [ADR-008: Contrato OpenAPI compartido](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md) · [ADR-006: Plugins encapsulados por feature](../../../../../helpdesk-backend/docs/adr/ADR-006-plugins-encapsulados-por-feature.md) · [ADR-007: Persistencia local con SQLite](../../../../../helpdesk-backend/docs/adr/ADR-007-sqlite-persistencia-local.md) · [ADR-004: Pruebas con Vitest y Fastify inject](../../../../../helpdesk-backend/docs/adr/ADR-004-vitest-fastify-inject.md)
- **Estándares:** [API Standards](../../../../../helpdesk-backend/docs/standards/api.md) · [Architecture Standards](../../../../../helpdesk-backend/docs/standards/architecture.md) · [Persistence Standards](../../../../../helpdesk-backend/docs/standards/persistence.md) · [Testing Standards](../../../../../helpdesk-backend/docs/standards/testing.md)
- **Documentación técnica:** [API-001: GET /incidents/summary](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents-summary) · [MD-003: Indicadores de incidencias](../../../../architecture/incidencias/models/MD-003-indicadores-incidencias.md) · [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md)

## Archivos afectados

```text
helpdesk-backend/
├── src/modules/incidents/
│   ├── ~ incidents.repository.ts                    # recuento de incidencias agrupado por status
│   ├── ~ incidents.schemas.ts                       # esquema de respuesta de los indicadores (MD-003)
│   ├── ~ incidents.service.ts                       # arma los indicadores: total = open + inProgress + resolved
│   └── ~ incidents.routes.ts                        # GET /summary con respuestas 200 y 500
└── test/
    ├── + incidents.summary.test.ts                  # inject: vacío, mezcla de estados, cambio de estado, 500
    ├── ~ incidents.repository.test.ts               # recuento por estado
    └── ~ openapi.test.ts                            # la ruta aparece en el contrato
```

## Plan de implementación

- [ ] **IT-01** — Añadir en `incidents.repository.ts` el método de recuento por estado con una sola consulta agrupada por `status`; sin importar tipos de Fastify
  Devuelve el recuento de `Open`, `In Progress` y `Resolved`; un estado sin incidencias cuenta `0`.
- [ ] **IT-02** — Definir en `incidents.schemas.ts` el esquema de respuesta `{ total, open, inProgress, resolved }`, todos enteros ≥ 0
- [ ] **IT-03** — Implementar en `incidents.service.ts` la obtención de los indicadores: mapear los recuentos a `open`, `inProgress` y `resolved` y calcular `total` como su suma; sin caché ni valores precalculados
- [ ] **IT-04** — Declarar `GET /summary` en `incidents.routes.ts` junto a `GET /:id`, sin esquema de query ni body, con respuestas `200` y `500` declaradas en el contrato
  La ruta estática se resuelve antes que la paramétrica; `GET /incidents/summary` nunca debe llegar al manejador de `/:id`.
- [ ] **IT-05** — Pruebas del repositorio: base vacía devuelve los tres recuentos en `0`; con incidencias en varios estados devuelve el recuento correcto de cada uno
- [ ] **IT-06** — Pruebas con `inject` del endpoint: sin incidencias responde `200` con los cuatro valores en `0`; con una mezcla de estados responde los recuentos esperados y `total` igual a su suma
  Los datos de prueba se siembran en la base `:memory:` con incidencias en `Open`, `In Progress` y `Resolved` (Object Mother).
- [ ] **IT-07** — Prueba de recálculo: registrar una incidencia y cambiar el estado de otra en la base de pruebas entre dos consultas, y verificar que la segunda respuesta refleja ambos cambios
- [ ] **IT-08** — Prueba de error: un fallo del repositorio durante la consulta responde `500` con `{statusCode, error, message}` sin exponer detalles internos
- [ ] **IT-09** — Verificar que `GET /incidents/summary` y su esquema de respuesta figuran en el documento OpenAPI y no alteran la operación `GET /incidents/{id}`
