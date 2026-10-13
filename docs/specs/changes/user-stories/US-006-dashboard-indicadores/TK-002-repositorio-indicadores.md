# TK-002: Lectura de indicadores en el repositorio de incidencias

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-006](./README.md)
**Repositorio:** helpdesk-frontend

## Descripción

Disponer en el frontend de la lectura de los indicadores conforme al contrato: modelo `IncidentSummary` con `total`, `open`, `inProgress` y `resolved`, método de lectura `getSummary` en `IncidentRepository` que consulta `GET /incidents/summary`, y manejadores MSW de esa operación (con datos, todo en cero y error `500`) para las pruebas unitarias.

## Dependencias

- `IncidentRepository` y la feature `src/app/features/incidents/` ([TK-004 de US-001](../US-001-registrar-incidencia/TK-004-repositorio-api-incidencias.md)) — repositorio que se amplía con la lectura.
- `getResource` y `getApiUrl` de `src/app/core/utils/async-resources.ts` — lectura de recurso único y resolución de la URL.
- `msw` y `src/mocks/handlers.ts` — mock HTTP en pruebas unitarias.

## Referencias

- **Arquitectura:** [ADR-010: Patrón Repository para REST](../../../../../helpdesk-frontend/docs/adr/ADR-010-repository-pattern-rest-api.md) · [ADR-007: Mocks HTTP con MSW](../../../../../helpdesk-frontend/docs/adr/ADR-007-http-api-mocks-msw.md) · [ADR-003: Pruebas unitarias con Vitest y Testing Library](../../../../../helpdesk-frontend/docs/adr/ADR-003-unit-testing-vitest-testing-library.md) · [ADR-008 (backend): Contrato OpenAPI compartido](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md)
- **Estándares:** [Architecture Standards](../../../../../helpdesk-frontend/docs/standards/architecture.md) · [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [MD-003: Indicadores de incidencias](../../../../architecture/incidencias/models/MD-003-indicadores-incidencias.md) · [API-001: GET /incidents/summary](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents-summary)

## Archivos afectados

```text
helpdesk-frontend/
└── src/
    ├── ~ mocks/handlers.ts                                  # handler MSW de GET /incidents/summary (200 con datos)
    └── app/features/incidents/
        ├── models/
        │   └── + incident-summary.ts                        # IncidentSummary: total, open, inProgress, resolved
        └── repositories/
            ├── ~ incident-repository.ts                     # lectura getSummary vía getResource
            └── ~ incident-repository.spec.ts                # unit con MSW: datos, ceros y 500
```

## Plan de implementación

- [ ] **IT-01** — Definir el modelo `IncidentSummary` con los cuatro campos enteros del contrato (`total`, `open`, `inProgress`, `resolved`), con TSDoc que remita a MD-003
- [ ] **IT-02** — Añadir a `IncidentRepository` la lectura `getSummary` mediante `getResource()` sobre `getApiUrl('incidents/summary')`, sin parámetros
  Expone valor, carga, error y recarga del recurso para que la página muestre carga, error y reintento.
- [ ] **IT-03** — Añadir en `src/mocks/handlers.ts` el handler por defecto de `GET /incidents/summary` con una respuesta `200` coherente (`total` igual a la suma de los tres estados)
- [ ] **IT-04** — Definir las variantes de respuesta que las pruebas aplican con `server.use(...)`: `200` con los cuatro valores en `0` y `500` con `{statusCode, error, message}`
- [ ] **IT-05** — Pruebas unitarias de `getSummary`: con datos devuelve el modelo tal cual; sin incidencias devuelve los cuatro ceros sin error; ante `500` y fallo de red expone el error y deja de estar en carga
