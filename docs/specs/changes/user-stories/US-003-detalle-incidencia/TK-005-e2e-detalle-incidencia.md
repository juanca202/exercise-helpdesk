# TK-005: Pruebas E2E del detalle de incidencia

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-003](./README.md)
**Repositorio:** helpdesk-frontend

## Descripción

Cubrir con Playwright la consulta del detalle de incidencia desde el navegador: detalle con comentarios en orden cronológico, detalle sin comentarios, incidencia no encontrada con vuelta al listado, error de la API con reintento y apertura directa por URL, de modo que el comportamiento de la historia quede verificado de extremo a extremo en el cliente.

## Dependencias

- Página de detalle de [TK-004](./TK-004-pagina-detalle-incidencia.md) — pantalla bajo prueba en `/incidents/:id`.
- `@playwright/test` y `playwright.config.ts` — runner y servidor de desarrollo.
- `page.route` / `fulfill` de Playwright — simulan `GET /incidents/{id}` conforme al contrato, sin requerir el backend ni MSW.

## Referencias

- **Arquitectura:** [ADR-005: Pruebas end-to-end con Playwright](../../../../../helpdesk-frontend/docs/adr/ADR-005-e2e-testing-playwright.md)
- **Estándares:** [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [API-001: GET /incidents/{id}](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents-id) · [FL-003: Consulta del detalle de una incidencia](../../../../architecture/incidencias/flows/FL-003-consulta-detalle-incidencia.md)
- **Diseño:** [WF-001: Detalle de incidencia](./wireframes/WF-001-detalle-incidencia.md) — Approved

## Archivos afectados

```text
helpdesk-frontend/
└── e2e/
    └── + incident-detail.spec.ts                            # escenarios E2E del detalle
```

## Plan de implementación

- [x] **IT-01** — Escenario de detalle con comentarios: abrir `/incidents/{id}` y verificar título, id, descripción, estado, categoría, prioridad, fecha de creación y los comentarios en orden ascendente con autor, fecha y contenido
- [x] **IT-02** — Escenario sin comentarios: verificar «Comentarios (0)» y el mensaje «Aún no hay comentarios en esta incidencia»
- [x] **IT-03** — Escenario de incidencia no encontrada (`404`, con un UUID inexistente y con un id inválido): verificar «Incidencia no encontrada» y que «Volver al listado» navega a `/incidents`
- [x] **IT-04** — Escenario de error de la API (`500`): verificar la alerta «No se pudo cargar la incidencia» y que «Reintentar» muestra el detalle cuando la siguiente respuesta es `200`
- [x] **IT-05** — Escenario de apertura directa por URL: cargar `/incidents/{id}` en una página nueva, sin navegar desde otra pantalla, y verificar que se consulta ese id y se muestra su detalle
