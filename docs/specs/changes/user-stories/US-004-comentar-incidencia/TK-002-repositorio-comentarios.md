# TK-002: Repositorio y mapper de comentarios en el cliente

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-004](./README.md)
**Repositorio:** helpdesk-frontend

## Descripción

Disponer en el frontend del acceso a `POST /incidents/{id}/comments` conforme al contrato: tipo del cuerpo de alta, mapper de funciones puras entre los datos del formulario de comentario y el cuerpo de la API, mutación `addComment` en `IncidentRepository` que devuelve el comentario creado, y manejadores MSW que reproducen las respuestas `201`, `400` con `issues`, `404` y `500` para las pruebas unitarias.

## Dependencias

- `IncidentRepository` ([TK-004 de US-001](../US-001-registrar-incidencia/TK-004-repositorio-api-incidencias.md)) — se amplía su bloque de mutaciones con `addComment`.
- Modelo `Comment` ([TK-003 de US-003](../US-003-detalle-incidencia/TK-003-repositorio-detalle-incidencia.md)) — tipo de la respuesta `201`.
- Helpers de `src/app/core/utils/async-resources.ts` (`getMutations`, `getApiUrl`) — mutación y resolución de la URL.
- `msw` y `src/mocks/handlers.ts` — mock HTTP en pruebas unitarias.

## Referencias

- **Arquitectura:** [ADR-010: Patrón Repository para REST](../../../../../helpdesk-frontend/docs/adr/ADR-010-repository-pattern-rest-api.md) · [ADR-012: Mappers como funciones puras](../../../../../helpdesk-frontend/docs/adr/ADR-012-feature-mappers-pure-functions.md) · [ADR-007: Mocks HTTP con MSW](../../../../../helpdesk-frontend/docs/adr/ADR-007-http-api-mocks-msw.md) · [ADR-003: Pruebas unitarias con Vitest y Testing Library](../../../../../helpdesk-frontend/docs/adr/ADR-003-unit-testing-vitest-testing-library.md) · [ADR-008: Contrato OpenAPI compartido](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md)
- **Estándares:** [Architecture Standards](../../../../../helpdesk-frontend/docs/standards/architecture.md) · [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [MD-002: Comentario](../../../../architecture/incidencias/models/MD-002-comentario.md) · [API-001: POST /incidents/{id}/comments](../../../../architecture/incidencias/apis/API-001-incidencias.md#post-incidents-id-comments) · [FL-004: Registro de un comentario](../../../../architecture/incidencias/flows/FL-004-registro-comentario.md)

## Archivos afectados

```text
helpdesk-frontend/
└── src/
    ├── ~ mocks/handlers.ts                                  # handlers MSW de POST /incidents/:id/comments
    └── app/features/incidents/
        ├── models/
        │   └── ~ comment.ts                                 # tipo NewComment (cuerpo de alta)
        ├── utils/
        │   ├── + comment-mapper.ts                          # formulario ↔ cuerpo de la API (funciones puras)
        │   └── + comment-mapper.spec.ts                     # unit del mapper
        └── repositories/
            ├── ~ incident-repository.ts                     # mutación addComment vía getMutations
            └── ~ incident-repository.spec.ts                # unit de addComment con MSW
```

## Plan de implementación

- [ ] **IT-01** — Añadir al modelo de comentario el tipo `NewComment` con el único campo `content`, alineado con el contrato
- [ ] **IT-02** — Implementar `CommentMapper` en `utils/comment-mapper.ts` con funciones puras entre los datos del formulario y el cuerpo de `POST /incidents/{id}/comments`
  El cuerpo resultante contiene solo `content`, recortado; nunca `id`, `incidentId`, `author` ni `createdAt`. Sin validación ni formateo de UI en el mapper.
- [ ] **IT-03** — Añadir la mutación `addComment` a `IncidentRepository` mediante `getMutations()`, con parámetros id de incidencia y cuerpo, URL `getApiUrl('incidents/{id}/comments')` y resultado `Comment`
  Los errores HTTP se propagan sin transformar para que el formulario distinga `400` (con `issues`), `404` y `500`/red.
- [ ] **IT-04** — Añadir en `src/mocks/handlers.ts` los handlers de `POST /incidents/:id/comments`: `201` con el comentario (`author` «Agente demo»), `400` con `{statusCode, error, message, issues[]}` e `instancePath` `/content`, `404` con `{statusCode, error: "Not Found", message}` y `500` genérico
- [ ] **IT-05** — Pruebas unitarias del mapper: cuerpo solo con `content` recortado y sin mutar la entrada
- [ ] **IT-06** — Pruebas unitarias de `addComment` con MSW: `201` devuelve el comentario, `400` expone los `issues`, `404` y `500` quedan como error, y el estado de envío se activa durante la petición
  Datos de prueba construidos con el Object Mother de comentarios de [TK-003 de US-003](../US-003-detalle-incidencia/TK-003-repositorio-detalle-incidencia.md) (`testing/incident-detail.mother.ts`).
