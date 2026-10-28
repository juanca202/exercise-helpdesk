---
id: TK-003
type: task
status: Ready
story: ./README.md
repository: helpdesk-frontend
---
# TK-003: Página de listado de incidencias

## Descripción

Implementar la página `/incidents` según el wireframe aprobado: tabla de incidencias (tarjetas en móvil) con identificador abreviado a 8 caracteres, título, categoría, prioridad, estado y fecha de creación, y acceso al detalle en `/incidents/:id`; filtros de estado, prioridad y categoría con las etiquetas de filtros aplicados y la acción «Limpiar filtros»; orden por fecha de creación invertible; y paginación de 20 con totales. Los filtros, el orden y la página viven en los parámetros de query de la dirección, de modo que recargar o volver atrás conserva la vista, y cambiar un filtro o el orden vuelve a la página 1. La página muestra los estados de carga, sin incidencias, sin coincidencias (con limpiar filtros) y error con reintento, y presenta por parámetro los errores de validación `400` devueltos por la API.

## Dependencias

- `IncidentRepository.findAll()`, modelos de consulta y página, opciones de estado y mapper del listado ([TK-002](./TK-002-repositorio-listado-incidencias.md)) — carga de la página y sincronización con la dirección.
- Opciones de categoría y prioridad con sus etiquetas en español ([TK-004 de US-001](../US-001-registrar-incidencia/TK-004-repositorio-api-incidencias.md)) — selectores de filtro y columnas de la tabla.
- `incidents.routes.ts` de la feature y ruta `/incidents/new` ([TK-005 de US-001](../US-001-registrar-incidencia/TK-005-formulario-registro-incidencia.md)) — se añade la ruta del listado; el botón «Nueva incidencia» navega al registro.
- Router de Angular con `withComponentInputBinding()` (ya activo en `app.config.ts`) — parámetros de query como `input()` de la página y navegación al cambiar filtros, orden o página.
- Angular Material, Tailwind CSS y `ft-icon` (`@factor_ec/ui`) — controles, estilos e iconos de la página.
- `DatePipe` de Angular — fecha de creación en formato `dd/mm/aaaa hh:mm`.

## Referencias

- **Arquitectura:** [ADR-001: Arquitectura por capas y features](../../../../../helpdesk-frontend/docs/adr/ADR-001-hybrid-layered-feature-architecture.md) · [ADR-010: Patrón Repository para REST](../../../../../helpdesk-frontend/docs/adr/ADR-010-repository-pattern-rest-api.md) · [ADR-006: Presentación con Tailwind CSS](../../../../../helpdesk-frontend/docs/adr/ADR-006-presentation-tailwind-css.md) · [ADR-013: Componente de iconos unificado](../../../../../helpdesk-frontend/docs/adr/ADR-013-unified-ft-icon-component.md) · [ADR-003: Pruebas unitarias con Vitest y Testing Library](../../../../../helpdesk-frontend/docs/adr/ADR-003-unit-testing-vitest-testing-library.md) · [ADR-007: Mocks HTTP con MSW](../../../../../helpdesk-frontend/docs/adr/ADR-007-http-api-mocks-msw.md)
- **Estándares:** [Frontend Standards](../../../../../helpdesk-frontend/docs/standards/frontend.md) · [Architecture Standards](../../../../../helpdesk-frontend/docs/standards/architecture.md) · [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [FL-002: Consulta del listado de incidencias](../../../../architecture/incidencias/flows/FL-002-consulta-listado-incidencias.md) · [API-001: GET /incidents](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents) · [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md)
- **Diseño:** [WF-001: Listado de incidencias](./wireframes/WF-001-listado-incidencias.md) — Approved · etiquetas de categoría y prioridad de [WF-001 de US-001](../US-001-registrar-incidencia/wireframes/WF-001-registro-incidencia.md)

## Archivos afectados

```text
helpdesk-frontend/
└── src/app/features/incidents/
    ├── ~ incidents.routes.ts                                # ruta del listado (/incidents) con carga diferida
    └── pages/incident-list/
        ├── + incident-list.ts                               # página: estado desde la URL, carga y navegación
        ├── + incident-list.html                             # filtros, etiquetas, tabla/tarjetas, paginación y estados
        ├── + incident-list.css                              # estilos
        └── + incident-list.spec.ts                          # unit con Testing Library, MSW y AXE
```

## Plan de implementación

- [x] **IT-01** — Registrar en `incidents.routes.ts` la ruta del listado con `loadComponent`, accesible en `/incidents`, junto a la ruta `/incidents/new` existente
- [x] **IT-02** — Leer `status`, `priority`, `category`, `order` y `page` de los parámetros de query como `input()` de la página, construir la consulta con el mapper de [TK-002](./TK-002-repositorio-listado-incidencias.md) y cargar la página con `findAll()` cada vez que cambian, con `notifyError: false`
  La dirección es la única fuente del estado de la vista: recargar, compartir la dirección o volver atrás desde el detalle restaura filtros, orden y página.
- [x] **IT-03** — Implementar la barra de filtros con tres selectores de un solo valor y su opción «Todos»/«Todas»: Estado (valores literales), Prioridad (Baja, Media, Alta) y Categoría (etiquetas en español)
  Cambiar un selector navega con el nuevo valor y `page` en 1; se conserva el orden.
- [x] **IT-04** — Mostrar una etiqueta por filtro aplicado («Prioridad: Alta ✕») cuya ✕ quita solo ese filtro, y el botón «Limpiar filtros», deshabilitado sin filtros, que quita todos y vuelve a la página 1 conservando el orden
- [x] **IT-05** — Implementar la tabla: ID (8 primeros caracteres), Título (enlace), Categoría, Prioridad, Estado (etiqueta con el valor literal) y Fecha de creación (`dd/mm/aaaa hh:mm`); toda la fila abre `/incidents/:id`
  El encabezado «Fecha de creación» es un botón que alterna el orden (↓ más recientes primero por defecto, ↑ ascendente), navega con `page` en 1 y expone el sentido con `aria-sort`.
- [x] **IT-06** — Implementar la paginación: «Mostrando a–b de N · Página x de y», «‹ Anterior», números de página y «Siguiente ›», con «Anterior» deshabilitado en la primera página y «Siguiente» en la última; el número de páginas se calcula con el total y el tamaño 20
- [x] **IT-07** — Mostrar el título «Incidencias», el número de resultados con los filtros actuales y el botón primario «Nueva incidencia» que navega a `/incidents/new`
- [x] **IT-08** — Implementar la vista móvil: tarjetas (título como enlace, id abreviado y fecha, etiquetas de categoría, prioridad y estado), filtros tras el botón «Filtros (n)», orden con «Orden: Recientes ↓» y paginación reducida «‹ Anterior · Página x de y · Siguiente ›»
- [x] **IT-09** — Implementar el estado de carga: filas de marcador y «Cargando incidencias…», filtros visibles y sin paginación
- [x] **IT-10** — Implementar los estados vacíos: sin filtros y `total` 0 → «Aún no hay incidencias registradas» con «Nueva incidencia»; con filtros y sin resultados → «Ninguna incidencia coincide con los filtros» con «Limpiar filtros»; página fuera de rango (`items` vacío con `total` > 0) → estado vacío con la paginación y los totales reales para volver a una página válida
- [x] **IT-11** — Implementar el estado de error: ante `500` o fallo de red, alerta «No se pudo cargar el listado de incidencias» con motivo comprensible y «Reintentar» (`reload()`), conservando los filtros; ante `400`, la alerta lista cada parámetro con su motivo en español devuelto por la API y ofrece «Limpiar filtros»
- [x] **IT-12** — Pruebas unitarias con Testing Library y MSW
  Columnas, id abreviado y enlace al detalle; filtros individuales y combinados reflejados en la petición y en la dirección con vuelta a la página 1; etiquetas y limpiar filtros; alternar el orden; paginación y botones deshabilitados en los extremos; estado restaurado desde los parámetros de query iniciales; estados de carga, sin incidencias, sin coincidencias, fuera de rango, error con reintento y `400` con los motivos por parámetro; verificación de accesibilidad AXE de los estados principales, como en [TK-005 de US-001](../US-001-registrar-incidencia/TK-005-formulario-registro-incidencia.md).
