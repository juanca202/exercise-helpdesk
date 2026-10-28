---
id: TK-003
type: task
status: Ready
story: ./README.md
repository: helpdesk-frontend
---
# TK-003: Acciones de estado en el detalle de incidencia

## Descripción

Implementar la sección «Acciones de estado» del panel «Datos de la incidencia» de la pantalla de detalle según el wireframe aprobado: ofrecer solo la acción válida desde el estado actual («Iniciar atención» en `Open`, «Resolver» en `In Progress`, «Reabrir» en `Resolved`) con el estado destino indicado debajo, y aplicarla sin confirmación. Mientras se aplica, la acción muestra un indicador de carga y queda bloqueada; al éxito, el detalle muestra el nuevo estado con su acción y aparece un aviso temporal «Estado actualizado a …»; ante un error, una alerta «No se pudo cambiar el estado» con un mensaje comprensible (el de la API si la transición fue rechazada) conserva el estado anterior y ofrece «Recargar incidencia».

## Dependencias

- Mutación `changeStatus`, `IncidentStatus` y tipo de error de cambio de estado de [TK-002](./TK-002-repositorio-cambio-estado.md) — aplicación de la transición y clasificación del error.
- Página de detalle de [TK-004 de US-003](../US-003-detalle-incidencia/TK-004-pagina-detalle-incidencia.md) — panel «Datos de la incidencia» y barra de acciones donde se integra la sección.
- Lectura del detalle de [TK-003 de US-003](../US-003-detalle-incidencia/TK-003-repositorio-detalle-incidencia.md) — recarga de la incidencia tras un error.
- `notify` de `src/app/core/utils/notification.ts` — aviso temporal de éxito con `level: 'success'`.
- Angular Material y Tailwind CSS — botón, indicador de carga y alerta.

## Referencias

- **Arquitectura:** [ADR-009: Puente de notificaciones por eventos](../../../../../helpdesk-frontend/docs/adr/ADR-009-core-event-notification-bridge.md) · [ADR-010: Patrón Repository para REST](../../../../../helpdesk-frontend/docs/adr/ADR-010-repository-pattern-rest-api.md) · [ADR-001: Arquitectura por capas y features](../../../../../helpdesk-frontend/docs/adr/ADR-001-hybrid-layered-feature-architecture.md) · [ADR-006: Presentación con Tailwind CSS](../../../../../helpdesk-frontend/docs/adr/ADR-006-presentation-tailwind-css.md) · [ADR-003: Pruebas unitarias con Vitest y Testing Library](../../../../../helpdesk-frontend/docs/adr/ADR-003-unit-testing-vitest-testing-library.md)
- **Estándares:** [Architecture Standards](../../../../../helpdesk-frontend/docs/standards/architecture.md) (`core-event-notification-bridge`) · [Frontend Standards](../../../../../helpdesk-frontend/docs/standards/frontend.md) · [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [FL-005: Cambio de estado de una incidencia](../../../../architecture/incidencias/flows/FL-005-cambio-estado-incidencia.md) · [DG-001: Estados de la incidencia](../../../../architecture/incidencias/diagrams/DG-001-estados-incidencia.md) · [API-001: PATCH /incidents/{id}/status](../../../../architecture/incidencias/apis/API-001-incidencias.md#patch-incidents-id-status)
- **Diseño:** [WF-001: Acciones de estado](./wireframes/WF-001-acciones-estado.md) — Approved · [WF-001 de US-003: Detalle de incidencia](../US-003-detalle-incidencia/wireframes/WF-001-detalle-incidencia.md) — ubicación de la sección y vista móvil

## Archivos afectados

```text
helpdesk-frontend/
└── src/app/features/incidents/
    ├── models/
    │   └── + incident-status-transition.ts                  # transición válida por estado: acción y destino
    ├── components/incident-status-actions/
    │   ├── + incident-status-actions.ts                     # componente de la sección «Acciones de estado»
    │   ├── + incident-status-actions.html                   # acción, aplicando, alerta de error y recarga
    │   ├── + incident-status-actions.css                    # estilos
    │   └── + incident-status-actions.spec.ts                # unit con Testing Library, MSW y AXE
    └── pages/incident-detail/
        ├── ~ incident-detail.ts                             # aplica el nuevo estado y recarga la incidencia
        └── ~ incident-detail.html                           # integra la sección en el panel de datos
```

## Plan de implementación

- [x] **IT-01** — Definir la transición válida por estado según DG-001: `Open` → «Iniciar atención» / `In Progress`; `In Progress` → «Resolver» / `Resolved`; `Resolved` → «Reabrir» / `In Progress`
- [x] **IT-02** — Implementar el componente de la sección: recibe la incidencia (id y estado actual) y muestra solo el botón primario de la transición válida, con el estado destino indicado debajo; sin diálogo de confirmación
- [x] **IT-03** — Al pulsar, invocar `changeStatus` con `notifyError: false`; mientras se aplica, mostrar el indicador y el texto «Aplicando…», deshabilitar la acción y mantener el estado mostrado
- [x] **IT-04** — Al éxito, emitir la incidencia actualizada para que el detalle muestre el nuevo estado y su acción, conservando los comentarios ya cargados, y publicar con `notify` el aviso «Estado actualizado a {estado}» con `level: 'success'`
- [x] **IT-05** — Ante un error, mostrar la alerta «No se pudo cambiar el estado» con el mensaje del error de cambio de estado (el de la API tal cual si el motivo es `rejected`), conservar el estado anterior y volver a habilitar la acción
- [x] **IT-06** — Incluir en la alerta el enlace «Recargar incidencia», que vuelve a pedir el detalle, retira la alerta y muestra el estado vigente con su acción válida
- [x] **IT-07** — Integrar la sección en el panel «Datos de la incidencia» de la página de detalle; en móvil, botón a ancho completo bajo los datos
- [x] **IT-08** — Accesibilidad: acción con nombre accesible, estado ocupado (`aria-busy`) mientras se aplica y alerta anunciada (`role="alert"`)
- [x] **IT-09** — Pruebas unitarias con Testing Library y MSW: acción visible por cada estado y ausencia de las demás; aplicación sin confirmación con el cuerpo esperado; indicador y bloqueo durante la petición; nuevo estado y aviso de éxito; `409` con el mensaje de la API, estado conservado y recarga; `500` y error de red con mensaje comprensible y estado conservado; accesibilidad (AXE)
