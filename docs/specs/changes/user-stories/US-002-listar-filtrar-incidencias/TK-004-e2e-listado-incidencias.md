# TK-004: Pruebas E2E del listado de incidencias

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-002](./README.md)
**Repositorio:** helpdesk-frontend

## Descripción

Cubrir con Playwright el flujo de consulta del listado de incidencias desde el navegador: visualización del listado con acceso al detalle, filtros combinados, inversión del orden, paginación conservada en la dirección al recargar y volver atrás, y los estados sin incidencias, sin coincidencias y error con reintento, de modo que los criterios de la historia queden verificados de extremo a extremo en el cliente.

## Dependencias

- Página de listado de [TK-003](./TK-003-pagina-listado-incidencias.md) — pantalla bajo prueba.
- `@playwright/test` y `playwright.config.ts` — runner y servidor de desarrollo.
- `page.route` / `fulfill` de Playwright — simulan `GET /incidents` conforme al contrato, sin requerir el backend ni MSW, como en [TK-006 de US-001](../US-001-registrar-incidencia/TK-006-e2e-registro-incidencia.md).

## Referencias

- **Arquitectura:** [ADR-005: Pruebas end-to-end con Playwright](../../../../../helpdesk-frontend/docs/adr/ADR-005-e2e-testing-playwright.md)
- **Estándares:** [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [API-001: GET /incidents](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents) · [FL-002: Consulta del listado de incidencias](../../../../architecture/incidencias/flows/FL-002-consulta-listado-incidencias.md)
- **Diseño:** [WF-001: Listado de incidencias](./wireframes/WF-001-listado-incidencias.md) — Approved

## Archivos afectados

```text
helpdesk-frontend/
└── e2e/
    └── + incident-list.spec.ts                              # escenarios E2E del listado
```

## Plan de implementación

- [ ] **IT-01** — Escenario de listado: abrir `/incidents` con una página simulada y verificar las columnas, el id abreviado a 8 caracteres, los totales de paginación y que la fila enlaza a `/incidents/:id`
- [ ] **IT-02** — Escenario de filtros combinados: elegir estado y prioridad y verificar que la petición lleva ambos parámetros, que la dirección los refleja con `page` en 1 y que se muestran sus etiquetas; quitar una etiqueta y limpiar todos los filtros
- [ ] **IT-03** — Escenario de orden: pulsar el encabezado «Fecha de creación» y verificar `order=asc` en la petición y en la dirección, y el orden ascendente de las filas mostradas
- [ ] **IT-04** — Escenario de paginación: pulsar «Siguiente ›» y verificar `page=2` en la petición y en la dirección; recargar y volver atrás conservan filtros, orden y página
- [ ] **IT-05** — Escenarios de estados vacíos: sin incidencias registradas (`total` 0 sin filtros) y sin coincidencias con filtros, este con «Limpiar filtros» que restablece el listado
- [ ] **IT-06** — Escenarios de error: `500` muestra la alerta con «Reintentar» y, al reintentar con respuesta correcta, el listado; una dirección con un filtro inválido recibe `400` con `issues` y la alerta muestra el parámetro y su motivo en español
