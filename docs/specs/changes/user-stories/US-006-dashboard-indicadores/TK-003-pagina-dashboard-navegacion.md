---
id: TK-003
type: task
status: Ready
story: ./README.md
repository: helpdesk-frontend
---
# TK-003: Página de dashboard y menú de navegación

## Descripción

Implementar el dashboard como página principal de la aplicación en la ruta `/`, según el wireframe aprobado: cuatro indicadores solo informativos (Total, Abiertas, En atención, Resueltas) obtenidos al abrir la página, sin gráficos analíticos ni comparaciones históricas; mensaje explicativo cuando todos los valores son cero; indicador de carga y mensaje de error con reintento; y accesos rápidos al listado y al registro. Incorporar el layout principal con el menú de navegación común (Dashboard · Incidencias · Nueva incidencia) que enlaza a `/`, `/incidents` y `/incidents/new`, manteniendo `/incidents/new` como ruta del registro.

## Dependencias

- `IncidentRepository.getSummary` y modelo `IncidentSummary` de [TK-002](./TK-002-repositorio-indicadores.md) — lectura de los indicadores.
- Rutas de la feature `incidents` (`incidents.routes.ts`) y ruta `incidents/new` con carga diferida de [TK-005 de US-001](../US-001-registrar-incidencia/TK-005-formulario-registro-incidencia.md) — se conservan y pasan a colgar del layout principal.
- `MainLayout` (`src/app/shared/components/main-layout/`) y modelo `MenuItem` (`src/app/shared/models/menu-item.ts`) — contenedor de las rutas y menú de navegación.
- `ft-icon` de `@factor_ec/ui` — iconos del menú y de los accesos rápidos.
- Angular Material y Tailwind CSS — tarjetas, alerta, botones y estilos responsivos.
- `@testing-library/angular`, `msw` y AXE — pruebas unitarias y de accesibilidad.

## Referencias

- **Arquitectura:** [ADR-001: Arquitectura por capas y features](../../../../../helpdesk-frontend/docs/adr/ADR-001-hybrid-layered-feature-architecture.md) · [ADR-006: Presentación con Tailwind CSS](../../../../../helpdesk-frontend/docs/adr/ADR-006-presentation-tailwind-css.md) · [ADR-013: Componente de iconos unificado](../../../../../helpdesk-frontend/docs/adr/ADR-013-unified-ft-icon-component.md) · [ADR-003: Pruebas unitarias con Vitest y Testing Library](../../../../../helpdesk-frontend/docs/adr/ADR-003-unit-testing-vitest-testing-library.md) · [ADR-007: Mocks HTTP con MSW](../../../../../helpdesk-frontend/docs/adr/ADR-007-http-api-mocks-msw.md)
- **Estándares:** [Frontend Standards](../../../../../helpdesk-frontend/docs/standards/frontend.md) · [Architecture Standards](../../../../../helpdesk-frontend/docs/standards/architecture.md) · [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [API-001: GET /incidents/summary](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents-summary) · [MD-003: Indicadores de incidencias](../../../../architecture/incidencias/models/MD-003-indicadores-incidencias.md)
- **Diseño:** [WF-001: Dashboard](./wireframes/WF-001-dashboard.md) — Approved

## Archivos afectados

```text
helpdesk-frontend/
├── e2e/
│   └── ~ incident-registration.spec.ts                      # solo si navega a / esperando el formulario: pasa a /incidents/new
└── src/app/
    ├── ~ app.routes.ts                                      # MainLayout como contenedor; / = dashboard (lazy); /incidents/new se mantiene
    ├── shared/components/main-layout/
    │   ├── ~ main-layout.ts                                 # opciones de navegación Dashboard · Incidencias · Nueva incidencia
    │   ├── ~ main-layout.html                               # menú con sección activa y usuario de la demo
    │   └── ~ main-layout.spec.ts                            # enlaces del menú y opción activa
    └── features/incidents/pages/incident-dashboard/
        ├── + incident-dashboard.ts                          # página del dashboard
        ├── + incident-dashboard.html                        # indicadores, accesos rápidos y estados del wireframe
        ├── + incident-dashboard.css                         # estilos
        └── + incident-dashboard.spec.ts                     # unit con Testing Library, MSW y AXE
```

## Plan de implementación

- [x] **IT-01** — Reorganizar `app.routes.ts`: `MainLayout` como contenedor de las rutas de la aplicación, `/` carga el dashboard con carga diferida (`loadComponent`) y título «Dashboard», y `incidents` conserva las rutas de la feature con `incidents/new` como registro
  Se elimina la redirección de la raíz a `incidents/new` que dejó TK-005 de US-001; las rutas de error y la ruta comodín `**` se mantienen al final.
- [x] **IT-02** — Completar en `MainLayout` las opciones de navegación «Dashboard» (`/`), «Incidencias» (`/incidents`) y «Nueva incidencia» (`/incidents/new`) con su icono `ft-icon`, marcando la sección activa
  «Dashboard» solo se marca activa con coincidencia exacta de `/`; «Incidencias» no se marca activa en `/incidents/new`.
- [x] **IT-03** — Ajustar el pie del menú al wireframe: mostrar el usuario de la demo («Empleado demo») y retirar el acceso a `/settings`, que no tiene ruta
- [x] **IT-04** — Implementar la página `incident-dashboard` en `features/incidents/pages/` que solicita los indicadores con `getSummary` al abrirse, sin notificación global de error duplicada
- [x] **IT-05** — Mostrar el título «Dashboard», el texto que indica que los datos se calculan al abrir la página y las cuatro tarjetas de igual tamaño: «Total» (destacada), «Abiertas» (`Open`), «En atención» (`In Progress`) y «Resueltas» (`Resolved`), con el texto «Total = Abiertas + En atención + Resueltas»
  Mapeo `total`, `open`, `inProgress`, `resolved` ↔ Total, Abiertas, En atención, Resueltas. Las tarjetas son solo informativas: no son enlaces ni botones. Sin gráficos, series temporales, comparaciones ni botón «Actualizar».
- [x] **IT-06** — Mostrar los accesos rápidos «Ver listado de incidencias» (`/incidents`) y «Nueva incidencia» (`/incidents/new`), disponibles en todos los estados de la página
- [x] **IT-07** — Estado de carga: cada tarjeta muestra su título y un bloque gris en lugar del número, con el texto «Cargando indicadores…» anunciado a tecnologías de asistencia
- [x] **IT-08** — Estado de error: alerta «No se pudieron cargar los indicadores» con un motivo comprensible y el botón «Reintentar», que vuelve a consultar; las tarjetas no se muestran
- [x] **IT-09** — Estado sin incidencias: las cuatro tarjetas en `0` y el mensaje «Aún no hay incidencias registradas» con el botón «Nueva incidencia» hacia `/incidents/new`
- [x] **IT-10** — Vista responsiva según el wireframe móvil: tarjetas en cuadrícula de 2 × 2, accesos rápidos apilados a ancho completo y menú tras el icono ☰
- [x] **IT-11** — Pruebas unitarias de la página con MSW: valores y suma coherente, estado de carga, estado sin incidencias con su mensaje, error con reintento exitoso, enlaces de los accesos rápidos, ausencia de enlaces en las tarjetas y accesibilidad (AXE) en cada estado
- [x] **IT-12** — Pruebas unitarias de `MainLayout`: las tres opciones enlazan a `/`, `/incidents` y `/incidents/new` y se marca activa la sección actual
- [x] **IT-13** — Revisar el E2E de US-001 (`e2e/incident-registration.spec.ts`, de [TK-006 de US-001](../US-001-registrar-incidencia/TK-006-e2e-registro-incidencia.md)): si navega a `/` esperando el formulario de registro, cambiar esa navegación a `/incidents/new`; no modificar sus escenarios ni sus aserciones
