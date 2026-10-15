# TK-004: Página de detalle de incidencia

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-003](./README.md)
**Repositorio:** helpdesk-frontend

## Descripción

Implementar la pantalla de detalle de incidencia en la ruta `/incidents/:id` según el wireframe aprobado: título e identificador, descripción, estado, categoría, prioridad y fecha de creación, y los comentarios en orden cronológico ascendente con autor, fecha y contenido. La pantalla muestra un indicador de carga mientras consulta, un estado vacío cuando no hay comentarios, un error comprensible con «Reintentar» si la consulta falla y un mensaje de «Incidencia no encontrada» con la acción de volver al listado (`/incidents`) ante un `404`. La disposición deja reservados el área del formulario de comentario y la barra de acciones de estado, que implementan [US-004](../US-004-comentar-incidencia/README.md) y [US-005](../US-005-cambiar-estado-incidencia/README.md).

## Dependencias

- Consulta de detalle de `IncidentRepository`, modelos `IncidentDetail` y `Comment` y mapper ([TK-003](./TK-003-repositorio-detalle-incidencia.md)) — datos de la pantalla y distinción entre «no encontrada» y fallo recuperable.
- Opciones de categoría y prioridad con etiqueta en español ([TK-004 de US-001](../US-001-registrar-incidencia/TK-004-repositorio-api-incidencias.md)) — etiquetas visibles.
- `incidents.routes.ts` y la carga diferida de la feature en `app.routes.ts` ([TK-005 de US-001](../US-001-registrar-incidencia/TK-005-formulario-registro-incidencia.md)) — se añade la ruta `:id`.
- Handlers MSW de `GET /incidents/:id` y Object Mother del detalle ([TK-003](./TK-003-repositorio-detalle-incidencia.md)) — pruebas unitarias.
- Angular Material, Tailwind CSS y `<ft-icon />` — controles, estilos e iconos.
- Testing Library y AXE — pruebas de componente y de accesibilidad.

## Referencias

- **Arquitectura:** [ADR-001: Arquitectura por capas y features](../../../../../helpdesk-frontend/docs/adr/ADR-001-hybrid-layered-feature-architecture.md) · [ADR-006: Presentación con Tailwind CSS](../../../../../helpdesk-frontend/docs/adr/ADR-006-presentation-tailwind-css.md) · [ADR-013: Componente de iconos unificado](../../../../../helpdesk-frontend/docs/adr/ADR-013-unified-ft-icon-component.md) · [ADR-003: Pruebas unitarias con Vitest y Testing Library](../../../../../helpdesk-frontend/docs/adr/ADR-003-unit-testing-vitest-testing-library.md) · [ADR-007: Mocks HTTP con MSW](../../../../../helpdesk-frontend/docs/adr/ADR-007-http-api-mocks-msw.md)
- **Estándares:** [Frontend Standards](../../../../../helpdesk-frontend/docs/standards/frontend.md) · [Architecture Standards](../../../../../helpdesk-frontend/docs/standards/architecture.md) · [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [FL-003: Consulta del detalle de una incidencia](../../../../architecture/incidencias/flows/FL-003-consulta-detalle-incidencia.md) · [API-001: GET /incidents/{id}](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents-id) · [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md) · [MD-002: Comentario](../../../../architecture/incidencias/models/MD-002-comentario.md)
- **Diseño:** [WF-001: Detalle de incidencia](./wireframes/WF-001-detalle-incidencia.md) — Approved · áreas reservadas: [WF-001 de US-004: Formulario de comentario](../US-004-comentar-incidencia/wireframes/WF-001-formulario-comentario.md) y [WF-001 de US-005: Acciones de estado](../US-005-cambiar-estado-incidencia/wireframes/WF-001-acciones-estado.md)

## Archivos afectados

```text
helpdesk-frontend/
└── src/app/features/incidents/
    ├── ~ incidents.routes.ts                                # ruta :id con carga diferida de la página de detalle
    └── pages/incident-detail/
        ├── + incident-detail.ts                             # componente: lee el id de la ruta y consulta el detalle
        ├── + incident-detail.html                           # plantilla con los estados del wireframe
        ├── + incident-detail.css                            # estilos con @apply
        └── + incident-detail.spec.ts                        # unit con Testing Library, MSW y AXE
```

## Plan de implementación

- [x] **IT-01** — Registrar en `incidents.routes.ts` la ruta `:id` (dirección `/incidents/:id`) con carga diferida del componente de detalle, sin interferir con `incidents/new`
- [x] **IT-02** — Leer el `id` de la ruta y consultar el detalle mediante `IncidentRepository`, volviendo a consultar si el `id` cambia
- [x] **IT-03** — Mostrar el estado de carga con bloques de marcador y el texto «Cargando incidencia…» mientras dura la consulta
- [x] **IT-04** — Mostrar el detalle según el wireframe: enlace «← Volver al listado» a `/incidents`, título e id completo, tarjeta «Descripción» que respeta saltos de línea, y panel «Datos de la incidencia» con estado (literal `Open` / `In Progress` / `Resolved`), categoría y prioridad con su etiqueta en español y fecha de creación con formato `dd/mm/aaaa hh:mm`
- [x] **IT-05** — Mostrar la tarjeta «Comentarios (n)» con cada comentario (avatar, autor, fecha y contenido) en el orden recibido, ascendente; con cero comentarios, «Comentarios (0)» y el mensaje «Aún no hay comentarios en esta incidencia»
- [x] **IT-06** — Reservar en la disposición el área «Acciones de estado» del panel lateral y el área del formulario de comentario debajo de la lista, sin implementar su contenido
- [x] **IT-07** — Ante un fallo recuperable (`500` o red), mostrar la alerta «No se pudo cargar la incidencia» con motivo comprensible y el botón «Reintentar», manteniendo visible «Volver al listado»; al reintentar, volver al estado de carga y consultar de nuevo
- [x] **IT-08** — Ante un `404` (id inexistente o inválido), mostrar «Incidencia no encontrada» con texto explicativo y el botón «Volver al listado» que navega a `/incidents`
- [x] **IT-09** — Adaptar la vista móvil a una sola columna: datos en cuadrícula de 2 × 2 con las acciones de estado debajo, descripción, comentarios y formulario
- [x] **IT-10** — Pruebas unitarias con Testing Library y MSW: carga, detalle con todos los campos y comentarios en orden, sin comentarios, error con reintento exitoso, no encontrada con navegación al listado, y accesibilidad (AXE) en los estados de detalle, error y no encontrada
