---
id: TK-005
type: task
status: Ready
story: ./README.md
repository: helpdesk-frontend
---
# TK-005: Formulario de registro de incidencia

## Descripción

Implementar la pantalla de registro de incidencia según el wireframe aprobado: formulario con título, descripción, categoría y prioridad; validación por campo con contadores de caracteres; bloqueo y indicador de carga durante el envío; confirmación con el identificador al tener éxito; y mensaje de error con los datos conservados si el registro falla, mostrando en cada campo los errores de validación devueltos por la API.

## Dependencias

- `IncidentRepository`, modelo y mapper de [TK-004](./TK-004-repositorio-api-incidencias.md) — envío del registro.
- Signal Forms (`@angular/forms/signals`) — estado y validación del formulario.
- Angular Material y Tailwind CSS — controles y estilos del formulario.
- `app.routes.ts` — ruta con carga diferida de la feature `incidents`.

## Referencias

- **Arquitectura:** [ADR-001: Arquitectura por capas y features](../../../../../helpdesk-frontend/docs/adr/ADR-001-hybrid-layered-feature-architecture.md) · [ADR-006: Presentación con Tailwind CSS](../../../../../helpdesk-frontend/docs/adr/ADR-006-presentation-tailwind-css.md) · [ADR-003: Pruebas unitarias con Vitest y Testing Library](../../../../../helpdesk-frontend/docs/adr/ADR-003-unit-testing-vitest-testing-library.md)
- **Estándares:** [Frontend Standards](../../../../../helpdesk-frontend/docs/standards/frontend.md) · [Architecture Standards](../../../../../helpdesk-frontend/docs/standards/architecture.md)
- **Documentación técnica:** [FL-001: Registro de una incidencia](../../../../architecture/incidencias/flows/FL-001-registro-incidencia.md) · [API-001: POST /incidents](../../../../architecture/incidencias/apis/API-001-incidencias.md#post-incidents)
- **Diseño:** [WF-001: Registro de incidencia](./wireframes/WF-001-registro-incidencia.md) — Approved

## Archivos afectados

```text
helpdesk-frontend/
└── src/app/
    ├── ~ app.routes.ts                                      # ruta incidents/new con lazy loading
    └── features/incidents/
        ├── + incidents.routes.ts                            # rutas de la feature
        └── pages/incident-form/
            ├── + incident-form.ts                           # componente del formulario
            ├── + incident-form.html                         # plantilla con estados del wireframe
            ├── + incident-form.css                          # estilos
            └── + incident-form.spec.ts                      # unit con Testing Library y MSW
```

## Plan de implementación

- [x] **IT-01** — Registrar la ruta `incidents/new` con carga diferida y dejarla como pantalla de inicio de la aplicación
- [x] **IT-02** — Implementar el formulario con Signal Forms: título (recorte, 5–100), descripción (recorte, 10–2000), categoría y prioridad obligatorias, con las mismas reglas del contrato
- [x] **IT-03** — Mostrar el error de cada campo junto a él y los contadores `n / 100` y `n / 2000`; al enviar con errores mostrar todos y no enviar
- [x] **IT-04** — Enviar mediante `IncidentRepository`: indicador de carga y formulario bloqueado durante el envío para evitar duplicados
- [x] **IT-05** — Mostrar la confirmación con el identificador de la incidencia y la acción «Registrar otra incidencia» que limpia el formulario
- [x] **IT-06** — Ante `400`, asignar cada `issue` a su campo (`instancePath`); ante `500` o fallo de red, alerta sobre el formulario conservando los datos escritos
- [x] **IT-07** — Pruebas unitarias: validaciones y contadores, bloqueo durante el envío, éxito con identificador, errores de la API y accesibilidad (AXE)
