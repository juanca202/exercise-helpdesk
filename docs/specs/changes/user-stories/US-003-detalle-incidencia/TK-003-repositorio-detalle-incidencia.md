# TK-003: Repositorio y modelo del detalle de incidencia en el cliente

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-003](./README.md)
**Repositorio:** helpdesk-frontend

## Descripción

Disponer en el frontend de la lectura del detalle de una incidencia conforme al contrato de `GET /incidents/{id}`: modelos `Comment` e `IncidentDetail`, mapper de la respuesta de la API al modelo, consulta de detalle en `IncidentRepository` que permite distinguir «no encontrada» (`404`) de los demás fallos (`500` o error de red), y manejadores MSW del endpoint para las pruebas unitarias.

## Dependencias

- `IncidentRepository`, modelo `Incident`, opciones de categoría y prioridad y `incident-mapper.ts` ([TK-004 de US-001](../US-001-registrar-incidencia/TK-004-repositorio-api-incidencias.md)) — se amplían con la lectura del detalle.
- `getResource`, `getApiUrl` y `ResourceException` de `src/app/core/utils/async-resources.ts` — recurso reactivo de lectura con estados de carga, valor y error.
- `msw` y `src/mocks/handlers.ts` ([TK-004 de US-001](../US-001-registrar-incidencia/TK-004-repositorio-api-incidencias.md)) — mock HTTP en pruebas unitarias.

## Referencias

- **Arquitectura:** [ADR-010: Patrón Repository para REST](../../../../../helpdesk-frontend/docs/adr/ADR-010-repository-pattern-rest-api.md) · [ADR-012: Mappers como funciones puras](../../../../../helpdesk-frontend/docs/adr/ADR-012-feature-mappers-pure-functions.md) · [ADR-007: Mocks HTTP con MSW](../../../../../helpdesk-frontend/docs/adr/ADR-007-http-api-mocks-msw.md) · [ADR-003: Pruebas unitarias con Vitest y Testing Library](../../../../../helpdesk-frontend/docs/adr/ADR-003-unit-testing-vitest-testing-library.md) · [ADR-008: Contrato OpenAPI compartido](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md)
- **Estándares:** [Architecture Standards](../../../../../helpdesk-frontend/docs/standards/architecture.md) · [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [API-001: GET /incidents/{id}](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents-id) · [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md) · [MD-002: Comentario](../../../../architecture/incidencias/models/MD-002-comentario.md) · [FL-003: Consulta del detalle de una incidencia](../../../../architecture/incidencias/flows/FL-003-consulta-detalle-incidencia.md)

## Archivos afectados

```text
helpdesk-frontend/
└── src/
    ├── ~ mocks/handlers.ts                                  # handler MSW de GET /incidents/:id (200 por defecto)
    └── app/features/incidents/
        ├── models/
        │   ├── + comment.ts                                 # Comment (MD-002)
        │   └── ~ incident.ts                                # IncidentDetail = Incident + comments
        ├── mappers/
        │   ├── ~ incident-mapper.ts                         # respuesta de detalle → IncidentDetail (funciones puras)
        │   └── ~ incident-mapper.spec.ts                    # mapeo del detalle con y sin comentarios
        ├── repositories/
        │   ├── ~ incident-repository.ts                     # consulta de detalle por id vía getResource
        │   └── ~ incident-repository.spec.ts                # unit con MSW: 200, 404, 500, red
        └── testing/
            └── + incident-detail.mother.ts                  # Object Mother de detalle y comentarios
```

## Plan de implementación

- [ ] **IT-01** — Definir el modelo `Comment` (`id`, `incidentId`, `content`, `author`, `createdAt`) y el modelo `IncidentDetail` como la incidencia más su lista de comentarios
- [ ] **IT-02** — Implementar en el mapper la conversión pura de la respuesta de `GET /incidents/{id}` a `IncidentDetail`, conservando el orden de `comments` que entrega la API
- [ ] **IT-03** — Añadir a `IncidentRepository` la consulta de detalle por id con `getResource()` sobre `getApiUrl('incidents/{id}')`, con el id codificado en la URL
- [ ] **IT-04** — Exponer el resultado de forma que el consumidor distinga «no encontrada» (`404`) de un fallo recuperable (`500` o error de red) y pueda reintentar la última consulta
  El `404` no dispara la notificación global de error: la página lo presenta como estado propio.
- [ ] **IT-05** — Crear el Object Mother del detalle con variantes con comentarios (orden ascendente) y sin comentarios
- [ ] **IT-06** — Añadir en `src/mocks/handlers.ts` el handler de `GET /incidents/:id` con `200` y comentarios por defecto; las variantes `200` sin comentarios, `404` con `{ statusCode, error, message }` y `500` se sobrescriben por prueba con `server.use(...)`
- [ ] **IT-07** — Pruebas unitarias del mapper y del repositorio: detalle con comentarios en el orden recibido, detalle con `comments: []`, `404` reconocido como «no encontrada», `500` y error de red reconocidos como fallo recuperable, y reintento que vuelve a consultar
