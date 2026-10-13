# API-001: Incidencias

Operaciones sobre el recurso incidencia bajo `/incidents`, publicadas en el contrato OpenAPI 3.1 generado desde esquemas Zod ([ADR-001](../../../../helpdesk-backend/docs/adr/ADR-001-api-first-fastify-zod-openapi.md), [ADR-008](../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md)). Cubre registro, listado filtrado y paginado, indicadores, detalle con comentarios, alta de comentarios (subrecurso `/incidents/{id}/comments`) y cambio de estado (`/incidents/{id}/status`). Fuera: edición o borrado de incidencias y de comentarios.

## Operaciones

| Ancla | Método y ruta | Operación | Descripción |
| ----- | ------------- | --------- | ----------- |
| [#post-incidents](#post-incidents) | `POST /incidents` | Registrar incidencia | Crea una incidencia en estado `Open` |
| [#get-incidents](#get-incidents) | `GET /incidents` | Listar incidencias | Listado filtrable por estado, prioridad y categoría, ordenado por `createdAt` y paginado de 20 en 20 |
| [#get-incidents-summary](#get-incidents-summary) | `GET /incidents/summary` | Consultar indicadores | Total de incidencias y recuento por estado, calculados al consultar |
| [#get-incidents-id](#get-incidents-id) | `GET /incidents/{id}` | Consultar detalle de incidencia | Devuelve la incidencia con sus comentarios en orden cronológico ascendente |
| [#post-incidents-id-comments](#post-incidents-id-comments) | `POST /incidents/{id}/comments` | Agregar comentario | Registra un comentario del usuario de demostración en una incidencia existente |
| [#patch-incidents-id-status](#patch-incidents-id-status) | `PATCH /incidents/{id}/status` | Cambiar estado | Aplica una transición de estado permitida y devuelve la incidencia actualizada |

---

<a id="post-incidents"></a>
### `POST /incidents` — Registrar incidencia

Registra una incidencia con los datos aportados por el empleado; el sistema asigna `id`, `status = Open` y `createdAt`. Consumida por el formulario de registro ([FL-001](../flows/FL-001-registro-incidencia.md)).

**Request**

| Parámetro | Ubicación | Tipo | Requerido | Descripción |
| --------- | --------- | ---- | --------- | ----------- |
| title | body | string | Sí | Validaciones de `title` en [MD-001](../models/MD-001-incidencia.md): se recorta (trim) y luego 5–100 |
| description | body | string | Sí | Validaciones de `description` en [MD-001](../models/MD-001-incidencia.md): se recorta (trim) y luego 10–2000 |
| category | body | enum `IncidentCategory` | Sí | Valores de [MD-001](../models/MD-001-incidencia.md) |
| priority | body | enum `IncidentPriority` | Sí | `LOW` \| `MEDIUM` \| `HIGH` ([MD-001](../models/MD-001-incidencia.md)) |

`id`, `status` y `createdAt` no forman parte del request (BR-01). El esquema del body es **estricto**: si el cliente envía `id`, `status`, `createdAt` (o cualquier otra clave no declarada), la petición se rechaza con `400` y no se persiste nada.

```json
{
  "title": "No puedo acceder a la VPN",
  "description": "Desde esta mañana la VPN rechaza mis credenciales con error de autenticación.",
  "category": "NETWORK_CONNECTIVITY",
  "priority": "HIGH"
}
```

**Responses**

| Código | Condición | Cuerpo |
| ------ | --------- | ------ |
| 201 | Incidencia registrada y persistida | [MD-001](../models/MD-001-incidencia.md) completo (con `id`, `status = Open`, `createdAt`) |
| 400 | Cuerpo ausente o no JSON; algún campo ausente, vacío (también solo espacios), fuera de longitud tras el recorte o con valor fuera del enum (AC-004..AC-007); claves no declaradas en el body, incluidos `id`, `status` o `createdAt`. No se persiste nada | Error de validación del proyecto (ver estructura abajo) |
| 500 | Fallo inesperado al persistir | Error genérico del proyecto: `{ statusCode, error, message }` |

```json
{
  "id": "6f1c2b0e-8d3a-4f5e-9a71-2c4d5e6f7a8b",
  "title": "No puedo acceder a la VPN",
  "description": "Desde esta mañana la VPN rechaza mis credenciales con error de autenticación.",
  "category": "NETWORK_CONNECTIVITY",
  "priority": "HIGH",
  "status": "Open",
  "createdAt": "2026-10-12T14:32:05.120Z"
}
```

**Estructura del error de validación (400).** Se conserva la estructura que ya emite el manejador de errores común del backend ([`src/plugins/error-handler.ts`](../../../../helpdesk-backend/src/plugins/error-handler.ts)) para toda petición que no cumple el esquema Zod: `{ statusCode, error, message, issues[] }`, con un elemento en `issues` por cada problema detectado. El campo inválido se obtiene de `instancePath` (`/title` → `title`) y el motivo de `message`, que es un **mensaje de validación propio en español** definido junto al esquema (no el mensaje por defecto de Zod). Ajustar el manejador común para ello es un cambio a realizar en la implementación.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| statusCode | integer | `400` |
| error | string | `"Bad Request"` |
| message | string | `"Request does not match the contract"` |
| issues | array | Un elemento por problema: `keyword` (código de Zod), `instancePath` (ruta JSON del campo), `schemaPath`, `message` (motivo), `params` (detalle del issue) |

```json
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": "Request does not match the contract",
  "issues": [
    {
      "keyword": "too_small",
      "instancePath": "/title",
      "schemaPath": "#/title/too_small",
      "message": "El título debe tener entre 5 y 100 caracteres",
      "params": { "origin": "string", "minimum": 5, "inclusive": true }
    },
    {
      "keyword": "invalid_value",
      "instancePath": "/priority",
      "schemaPath": "#/priority/invalid_value",
      "message": "La prioridad debe ser LOW, MEDIUM o HIGH",
      "params": { "values": ["LOW", "MEDIUM", "HIGH"] }
    }
  ]
}
```

> Los textos de `message` en `issues` del ejemplo ilustran el estilo de los mensajes propios en español (motivo comprensible, AC-007); su redacción exacta se fija en la implementación. No se normaliza la respuesta a `{ field, message }`.

<a id="get-incidents"></a>
### `GET /incidents` — Listar incidencias

Devuelve una página del listado de incidencias, filtrable por estado, prioridad y categoría (combinables con «Y»), ordenada por `createdAt` y paginada con tamaño fijo de 20 ([US-002](../../../specs/changes/user-stories/US-002-listar-filtrar-incidencias/README.md), [FL-002](../flows/FL-002-consulta-listado-incidencias.md)).

**Request**

| Parámetro | Ubicación | Tipo | Requerido | Descripción |
| --------- | --------- | ---- | --------- | ----------- |
| status | query | enum `IncidentStatus` | No | Un valor: `Open` \| `In Progress` \| `Resolved` (codificado en la URL: `In%20Progress`). Sin él, no filtra por estado |
| priority | query | enum `IncidentPriority` | No | Un valor: `LOW` \| `MEDIUM` \| `HIGH`. Sin él, no filtra por prioridad |
| category | query | enum `IncidentCategory` | No | Un valor de `category` de [MD-001](../models/MD-001-incidencia.md). Sin él, no filtra por categoría |
| order | query | enum `asc` \| `desc` | No | Sentido del orden por `createdAt`; por defecto `desc` (más recientes primero) |
| page | query | integer | No | Página solicitada, base 1; entero ≥ 1; por defecto `1` |

Ejemplo: `GET /incidents?status=In%20Progress&priority=HIGH&order=asc&page=2`

**Responses**

| Código | Condición | Cuerpo |
| ------ | --------- | ------ |
| 200 | Consulta válida, incluido sin resultados o con `page` mayor que `totalPages` (entonces `items` es `[]`) | Página de incidencias (estructura abajo) |
| 400 | Algún parámetro con valor fuera de su enum o de su rango (`page` no entero o < 1, `order` distinto de `asc`/`desc`) | Error de validación del proyecto (misma estructura que en [POST /incidents](#post-incidents)), un elemento en `issues` por parámetro inválido, con `instancePath` = `/<parámetro>` y `message` propio en español |
| 500 | Fallo inesperado al consultar | Error genérico del proyecto: `{ statusCode, error, message }` |

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| items | [MD-001](../models/MD-001-incidencia.md)[] | Incidencias de la página (sin comentarios), en el orden solicitado; máximo 20 |
| total | integer | Número total de incidencias que cumplen los filtros (todas las páginas) |
| page | integer | Página devuelta (la solicitada) |
| pageSize | integer | Tamaño de página; siempre `20` |
| totalPages | integer | `ceil(total / 20)`; `0` cuando `total = 0` |

```json
{
  "items": [
    {
      "id": "6f1c2b0e-8d3a-4f5e-9a71-2c4d5e6f7a8b",
      "title": "No puedo acceder a la VPN",
      "description": "Desde esta mañana la VPN rechaza mis credenciales con error de autenticación.",
      "category": "NETWORK_CONNECTIVITY",
      "priority": "HIGH",
      "status": "In Progress",
      "createdAt": "2026-10-12T14:32:05.120Z"
    }
  ],
  "total": 21,
  "page": 2,
  "pageSize": 20,
  "totalPages": 2
}
```

```json
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": "Request does not match the contract",
  "issues": [
    {
      "keyword": "invalid_value",
      "instancePath": "/status",
      "schemaPath": "#/status/invalid_value",
      "message": "El estado debe ser Open, In Progress o Resolved",
      "params": { "values": ["Open", "In Progress", "Resolved"] }
    },
    {
      "keyword": "too_small",
      "instancePath": "/page",
      "schemaPath": "#/page/too_small",
      "message": "La página debe ser un número entero mayor o igual a 1",
      "params": { "origin": "number", "minimum": 1, "inclusive": true }
    }
  ]
}
```

- **Filtros combinados:** cada filtro presente restringe el resultado; una incidencia aparece solo si cumple todos (BR-01). Sin filtros devuelve todas (AC-003).
- **Orden estable:** se ordena por `createdAt` en el sentido de `order` y, a igualdad de `createdAt`, por `id` en el mismo sentido, para que la paginación no repita ni omita elementos.
- **Página fuera de rango:** `page > totalPages` no es un error: `200` con `items: []` y `total`/`totalPages` reales (AC-005).
- **Parámetros de query no declarados** (p. ej. `pageSize`): se ignoran; el tamaño de página no es configurable (ver Observaciones del [README de la capability](../README.md)).

<a id="get-incidents-summary"></a>
### `GET /incidents/summary` — Consultar indicadores

Devuelve los indicadores del dashboard ([MD-003](../models/MD-003-indicadores-incidencias.md)), calculados sobre los datos persistidos en el momento de la petición ([US-006](../../../specs/changes/user-stories/US-006-dashboard-indicadores/README.md)).

**Request**

Sin parámetros ni body.

**Responses**

| Código | Condición | Cuerpo |
| ------ | --------- | ------ |
| 200 | Siempre que la consulta se complete, incluido sin incidencias (todos los valores en `0`) | [MD-003](../models/MD-003-indicadores-incidencias.md) |
| 500 | Fallo inesperado al consultar | Error genérico del proyecto: `{ statusCode, error, message }` |

```json
{
  "total": 12,
  "open": 5,
  "inProgress": 4,
  "resolved": 3
}
```

- **Precedencia de rutas:** `/incidents/summary` es una ruta estática y Fastify la resuelve antes que la paramétrica `/incidents/{id}`; aun así, `summary` no es un UUID v4 y nunca colisionaría con un id real.

<a id="get-incidents-id"></a>
### `GET /incidents/{id}` — Consultar detalle de incidencia

Devuelve la incidencia con todos sus datos y sus comentarios en orden cronológico ascendente ([US-003](../../../specs/changes/user-stories/US-003-detalle-incidencia/README.md), [FL-003](../flows/FL-003-consulta-detalle-incidencia.md)). Sirve también a la recuperación posterior al registro (AC-003 de [US-001](../../../specs/changes/user-stories/US-001-registrar-incidencia/README.md)).

**Request**

| Parámetro | Ubicación | Tipo | Requerido | Descripción |
| --------- | --------- | ---- | --------- | ----------- |
| id | path | string | Sí | `id` de la incidencia ([MD-001](../models/MD-001-incidencia.md)), devuelto por [POST /incidents](#post-incidents). El esquema del parámetro **no** exige formato UUID: un valor que no es un UUID v4 válido se trata como incidencia inexistente (`404`), no como error de validación |

**Responses**

| Código | Condición | Cuerpo |
| ------ | --------- | ------ |
| 200 | Incidencia encontrada | Detalle de incidencia: campos de [MD-001](../models/MD-001-incidencia.md) + `comments`: [MD-002](../models/MD-002-comentario.md)[] ordenados por `createdAt` ascendente; `[]` si no tiene comentarios |
| 404 | No existe una incidencia con ese id, incluido un id que no es un UUID v4 válido; mismo cuerpo en ambos casos (BR-01 de US-003) | Error de recurso no encontrado (estructura abajo) |
| 500 | Fallo inesperado al consultar | Error genérico del proyecto: `{ statusCode, error, message }` |

```json
{
  "id": "6f1c2b0e-8d3a-4f5e-9a71-2c4d5e6f7a8b",
  "title": "No puedo acceder a la VPN",
  "description": "Desde esta mañana la VPN rechaza mis credenciales con error de autenticación.",
  "category": "NETWORK_CONNECTIVITY",
  "priority": "HIGH",
  "status": "In Progress",
  "createdAt": "2026-10-12T14:32:05.120Z",
  "comments": [
    {
      "id": "0b7e9c1a-3f2d-4c8e-a5b6-7d8e9f0a1b2c",
      "incidentId": "6f1c2b0e-8d3a-4f5e-9a71-2c4d5e6f7a8b",
      "content": "Revisando las credenciales en el servidor VPN.",
      "author": "Agente demo",
      "createdAt": "2026-10-12T15:02:44.310Z"
    }
  ]
}
```

**Estructura del error de recurso no encontrado (404).** Usa la estructura del error genérico del proyecto `{ statusCode, error, message }` que emite el manejador común ([`src/plugins/error-handler.ts`](../../../../helpdesk-backend/src/plugins/error-handler.ts)) y es común a toda operación sobre `/incidents/{id}`. El manejador hoy toma `error` del nombre de la excepción; la implementación debe asegurar `error: "Not Found"`.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| statusCode | integer | `404` |
| error | string | `"Not Found"` |
| message | string | Mensaje propio en español que comunica que la incidencia no fue encontrada (AC-004 de US-003); redacción exacta en la implementación |

```json
{
  "statusCode": 404,
  "error": "Not Found",
  "message": "La incidencia no fue encontrada"
}
```

<a id="post-incidents-id-comments"></a>
### `POST /incidents/{id}/comments` — Agregar comentario

Registra un comentario ([MD-002](../models/MD-002-comentario.md)) en una incidencia existente; el sistema asigna `id`, `incidentId`, `author` (usuario de demostración) y `createdAt` ([US-004](../../../specs/changes/user-stories/US-004-comentar-incidencia/README.md), [FL-004](../flows/FL-004-registro-comentario.md)). No hay operaciones de edición ni de eliminación de comentarios.

**Request**

| Parámetro | Ubicación | Tipo | Requerido | Descripción |
| --------- | --------- | ---- | --------- | ----------- |
| id | path | string | Sí | `id` de la incidencia; mismo tratamiento que en [GET /incidents/{id}](#get-incidents-id): sin formato UUID en el esquema, un valor que no es UUID v4 se responde como `404` |
| content | body | string | Sí | Validaciones de `content` en [MD-002](../models/MD-002-comentario.md): se recorta (trim) y luego 1–500 |

El esquema del body es **estricto**: si el cliente envía `id`, `createdAt`, `author` (o cualquier otra clave no declarada, p. ej. `incidentId`), la petición se rechaza con `400` y no se persiste nada (AC-002 de US-004).

```json
{
  "content": "Revisando las credenciales en el servidor VPN."
}
```

**Responses**

| Código | Condición | Cuerpo |
| ------ | --------- | ------ |
| 201 | Comentario registrado y persistido | [MD-002](../models/MD-002-comentario.md) completo |
| 400 | Cuerpo ausente o no JSON; `content` ausente, vacío (también solo espacios) o de más de 500 caracteres tras el recorte; claves no declaradas, incluidos `id`, `createdAt` o `author`. No se persiste nada | Error de validación del proyecto (misma estructura que en [POST /incidents](#post-incidents)), con `instancePath` = `/content` o la clave sobrante y `message` propio en español |
| 404 | La incidencia no existe (incluido id no UUID v4). No se persiste nada | Error de recurso no encontrado (ver [GET /incidents/{id}](#get-incidents-id)) |
| 500 | Fallo inesperado al persistir | Error genérico del proyecto: `{ statusCode, error, message }` |

```json
{
  "id": "0b7e9c1a-3f2d-4c8e-a5b6-7d8e9f0a1b2c",
  "incidentId": "6f1c2b0e-8d3a-4f5e-9a71-2c4d5e6f7a8b",
  "content": "Revisando las credenciales en el servidor VPN.",
  "author": "Agente demo",
  "createdAt": "2026-10-12T15:02:44.310Z"
}
```

- **Orden de validación:** el body se valida contra el esquema Zod antes de ejecutar el handler; por eso una petición con body inválido sobre una incidencia inexistente responde `400`, y la existencia de la incidencia (`404`) solo se comprueba con un body válido.
- **Idempotencia:** no es idempotente; dos envíos producen dos comentarios. El doble envío se evita bloqueando el formulario (AC-009 de US-004).

<a id="patch-incidents-id-status"></a>
### `PATCH /incidents/{id}/status` — Cambiar estado

Aplica una transición de estado permitida ([DG-001](../diagrams/DG-001-estados-incidencia.md)) y devuelve la incidencia actualizada ([US-005](../../../specs/changes/user-stories/US-005-cambiar-estado-incidencia/README.md), [FL-005](../flows/FL-005-cambio-estado-incidencia.md)). Una transición no permitida no modifica ningún dato.

**Request**

| Parámetro | Ubicación | Tipo | Requerido | Descripción |
| --------- | --------- | ---- | --------- | ----------- |
| id | path | string | Sí | `id` de la incidencia; mismo tratamiento que en [GET /incidents/{id}](#get-incidents-id): un valor que no es UUID v4 se responde como `404` |
| status | body | enum `IncidentStatus` | Sí | Estado destino: `Open` \| `In Progress` \| `Resolved` ([MD-001](../models/MD-001-incidencia.md)) |

El esquema del body es **estricto**: cualquier clave distinta de `status` se rechaza con `400`.

```json
{
  "status": "In Progress"
}
```

**Responses**

| Código | Condición | Cuerpo |
| ------ | --------- | ------ |
| 200 | Transición permitida aplicada y persistida | [MD-001](../models/MD-001-incidencia.md) con el nuevo `status` (sin comentarios) |
| 400 | Cuerpo ausente o no JSON; `status` ausente o fuera de los valores permitidos; claves no declaradas. Nada se modifica | Error de validación del proyecto (misma estructura que en [POST /incidents](#post-incidents)), con `instancePath` = `/status` y `message` propio en español |
| 404 | La incidencia no existe (incluido id no UUID v4). Nada se modifica | Error de recurso no encontrado (ver [GET /incidents/{id}](#get-incidents-id)) |
| 409 | Transición no permitida desde el estado actual, incluido el mismo estado (BR-02 de US-005). Nada se modifica | Error genérico del proyecto con `error: "Conflict"` y `message` propio en español que indica que la transición no está permitida desde el estado actual (AC-003 de US-005) |
| 500 | Fallo inesperado al persistir | Error genérico del proyecto: `{ statusCode, error, message }` |

```json
{
  "id": "6f1c2b0e-8d3a-4f5e-9a71-2c4d5e6f7a8b",
  "title": "No puedo acceder a la VPN",
  "description": "Desde esta mañana la VPN rechaza mis credenciales con error de autenticación.",
  "category": "NETWORK_CONNECTIVITY",
  "priority": "HIGH",
  "status": "In Progress",
  "createdAt": "2026-10-12T14:32:05.120Z"
}
```

```json
{
  "statusCode": 409,
  "error": "Conflict",
  "message": "No se puede cambiar el estado de Open a Resolved"
}
```

> El texto de `message` del `409` ilustra el estilo (estado actual y destino, en español); su redacción exacta se fija en la implementación. Como en el `404`, la implementación debe asegurar que el manejador común emita `error: "Conflict"`.

- **Atomicidad:** la transición se aplica con una única actualización condicionada al estado actual (`status` destino solo si el estado vigente es el origen permitido). Si otra petición cambió el estado entre la lectura y la escritura, la actualización no afecta filas y se responde `409` sin cambios parciales ([persistence.md](../../../../helpdesk-backend/docs/standards/persistence.md), `consistencia-transaccional`).

## Notas

- **Autenticación y permisos:** Pública en todas las operaciones. La aplicación es de demostración y no implementa autenticación ni roles (ver Fuera de alcance de [US-001](../../../specs/changes/user-stories/US-001-registrar-incidencia/README.md) y [US-005](../../../specs/changes/user-stories/US-005-cambiar-estado-incidencia/README.md)); el plugin `src/plugins/auth.ts` está vacío.
- **Prefijo y versionado:** las rutas cuelgan de `/incidents`, sin prefijo de versión, como el resto del servicio (`/health`). El módulo Fastify se registra con `prefix: '/incidents'` y declara sus rutas relativas al prefijo (`/`, `/summary`, `/:id`, `/:id/comments`, `/:id/status`) (estándar de arquitectura del backend).
- **Errores:** validación → `400` con `{ statusCode, error, message, issues[] }` y mensajes propios en español en `issues[].message`; recurso inexistente → `404`; transición no permitida → `409`; fallo inesperado → `500`. Los `404`/`409` usan la estructura genérica `{ statusCode, error, message }`. Todos se declaran en el contrato OpenAPI de cada operación (CR-004 de [api.md](../../../../helpdesk-backend/docs/standards/api.md)).
- **Código de transición no permitida:** se adopta `409 Conflict` (el recurso existe y la petición es válida, pero entra en conflicto con su estado actual); los estándares y ADR del backend no fijan otro código.

## Historial de cambios

| Fecha | Cambio | Origen |
| ----- | ------ | ------ |
| 2026-10-12 | Añadida `GET /incidents` (filtros, orden, paginación de 20) | [US-002](../../../specs/changes/user-stories/US-002-listar-filtrar-incidencias/README.md) |
| 2026-10-12 | `GET /incidents/{id}` ampliada con `comments` y cerrado el `404` (incluido id no UUID v4, mismo cuerpo) | [US-003](../../../specs/changes/user-stories/US-003-detalle-incidencia/README.md) |
| 2026-10-12 | Añadida `POST /incidents/{id}/comments` | [US-004](../../../specs/changes/user-stories/US-004-comentar-incidencia/README.md) |
| 2026-10-12 | Añadida `PATCH /incidents/{id}/status` con `409` para transiciones no permitidas | [US-005](../../../specs/changes/user-stories/US-005-cambiar-estado-incidencia/README.md) |
| 2026-10-12 | Añadida `GET /incidents/summary` | [US-006](../../../specs/changes/user-stories/US-006-dashboard-indicadores/README.md) |
