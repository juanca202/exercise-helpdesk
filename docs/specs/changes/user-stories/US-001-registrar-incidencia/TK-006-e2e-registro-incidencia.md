# TK-006: Pruebas E2E del registro de incidencia

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-001](./README.md)
**Repositorio:** helpdesk-frontend

## Descripción

Cubrir con Playwright el flujo crítico de registro de incidencia desde el navegador: registro válido con confirmación, validaciones visibles que impiden el envío y manejo de errores de la API, de modo que los criterios de la historia queden verificados de extremo a extremo en el cliente.

## Dependencias

- Formulario de [TK-005](./TK-005-formulario-registro-incidencia.md) — pantalla bajo prueba.
- `@playwright/test` y `playwright.config.ts` — runner y servidor de desarrollo.
- `page.route` / `fulfill` de Playwright — simulan la API conforme al contrato, sin requerir el backend ni MSW.

## Referencias

- **Arquitectura:** [ADR-005: Pruebas end-to-end con Playwright](../../../../../helpdesk-frontend/docs/adr/ADR-005-e2e-testing-playwright.md)
- **Estándares:** [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [API-001: POST /incidents](../../../../architecture/incidencias/apis/API-001-incidencias.md#post-incidents) · [FL-001: Registro de una incidencia](../../../../architecture/incidencias/flows/FL-001-registro-incidencia.md)
- **Diseño:** [WF-001: Registro de incidencia](./wireframes/WF-001-registro-incidencia.md) — Approved

## Archivos afectados

```text
helpdesk-frontend/
└── e2e/
    └── + incident-registration.spec.ts                      # escenarios E2E del registro
```

## Plan de implementación

- [ ] **IT-01** — Escenario de registro válido: completar el formulario, enviar y verificar la confirmación con el identificador devuelto por la API simulada
- [ ] **IT-02** — Escenario de validación: enviar vacío, verificar los errores por campo y que no se realiza ninguna petición
- [ ] **IT-03** — Escenario de error de validación de la API (`400` con `issues`): verificar el error en el campo correspondiente y los datos conservados
- [ ] **IT-04** — Escenario de fallo de la API (`500`): verificar la alerta general y los datos conservados para reintentar
