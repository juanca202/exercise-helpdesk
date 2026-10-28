---
id: TK-001
type: task
status: Ready
story: ./README.md
repository: helpdesk-backend
---
# TK-001: Endpoint de listado de incidencias

## Descripción

Exponer `GET /incidents` en el módulo `incidents`: devolver una página del listado filtrada por `status`, `priority` y `category` (un valor por filtro, combinables: una incidencia aparece solo si cumple todos los filtros presentes), ordenada por `createdAt` en el sentido de `order` (`desc` por defecto, `asc` a petición; a igualdad de `createdAt`, por `id` en el mismo sentido) y paginada con tamaño fijo de 20 (`page` base 1, por defecto 1). La respuesta `200` es `{ items, total, page, pageSize, totalPages }`; una página fuera de rango responde `200` con `items: []` y los totales reales; un parámetro inválido responde `400` con un elemento en `issues` por parámetro y su motivo en español; los parámetros no declarados se ignoran. La operación queda publicada en el contrato OpenAPI.

## Dependencias

- `incidents.repository.ts` ([TK-001 de US-001](../US-001-registrar-incidencia/TK-001-persistencia-sqlite-incidencias.md)) — tabla `incidents` y mapeo `created_at` ↔ `createdAt`; se amplía con el recuento y la consulta paginada.
- `incidents.schemas.ts`, `incidents.service.ts` e `incidents.routes.ts` ([TK-002 de US-001](../US-001-registrar-incidencia/TK-002-endpoint-crear-incidencia.md)) — enums de categoría y prioridad, esquema de respuesta de la incidencia y módulo registrado con `prefix: '/incidents'`; se amplían con la operación de listado.
- `@fastify/type-provider-zod` / `zod` — esquema de la query (con coerción de `page`) y de la respuesta paginada.
- `src/plugins/error-handler.ts` — respuesta `400` con `{statusCode, error, message, issues[]}` ya ajustada en [TK-002 de US-001](../US-001-registrar-incidencia/TK-002-endpoint-crear-incidencia.md) para exponer los mensajes propios en español.

## Referencias

- **Arquitectura:** [ADR-001: API First con Fastify, Zod y OpenAPI](../../../../../helpdesk-backend/docs/adr/ADR-001-api-first-fastify-zod-openapi.md) · [ADR-008: Contrato OpenAPI compartido](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md) · [ADR-007: Persistencia local con SQLite](../../../../../helpdesk-backend/docs/adr/ADR-007-sqlite-persistencia-local.md) · [ADR-006: Plugins encapsulados por feature](../../../../../helpdesk-backend/docs/adr/ADR-006-plugins-encapsulados-por-feature.md) · [ADR-004: Pruebas con Vitest e inject](../../../../../helpdesk-backend/docs/adr/ADR-004-vitest-fastify-inject.md)
- **Estándares:** [API Standards](../../../../../helpdesk-backend/docs/standards/api.md) · [Persistence Standards](../../../../../helpdesk-backend/docs/standards/persistence.md) · [Architecture Standards](../../../../../helpdesk-backend/docs/standards/architecture.md) · [Testing Standards](../../../../../helpdesk-backend/docs/standards/testing.md)
- **Documentación técnica:** [API-001: GET /incidents](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents) · [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md) · [FL-002: Consulta del listado de incidencias](../../../../architecture/incidencias/flows/FL-002-consulta-listado-incidencias.md)

## Archivos afectados

```text
helpdesk-backend/
├── src/modules/incidents/
│   ├── ~ incidents.schemas.ts                       # esquema de query del listado y de la página de respuesta
│   ├── ~ incidents.repository.ts                    # count y findPage con filtros, orden y LIMIT/OFFSET
│   ├── ~ incidents.service.ts                       # listado: total, totalPages e items de la página
│   └── ~ incidents.routes.ts                        # GET / con respuestas 200 y 400
└── test/
    ├── ~ incidents.repository.test.ts               # filtros, orden con desempate y paginación
    ├── + incidents.list.test.ts                     # inject: casos válidos, fuera de rango e inválidos
    └── ~ openapi.test.ts                            # GET /incidents aparece en el contrato
```

## Plan de implementación

- [x] **IT-01** — Definir en `incidents.schemas.ts` el esquema de la query del listado: `status`, `priority`, `category`, `order` y `page`, todos opcionales, con mensajes de validación propios en español por parámetro
  `status` con los valores literales `Open`, `In Progress` y `Resolved` (reutilizar el enum de estado si TK-002 de US-001 ya lo declara); `priority` y `category` con los enums existentes; `order` en `asc` | `desc` con `desc` por defecto; `page` convertido a número, entero ≥ 1, con `1` por defecto. Esquema no estricto: las claves no declaradas (p. ej. `pageSize`) se descartan sin error. Un filtro repetido (varios valores) no cumple el enum y se rechaza.
- [x] **IT-02** — Definir el esquema de respuesta `200`: `items` (incidencias sin comentarios, máximo 20), `total`, `page`, `pageSize` (siempre 20) y `totalPages`
- [x] **IT-03** — Ampliar `incidents.repository.ts` con el recuento de incidencias que cumplen los filtros y la consulta de una página (filtros, orden, límite y desplazamiento)
  Filtros como parámetros enlazados, sin concatenar valores de la petición; orden `created_at` y luego `id`, ambos en el sentido de `order` (tomado del valor ya validado); el formato ISO 8601 UTC de `created_at` fijado en TK-001 de US-001 ordena cronológicamente. Sin tipos de Fastify en el repositorio.
- [x] **IT-04** — Implementar el listado en `incidents.service.ts`: desplazamiento `(page - 1) × 20`, `totalPages = ceil(total / 20)` (`0` cuando `total = 0`) y la página solicitada en `page`, también cuando supera `totalPages` (`items` vacío)
- [x] **IT-05** — Registrar `GET /` en `incidents.routes.ts` con las respuestas `200` y `400` declaradas en el contrato OpenAPI
- [x] **IT-06** — Pruebas del repositorio con fechas de creación fijas: cada filtro por separado y combinados, orden descendente y ascendente, desempate por `id` en el mismo sentido, límite y desplazamiento sin repetir ni omitir elementos entre páginas
- [x] **IT-07** — Pruebas con `inject` de `GET /incidents`
  Sin filtros devuelve todas (orden descendente por defecto); filtros individuales y combinados, incluido `status=In%20Progress`; `order=asc`; 21 incidencias → página 1 con 20 y página 2 con 1, `total` 21 y `totalPages` 2; sin incidencias → `total` 0, `totalPages` 0, `items` vacío; `page` mayor que `totalPages` → `200` con `items` vacío y totales reales; `400` con un `issue` por parámetro (`instancePath` = `/<parámetro>`, motivo en español) para `status`, `priority` y `category` fuera de su enum, `order` distinto de `asc`/`desc`, `page` no numérico, decimal o < 1, y un filtro repetido; varios parámetros inválidos → varios `issues`; `pageSize` u otro parámetro no declarado no altera el resultado; la operación figura en el OpenAPI
