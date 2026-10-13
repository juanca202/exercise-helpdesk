# TK-001: Persistencia SQLite de comentarios

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-003](./README.md)
**Repositorio:** helpdesk-backend

## Descripción

Disponer en el backend de la persistencia de los comentarios de una incidencia: tabla `comments` creada por el plugin `database` junto a la de incidencias, tipo `Comment` conforme al modelo de comentario y un repositorio de acceso a datos que recupera los comentarios de una incidencia en orden cronológico ascendente, con desempate estable por orden de inserción, e inserta un comentario. Es infraestructura compartida: la lectura la consume el detalle de esta historia y la inserción queda disponible y probada para el registro de comentarios de [US-004](../US-004-comentar-incidencia/README.md).

## Dependencias

- `src/plugins/database.ts` ([TK-001 de US-001](../US-001-registrar-incidencia/TK-001-persistencia-sqlite-incidencias.md)) — conexión `DatabaseSync` decorada como `db`; aquí se amplía con la tabla `comments`.
- Tabla `incidents` ([TK-001 de US-001](../US-001-registrar-incidencia/TK-001-persistencia-sqlite-incidencias.md)) — destino de la clave foránea `incident_id`.
- `node:sqlite` (`DatabaseSync`) — sentencias preparadas de lectura e inserción.
- Tipo `Comment` — entidad del modelo de comentario, usada por el repositorio y, después, por los esquemas del detalle y del alta de comentarios.

## Referencias

- **Arquitectura:** [ADR-007: Persistencia local con SQLite](../../../../../helpdesk-backend/docs/adr/ADR-007-sqlite-persistencia-local.md) · [ADR-005: Monolito modular](../../../../../helpdesk-backend/docs/adr/ADR-005-monolito-modular-adaptador-http.md) · [ADR-004: Vitest e inject de Fastify](../../../../../helpdesk-backend/docs/adr/ADR-004-vitest-fastify-inject.md)
- **Estándares:** [Persistence Standards](../../../../../helpdesk-backend/docs/standards/persistence.md) · [Architecture Standards](../../../../../helpdesk-backend/docs/standards/architecture.md) · [Testing Standards](../../../../../helpdesk-backend/docs/standards/testing.md)
- **Documentación técnica:** [MD-002: Comentario](../../../../architecture/incidencias/models/MD-002-comentario.md) · [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md) · [FL-003: Consulta del detalle de una incidencia](../../../../architecture/incidencias/flows/FL-003-consulta-detalle-incidencia.md)

## Archivos afectados

```text
helpdesk-backend/
├── src/
│   ├── ~ plugins/database.ts                        # tabla comments, índice por incident_id y claves foráneas activas
│   └── modules/incidents/
│       └── + comments.repository.ts                 # tipo Comment, findByIncidentId e insert
└── test/
    ├── ~ database.test.ts                           # el esquema incluye comments y su índice
    └── + comments.repository.test.ts                # orden, desempate, lista vacía, inserción y clave foránea
```

## Plan de implementación

- [x] **IT-01** — Crear en el plugin `database` la tabla `comments` si no existe, después de `incidents`
  Columnas: `id` (TEXT, clave primaria), `incident_id` (TEXT, clave foránea a `incidents(id)`), `content`, `author`, `created_at` (TEXT ISO 8601 UTC, mismo formato que `incidents.created_at`); `NOT NULL` en todas. Sin borrado en cascada: no existe borrado de incidencias ni de comentarios.
- [x] **IT-02** — Crear el índice por `incident_id` y activar la verificación de claves foráneas de SQLite al abrir la conexión
- [x] **IT-03** — Declarar el tipo `Comment` (`id`, `incidentId`, `content`, `author`, `createdAt`) conforme a MD-002, sin importar tipos de Fastify
- [x] **IT-04** — Implementar `findByIncidentId(incidentId)` en `comments.repository.ts`: comentarios de la incidencia ordenados por `created_at` ascendente y, ante igualdad, por orden de inserción (`rowid` ascendente); lista vacía si no hay
  Mapear `incident_id` ↔ `incidentId` y `created_at` ↔ `createdAt`.
- [x] **IT-05** — Implementar `insert(comment)` en `comments.repository.ts` como una única sentencia atómica que persiste el comentario recibido con todos sus campos ya asignados
  La asignación de `id`, `author` y `createdAt` y la validación del contenido corresponden al servicio de [US-004](../US-004-comentar-incidencia/README.md); el repositorio solo persiste.
- [x] **IT-06** — Pruebas: el plugin crea la tabla `comments` y su índice; insertar y recuperar devuelve los mismos datos; una incidencia sin comentarios devuelve lista vacía; los comentarios salen en orden ascendente de `createdAt`; dos comentarios con el mismo `createdAt` salen en orden de inserción; insertar un comentario con un `incidentId` inexistente falla por la clave foránea y no persiste nada; los comentarios de otra incidencia no se mezclan
