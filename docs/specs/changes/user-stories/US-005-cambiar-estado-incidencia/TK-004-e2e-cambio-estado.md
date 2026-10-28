---
id: TK-004
type: task
status: Ready
story: ./README.md
repository: helpdesk-frontend
---
# TK-004: Pruebas E2E del cambio de estado de una incidencia

## Descripción

Cubrir con Playwright el flujo crítico de cambio de estado desde la pantalla de detalle: el ciclo completo `Open → In Progress → Resolved → In Progress` con solo la acción válida visible en cada estado, el indicador y el bloqueo mientras se aplica, el fallo de la API conservando el estado anterior y el rechazo `409` con el mensaje de la API y la opción de recargar la incidencia.

## Dependencias

- Acciones de estado de [TK-003](./TK-003-acciones-estado-detalle.md) — sección bajo prueba.
- Página de detalle en `/incidents/:id` de [TK-004 de US-003](../US-003-detalle-incidencia/TK-004-pagina-detalle-incidencia.md) — pantalla que aloja las acciones.
- `@playwright/test` y `playwright.config.ts` — runner y servidor de desarrollo.
- `page.route` / `fulfill` de Playwright — simulan `GET /incidents/{id}` y `PATCH /incidents/{id}/status` conforme al contrato, sin requerir el backend ni MSW.

## Referencias

- **Arquitectura:** [ADR-005: Pruebas end-to-end con Playwright](../../../../../helpdesk-frontend/docs/adr/ADR-005-e2e-testing-playwright.md)
- **Estándares:** [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [API-001: PATCH /incidents/{id}/status](../../../../architecture/incidencias/apis/API-001-incidencias.md#patch-incidents-id-status) · [API-001: GET /incidents/{id}](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents-id) · [FL-005: Cambio de estado de una incidencia](../../../../architecture/incidencias/flows/FL-005-cambio-estado-incidencia.md) · [DG-001: Estados de la incidencia](../../../../architecture/incidencias/diagrams/DG-001-estados-incidencia.md)
- **Diseño:** [WF-001: Acciones de estado](./wireframes/WF-001-acciones-estado.md) — Approved

## Archivos afectados

```text
helpdesk-frontend/
└── e2e/
    └── + incident-status.spec.ts                            # escenarios E2E del cambio de estado
```

## Plan de implementación

- [x] **IT-01** — Escenario del ciclo completo: abrir el detalle de una incidencia en `Open`, verificar que solo se ofrece «Iniciar atención», aplicarla y verificar `In Progress`, el aviso de éxito y que solo se ofrece «Resolver»; aplicarla y verificar `Resolved` con solo «Reabrir»; reabrir y verificar `In Progress`
  En cada paso, comprobar que la petición `PATCH` lleva el cuerpo `{ status }` del destino y que no aparece diálogo de confirmación.
- [x] **IT-02** — Escenario de aplicación en curso: con la respuesta `PATCH` retenida, verificar el indicador «Aplicando…», la acción deshabilitada y el estado mostrado sin cambios hasta la respuesta
- [x] **IT-03** — Escenario de fallo de la API (`500`): verificar la alerta «No se pudo cambiar el estado» con un mensaje comprensible, el estado anterior conservado y la acción habilitada de nuevo
- [x] **IT-04** — Escenario de transición rechazada (`409`): verificar la alerta con el mensaje en español devuelto por la API y el estado anterior conservado; pulsar «Recargar incidencia» y verificar el estado vigente devuelto por `GET /incidents/{id}` con su acción válida
