# TK-004: Repositorio y modelo de incidencias en el cliente

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-001](./README.md)
**Repositorio:** helpdesk-frontend

## Descripción

Disponer en el frontend de la capa de acceso a la API de incidencias conforme al contrato: modelo y opciones de categoría y prioridad, mapper entre el formulario y el cuerpo de la API, `IncidentRepository` con la mutación de registro (`POST /incidents`) y manejadores MSW para las pruebas unitarias.

## Dependencias

- `BaseRepository` y helpers de `src/app/core/utils/async-resources.ts` (`getMutations`, `getApiUrl`) — patrón Repository del proyecto.
- `msw` y `src/mocks/handlers.ts` — mock HTTP en pruebas unitarias.
- `src/app/features/incidents/` — feature nueva que aloja modelos, mapper y repositorio.

## Referencias

- **Arquitectura:** [ADR-010: Patrón Repository para REST](../../../../../helpdesk-frontend/docs/adr/ADR-010-repository-pattern-rest-api.md) · [ADR-012: Mappers como funciones puras](../../../../../helpdesk-frontend/docs/adr/ADR-012-feature-mappers-pure-functions.md) · [ADR-007: Mocks HTTP con MSW](../../../../../helpdesk-frontend/docs/adr/ADR-007-http-api-mocks-msw.md) · [ADR-001: Arquitectura por capas y features](../../../../../helpdesk-frontend/docs/adr/ADR-001-hybrid-layered-feature-architecture.md)
- **Estándares:** [Architecture Standards](../../../../../helpdesk-frontend/docs/standards/architecture.md) · [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md) · [API-001: POST /incidents](../../../../architecture/incidencias/apis/API-001-incidencias.md#post-incidents)

## Archivos afectados

```text
helpdesk-frontend/
├── src/
│   ├── ~ mocks/handlers.ts                                  # handler MSW de POST /incidents
│   └── app/features/incidents/
│       ├── models/
│       │   └── + incident.ts                                # Incident, NewIncident, categorías y prioridades
│       ├── mappers/
│       │   └── + incident-mapper.ts                         # formulario ↔ cuerpo de la API (funciones puras)
│       └── repositories/
│           ├── + incident-repository.ts                     # mutación de registro vía getMutations
│           └── + incident-repository.spec.ts                # unit con MSW
```

## Plan de implementación

- [ ] **IT-01** — Definir el modelo `Incident`/`NewIncident` y las opciones de categoría y prioridad con su etiqueta en español (Hardware, Software, Accesos y permisos, Red y conectividad, Correo y comunicación, Otro; Baja, Media, Alta) ligadas a los códigos del contrato
- [ ] **IT-02** — Implementar el mapper de funciones puras entre los datos del formulario y el cuerpo de `POST /incidents`
- [ ] **IT-03** — Implementar `IncidentRepository` (extiende `BaseRepository`) con la mutación de registro mediante `getMutations()` y URLs con `getApiUrl('incidents')`
- [ ] **IT-04** — Añadir el handler MSW de `POST /incidents` (`201` con id, `Open` y `createdAt`; `400` con la estructura `{statusCode, error, message, issues[]}`)
- [ ] **IT-05** — Pruebas unitarias del mapper y del repositorio: éxito, error `400` con `issues` y error de red
