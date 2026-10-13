# TK-002: Mutación de cambio de estado en el repositorio de incidencias

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-005](./README.md)
**Repositorio:** helpdesk-frontend

## Descripción

Disponer en el frontend del acceso a `PATCH /incidents/{id}/status` conforme al contrato: la mutación `changeStatus` en `IncidentRepository`, el mapper del cuerpo `{ status }` y de la respuesta, y un tipo de error de cambio de estado que distingue la transición rechazada (`409`) y conserva el `message` de la API para mostrarlo tal cual. Las pruebas unitarias cuentan con manejadores MSW de la operación para las respuestas `200`, `400`, `404`, `409` y `500`.

## Dependencias

- `IncidentRepository`, modelo `Incident` y mapper de incidencias ([TK-004 de US-001](../US-001-registrar-incidencia/TK-004-repositorio-api-incidencias.md)) — se amplían con el cambio de estado.
- `getMutations`, `getApiUrl` y `ResourceException` de `src/app/core/utils/async-resources.ts` — mutación y acceso a la respuesta HTTP original del error.
- `msw` y `src/mocks/handlers.ts` — mock HTTP en pruebas unitarias.

## Referencias

- **Arquitectura:** [ADR-010: Patrón Repository para REST](../../../../../helpdesk-frontend/docs/adr/ADR-010-repository-pattern-rest-api.md) · [ADR-012: Mappers como funciones puras](../../../../../helpdesk-frontend/docs/adr/ADR-012-feature-mappers-pure-functions.md) · [ADR-007: Mocks HTTP con MSW](../../../../../helpdesk-frontend/docs/adr/ADR-007-http-api-mocks-msw.md) · [ADR-003: Pruebas unitarias con Vitest y Testing Library](../../../../../helpdesk-frontend/docs/adr/ADR-003-unit-testing-vitest-testing-library.md)
- **Estándares:** [Architecture Standards](../../../../../helpdesk-frontend/docs/standards/architecture.md) (`rest-repository-pattern`, `feature-entity-mappers`) · [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [API-001: PATCH /incidents/{id}/status](../../../../architecture/incidencias/apis/API-001-incidencias.md#patch-incidents-id-status) · [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md) · [DG-001: Estados de la incidencia](../../../../architecture/incidencias/diagrams/DG-001-estados-incidencia.md)

## Archivos afectados

```text
helpdesk-frontend/
└── src/
    ├── ~ mocks/handlers.ts                                  # handlers MSW de PATCH /incidents/:id/status
    └── app/features/incidents/
        ├── models/
        │   └── ~ incident.ts                                # IncidentStatus y tipo de error de cambio de estado
        ├── mappers/
        │   ├── ~ incident-mapper.ts                         # cuerpo { status }, respuesta y error de la API → error de cambio de estado
        │   └── ~ incident-mapper.spec.ts                    # unit de las nuevas funciones del mapper
        └── repositories/
            ├── ~ incident-repository.ts                     # mutación changeStatus vía getMutations
            └── ~ incident-repository.spec.ts                # unit con MSW de changeStatus
```

## Plan de implementación

- [ ] **IT-01** — Exponer en el modelo el tipo `IncidentStatus` (`Open` | `In Progress` | `Resolved`, literales del contrato) si aún no es un tipo propio, y el tipo de error de cambio de estado con su motivo (`rejected` para `409`, `not-found`, `invalid`, `unexpected`) y el mensaje a mostrar
- [ ] **IT-02** — Añadir al mapper las funciones puras que construyen el cuerpo `{ status }` y convierten la respuesta `200` en `Incident`
- [ ] **IT-03** — Añadir al mapper la función pura que convierte un fallo de la mutación en el error de cambio de estado leyendo la respuesta HTTP original de `ResourceException`
  `409` → motivo `rejected` con el `message` de la API tal cual; `404` → `not-found`; `400` → `invalid`; `500`, error de red u otro → `unexpected`; los mensajes no provenientes de la API se redactan en español y comprensibles (p. ej. «No se pudo cambiar el estado. Inténtalo de nuevo.»).
- [ ] **IT-04** — Añadir a `IncidentRepository` la mutación `changeStatus({ id, status })` mediante `getMutations()`, con `PATCH` a `getApiUrl('incidents/{id}/status')` y resultado `Incident`
- [ ] **IT-05** — Añadir los handlers MSW de `PATCH /incidents/:id/status`: por defecto `200` con la incidencia y el `status` recibido; variantes `400` (`issues` en `/status`), `404` (`error: "Not Found"`), `409` (`error: "Conflict"` con mensaje en español) y `500` para sobrescribir en pruebas
- [ ] **IT-06** — Pruebas unitarias del mapper: cuerpo, respuesta y conversión de cada código de error al motivo y mensaje esperados
- [ ] **IT-07** — Pruebas unitarias del repositorio con MSW: `changeStatus` envía el cuerpo esperado y devuelve la incidencia actualizada; con `409` el error se convierte en `rejected` con el mensaje de la API; con `400`, `404`, `500` y error de red, en el motivo correspondiente
