---
id: TK-002
type: task
status: Ready
story: ./README.md
repository: helpdesk-frontend
---
# TK-002: Repositorio del listado de incidencias en el cliente

## Descripción

Ampliar la capa de acceso a la API de incidencias del frontend con la lectura del listado conforme al contrato de `GET /incidents`: modelos de la consulta (filtros, orden, página) y de la página de respuesta, funciones de mapeo entre la consulta, los parámetros de la dirección de la aplicación, los parámetros HTTP y la respuesta, el método de lectura paginada en `IncidentRepository` y los manejadores MSW que reproducen filtros, orden, paginación y el error `400` para las pruebas unitarias.

## Dependencias

- `IncidentRepository`, modelo `Incident`, opciones de categoría y prioridad y mapper de incidencias ([TK-004 de US-001](../US-001-registrar-incidencia/TK-004-repositorio-api-incidencias.md)) — se amplían con la lectura del listado.
- `getResourceCollection()` y `getApiUrl()` de `src/app/core/utils/async-resources.ts` — recurso de lista paginada con `loading`, `error`, `value` y `total`, y resolución de la URL.
- `ResourceException` de `src/app/core/utils/async-resources.ts` — conserva en `raw` la respuesta HTTP de error para extraer los `issues` del `400`.
- `HttpClient` / `HttpParams` de Angular — petición `GET` con los parámetros de query.
- `msw` y `src/mocks/handlers.ts` — handler de `GET /incidents` para las pruebas unitarias.

## Referencias

- **Arquitectura:** [ADR-010: Patrón Repository para REST](../../../../../helpdesk-frontend/docs/adr/ADR-010-repository-pattern-rest-api.md) · [ADR-012: Mappers como funciones puras](../../../../../helpdesk-frontend/docs/adr/ADR-012-feature-mappers-pure-functions.md) · [ADR-007: Mocks HTTP con MSW](../../../../../helpdesk-frontend/docs/adr/ADR-007-http-api-mocks-msw.md) · [ADR-003: Pruebas unitarias con Vitest y Testing Library](../../../../../helpdesk-frontend/docs/adr/ADR-003-unit-testing-vitest-testing-library.md) · [ADR-008 (backend): Contrato OpenAPI compartido](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md)
- **Estándares:** [Architecture Standards](../../../../../helpdesk-frontend/docs/standards/architecture.md) · [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [API-001: GET /incidents](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents) · [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md) · [FL-002: Consulta del listado de incidencias](../../../../architecture/incidencias/flows/FL-002-consulta-listado-incidencias.md)

## Archivos afectados

```text
helpdesk-frontend/
├── src/
│   ├── ~ mocks/handlers.ts                                  # handler MSW de GET /incidents con datos de ejemplo
│   └── app/features/incidents/
│       ├── models/
│       │   └── ~ incident.ts                                # estados, consulta del listado, página de respuesta, tamaño 20
│       ├── mappers/
│       │   ├── ~ incident-mapper.ts                         # consulta ↔ dirección, consulta → params HTTP, respuesta e issues
│       │   └── ~ incident-mapper.spec.ts                    # unit del mapeo del listado (crear si aún no existe)
│       └── repositories/
│           ├── ~ incident-repository.ts                     # findAll() vía getResourceCollection
│           └── ~ incident-repository.spec.ts                # unit con MSW: filtros, orden, página, 400 y red
```

## Plan de implementación

- [x] **IT-01** — Ampliar `incident.ts` con las opciones de estado (`Open`, `In Progress`, `Resolved`, mostradas literales), el orden (`asc` | `desc`), la consulta del listado (filtros opcionales, orden, página), la página de respuesta del contrato (`items`, `total`, `page`, `pageSize`, `totalPages`) y la constante de tamaño de página 20
  Los tipos siguen el contrato OpenAPI de [API-001](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents), sin redefinir a mano campos ajenos al contrato.
- [x] **IT-02** — Añadir al mapper la conversión de la consulta a los parámetros HTTP: omitir los filtros sin valor y enviar `order` y `page`
  `In Progress` viaja codificado en la URL; los valores no se validan en el mapper.
- [x] **IT-03** — Añadir al mapper la conversión entre la consulta y los parámetros de la dirección de la aplicación, en ambos sentidos
  De la dirección a la consulta: valores por defecto `order = desc` y `page = 1` cuando faltan, y valores conservados tal cual llegan (sin validarlos), para que la API rechace con `400` un valor inválido. De la consulta a la dirección: omitir filtros vacíos y los valores por defecto.
- [x] **IT-04** — Añadir al mapper la conversión de la página de respuesta al resultado de colección (`data`, `total`) y la extracción de los errores de validación de un `400` como lista de parámetro (de `instancePath`, sin la barra inicial) y motivo
- [x] **IT-05** — Implementar `findAll()` en `IncidentRepository`: recurso con `getResourceCollection()` cuya factoría recibe la consulta y llama a `GET` sobre `getApiUrl('incidents')` con los parámetros mapeados
- [x] **IT-06** — Añadir el handler MSW de `GET /incidents` sobre una colección determinista de incidencias de ejemplo (más de 20, con estados, prioridades, categorías y fechas variadas)
  Aplica los filtros presentes, el orden por `createdAt` con desempate por `id`, páginas de 20 con los totales del contrato e `items` vacío fuera de rango; responde `400` con la estructura `{statusCode, error, message, issues[]}` ante un valor fuera de enum o una página inválida. Los specs sobreescriben respuestas puntuales con `server.use(...)`.
- [x] **IT-07** — Pruebas unitarias del mapper (consulta ↔ dirección, params HTTP sin filtros vacíos, respuesta a colección, extracción de `issues`) y del repositorio con MSW: parámetros enviados por filtro, combinación y orden, `In Progress` codificado, `value` y `total` de la página, `400` con `issues` accesibles desde el error, `500` y fallo de red; datos de prueba con Object Mother
