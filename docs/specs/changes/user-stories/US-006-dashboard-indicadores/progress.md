# Progreso

## US-006-dashboard-indicadores

<!-- work:id=US-006 · status=Done -->
**Estado:** Done
**Tipo:** historia de usuario
**Fecha de creación:** 2026-10-13 16:05
**Ultima actualizacion:** 2026-10-13 16:32

## Unidades

### TK-001: Endpoint de indicadores de incidencias

<!-- unit:id=TK-001 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-13 16:08
**Finalizado:** 2026-10-13 16:15
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
~ helpdesk-backend/src/modules/incidents/incidents.repository.ts
~ helpdesk-backend/src/modules/incidents/incidents.schemas.ts
~ helpdesk-backend/src/modules/incidents/incidents.service.ts
~ helpdesk-backend/src/modules/incidents/incidents.routes.ts
+ helpdesk-backend/test/incidents.summary.test.ts
~ helpdesk-backend/test/incidents.repository.test.ts
~ helpdesk-backend/test/openapi.test.ts
```

**Notas:**
- Pruebas unitarias/integración de los archivos afectados, lint, typecheck, build y formato en verde. El worktree reutiliza `node_modules` del repo principal mediante enlace simbólico. La US no tiene test cases (`test-cases/`); las pruebas se derivaron de los AC-XXX.
- Rama `feature/US-006-dashboard-indicadores` creada desde `main` de cada submódulo (sin los cambios de US-002/US-003, aún sin integrar).

### TK-002: Lectura de indicadores en el repositorio de incidencias

<!-- unit:id=TK-002 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-13 16:10
**Finalizado:** 2026-10-13 16:13
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
+ helpdesk-frontend/src/app/features/incidents/models/incident-summary.ts
~ helpdesk-frontend/src/app/features/incidents/repositories/incident-repository.ts
~ helpdesk-frontend/src/app/features/incidents/repositories/incident-repository.spec.ts
~ helpdesk-frontend/src/mocks/handlers.ts
```

**Notas:**
- `getSummary` se expone como recurso (`getResource`): `getSummary.load()`, `.value`, `.loading`, `.error`, `.reload`. Las variantes vacía y `500` se exportan desde `src/mocks/handlers.ts` (`incidentSummaryEmptyHandler`, `incidentSummaryErrorHandler`) para reutilizarlas en TK-003.
- Spec del repositorio (10 pruebas), eslint y tsc en verde. Al correr un solo spec, el umbral global de cobertura de ramas de `ng test` no se alcanza; es artefacto de la ejecución acotada.
- Para `ng test` en el worktree hay que generar `git-version.json` con `node scripts/git-version.js`.

### TK-003: Página de dashboard y menú de navegación

<!-- unit:id=TK-003 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-13 16:15
**Finalizado:** 2026-10-13 16:25
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
~ helpdesk-frontend/src/app/app.routes.ts
~ helpdesk-frontend/src/app/shared/models/menu-item.ts
~ helpdesk-frontend/src/app/shared/components/main-layout/main-layout.ts
~ helpdesk-frontend/src/app/shared/components/main-layout/main-layout.html
~ helpdesk-frontend/src/app/shared/components/main-layout/main-layout.spec.ts
+ helpdesk-frontend/src/app/features/incidents/pages/incident-dashboard/incident-dashboard.ts
+ helpdesk-frontend/src/app/features/incidents/pages/incident-dashboard/incident-dashboard.html
+ helpdesk-frontend/src/app/features/incidents/pages/incident-dashboard/incident-dashboard.css
+ helpdesk-frontend/src/app/features/incidents/pages/incident-dashboard/incident-dashboard.spec.ts
```

**Notas:**
- Specs del dashboard y de `MainLayout` (25 pruebas), eslint, tsc, `ng build` y `npm run arch` en verde. `ng build` advierte que el bundle inicial excede el presupuesto de 500 kB en 38,58 kB.
- IT-11 (AXE): `axe-core` no está instalado en el repositorio, por lo que las pruebas de accesibilidad con AXE no se escribieron en la unidad; el marcado usa `role="status"`, `role="alert"` y `aria-label`. Pendiente decidir si se añade la dependencia.
- IT-13: sin cambios; `e2e/incident-registration.spec.ts` ya navega a `/incidents/new`.
- Decisión: `MenuItem` gana `exact?: boolean` para que «Incidencias» no se marque activa en `/incidents/new`; el menú pasó a cabecera superior (con ☰ en móvil) y el pie muestra «Empleado demo» sin enlace a `/settings`.

### TK-004: Pruebas E2E del dashboard

<!-- unit:id=TK-004 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-13 16:26
**Finalizado:** 2026-10-13 16:32
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
+ helpdesk-frontend/e2e/dashboard.spec.ts
```

**Notas:**
- 5 escenarios E2E (con datos, sin incidencias, error con reintento, carga, navegación) y los 4 de registro de US-001 en verde, con un servidor `ng serve` propio en el puerto 4310: el puerto 4200 estaba ocupado por otra aplicación ajena (`control-plane`) y `playwright.config.ts` reutiliza el servidor existente.
