---
id: TK-001
type: task
status: Ready
story: ./README.md
repository: helpdesk-backend
---
# TK-001: Persistencia SQLite de incidencias

## Descripción

Disponer en el backend de la persistencia local de incidencias con SQLite: conexión gestionada por el plugin `database`, esquema de la tabla de incidencias creado al arrancar y un repositorio de acceso a datos que inserta y recupera una incidencia por su identificador. Es la base que habilita registrar y recuperar incidencias (AC-002, AC-003).

## Dependencias

- `node:sqlite` (`DatabaseSync`, módulo integrado de Node ≥ 22) — motor y acceso a datos, sin dependencia nativa.
- `src/plugins/database.ts` — punto de enganche del kernel; se decora en la instancia raíz (`db`).
- `src/config/env.ts` — variable `DATABASE_PATH` (ruta del archivo SQLite; `:memory:` en pruebas).
- Tipo `Incident` — entidad definida en MD-001, usada por el repositorio.

## Referencias

- **Arquitectura:** [ADR-007: Persistencia local con SQLite](../../../../../helpdesk-backend/docs/adr/ADR-007-sqlite-persistencia-local.md) · [ADR-005: Monolito modular](../../../../../helpdesk-backend/docs/adr/ADR-005-monolito-modular-adaptador-http.md) · [ADR-006: Plugins encapsulados por feature](../../../../../helpdesk-backend/docs/adr/ADR-006-plugins-encapsulados-por-feature.md)
- **Estándares:** [Persistence Standards](../../../../../helpdesk-backend/docs/standards/persistence.md) · [Architecture Standards](../../../../../helpdesk-backend/docs/standards/architecture.md)
- **Documentación técnica:** [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md)

## Archivos afectados

```text
helpdesk-backend/
├── ~ .env.example                                   # DATABASE_PATH
├── src/
│   ├── ~ config/env.ts                              # variable DATABASE_PATH validada con Zod
│   ├── ~ plugins/database.ts                        # abre SQLite, crea esquema, decora `db`, cierra en onClose
│   ├── ~ types/fastify.d.ts                         # tipo del decorador `db`
│   └── modules/incidents/
│       └── + incidents.repository.ts                # insert y findById sobre la tabla incidents
└── test/
    ├── + database.test.ts                           # plugin: esquema y cierre
    └── + incidents.repository.test.ts               # round-trip de una incidencia
```

## Plan de implementación

- [x] **IT-01** — Añadir `DATABASE_PATH` al esquema de entorno y a `.env.example` (por defecto un archivo local; `:memory:` en test)
- [x] **IT-02** — Implementar el plugin `database`: abrir `DatabaseSync`, crear la tabla `incidents` si no existe y decorar la instancia con `db`
  Columnas: `id` (TEXT, clave primaria), `title`, `description`, `category`, `priority`, `status`, `created_at` (TEXT ISO 8601 UTC), con restricciones `CHECK` para los enums y `NOT NULL` en todas.
- [x] **IT-03** — Cerrar la conexión en el hook `onClose` y tipar el decorador en `src/types/fastify.d.ts`
- [x] **IT-04** — Implementar `incidents.repository.ts` con `insert` y `findById`, mapeando `created_at` ↔ `createdAt`; sin importar tipos de Fastify
- [x] **IT-05** — Pruebas: el plugin crea el esquema; insertar y recuperar devuelve los mismos datos; un id inexistente devuelve `undefined`
- [x] **IT-06** — Registrar la elección de `node:sqlite` como librería de acceso en ADR-007 mediante `/arch-manage`
