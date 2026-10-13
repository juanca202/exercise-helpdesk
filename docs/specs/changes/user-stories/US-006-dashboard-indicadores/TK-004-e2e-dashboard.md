# TK-004: Pruebas E2E del dashboard

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-006](./README.md)
**Repositorio:** helpdesk-frontend

## Descripción

Cubrir con Playwright el dashboard desde el navegador: indicadores con datos y total coherente con la suma por estado, estado sin incidencias con su mensaje, error de la consulta con reintento, y navegación desde el menú y los accesos rápidos hacia el listado y el registro, de modo que los criterios de la historia queden verificados de extremo a extremo en el cliente.

## Dependencias

- Página de dashboard y menú de navegación de [TK-003](./TK-003-pagina-dashboard-navegacion.md) — pantalla bajo prueba.
- Formulario de registro en `/incidents/new` de [TK-005 de US-001](../US-001-registrar-incidencia/TK-005-formulario-registro-incidencia.md) — destino de «Nueva incidencia».
- Página de listado en `/incidents` de [TK-003 de US-002](../US-002-listar-filtrar-incidencias/TK-003-pagina-listado-incidencias.md) — destino de «Incidencias».
- `@playwright/test` y `playwright.config.ts` — runner y servidor de desarrollo.
- `page.route` / `fulfill` de Playwright — simulan `GET /incidents/summary` (y las lecturas de las páginas destino) conforme al contrato, sin requerir el backend ni MSW.

## Referencias

- **Arquitectura:** [ADR-005: Pruebas end-to-end con Playwright](../../../../../helpdesk-frontend/docs/adr/ADR-005-e2e-testing-playwright.md)
- **Estándares:** [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [API-001: GET /incidents/summary](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents-summary) · [MD-003: Indicadores de incidencias](../../../../architecture/incidencias/models/MD-003-indicadores-incidencias.md)
- **Diseño:** [WF-001: Dashboard](./wireframes/WF-001-dashboard.md) — Approved

## Archivos afectados

```text
helpdesk-frontend/
└── e2e/
    └── + dashboard.spec.ts                                  # escenarios E2E del dashboard
```

## Plan de implementación

- [ ] **IT-01** — Escenario con datos: abrir `/`, verificar que es el dashboard y que las cuatro tarjetas muestran los valores devueltos por la API simulada, con el total igual a la suma de abiertas, en atención y resueltas, y sin gráficos ni comparaciones
- [ ] **IT-02** — Escenario sin incidencias: la API simulada responde los cuatro valores en `0`; verificar las tarjetas en `0`, el mensaje «Aún no hay incidencias registradas» y su botón «Nueva incidencia»
- [ ] **IT-03** — Escenario de error con reintento: la primera consulta responde `500`; verificar la alerta con «Reintentar» y que no se muestran tarjetas; tras pulsar «Reintentar» con la API respondiendo `200`, verificar los indicadores
- [ ] **IT-04** — Escenario de carga: con la respuesta de la API retenida, verificar el texto «Cargando indicadores…» y que los accesos rápidos siguen disponibles
- [ ] **IT-05** — Escenario de navegación: desde el menú, «Incidencias» lleva a `/incidents`, «Nueva incidencia» a `/incidents/new` y «Dashboard» vuelve a `/`, con la sección activa marcada; los accesos rápidos llevan al listado y al registro
