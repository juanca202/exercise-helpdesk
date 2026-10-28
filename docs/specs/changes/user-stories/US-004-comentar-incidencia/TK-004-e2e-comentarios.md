---
id: TK-004
type: task
status: Ready
story: ./README.md
repository: helpdesk-frontend
---
# TK-004: Pruebas E2E de comentarios en el detalle de incidencia

## Descripción

Cubrir con Playwright el flujo de agregar un comentario desde la pantalla de detalle (`/incidents/:id`): alta válida con el comentario visible al final de la lista y el campo vacío, validaciones visibles que impiden el envío, y manejo de los errores `400` y `500` de la API conservando el texto escrito, de modo que el comportamiento de la historia quede verificado de extremo a extremo en el cliente.

## Dependencias

- Formulario de comentario de [TK-003](./TK-003-formulario-comentario.md) — componente bajo prueba.
- Pantalla de detalle de incidencia ([TK-004 de US-003](../US-003-detalle-incidencia/TK-004-pagina-detalle-incidencia.md)) — ruta `/incidents/:id` que aloja el formulario.
- `@playwright/test` y `playwright.config.ts` — runner y servidor de desarrollo.
- `page.route` / `fulfill` de Playwright — simulan `GET /incidents/{id}` y `POST /incidents/{id}/comments` conforme al contrato, sin requerir el backend ni MSW.

## Referencias

- **Arquitectura:** [ADR-005: Pruebas end-to-end con Playwright](../../../../../helpdesk-frontend/docs/adr/ADR-005-e2e-testing-playwright.md)
- **Estándares:** [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [API-001: POST /incidents/{id}/comments](../../../../architecture/incidencias/apis/API-001-incidencias.md#post-incidents-id-comments) · [API-001: GET /incidents/{id}](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents-id) · [FL-004: Registro de un comentario](../../../../architecture/incidencias/flows/FL-004-registro-comentario.md)
- **Diseño:** [WF-001: Formulario de comentario](./wireframes/WF-001-formulario-comentario.md) — Approved

## Archivos afectados

```text
helpdesk-frontend/
└── e2e/
    └── + incident-comments.spec.ts                          # escenarios E2E de comentarios
```

## Plan de implementación

- [x] **IT-01** — Preparar la simulación con `page.route`: `GET /incidents/{id}` con una incidencia y un comentario previo, y `POST /incidents/{id}/comments` configurable por escenario
- [x] **IT-02** — Escenario de alta válida: escribir el comentario, enviar y verificar que aparece al final de la lista con autor «Agente demo» y fecha, que el contador de la tarjeta aumenta y que el campo queda vacío, sin recargar la página
  Verificar además que el cuerpo enviado contiene solo `content`.
- [x] **IT-03** — Escenario de validación: enviar vacío (y solo con espacios) y con más de 500 caracteres, verificar el error junto al campo y que no se realiza ninguna petición `POST`
- [x] **IT-04** — Escenario de error de validación de la API (`400` con `issues` en `/content`): verificar el error en el campo y el texto conservado
- [x] **IT-05** — Escenario de fallo de la API (`500`): verificar el aviso «No se pudo agregar el comentario», el texto conservado y el formulario habilitado para reintentar
