# Progreso

## US-002-listar-filtrar-incidencias

<!-- work:id=US-002 · status=Done -->
**Estado:** Done
**Tipo:** historia de usuario
**Fecha de creación:** 2026-10-13 16:10
**Ultima actualizacion:** 2026-10-13 18:50

## Unidades

### TK-001: Endpoint de listado de incidencias

<!-- unit:id=TK-001 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-13 16:15
**Finalizado:** 2026-10-13 16:45
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
~ helpdesk-backend/src/modules/incidents/incidents.schemas.ts
~ helpdesk-backend/src/modules/incidents/incidents.repository.ts
~ helpdesk-backend/src/modules/incidents/incidents.service.ts
~ helpdesk-backend/src/modules/incidents/incidents.routes.ts
~ helpdesk-backend/test/incidents.repository.test.ts
+ helpdesk-backend/test/incidents.list.test.ts
~ helpdesk-backend/test/openapi.test.ts
```

**Notas:**
- Las pruebas de `incidents.repository.test.ts` y `incidents.list.test.ts` vacían la tabla `incidents` antes de sembrar datos, porque comparten la base en memoria del archivo.
- Verificado sobre los archivos afectados: `tsc`, ESLint, Prettier, `test:arch` y 63 pruebas de los módulos de incidencias en verde.

### TK-002: Repositorio del listado de incidencias en el cliente

<!-- unit:id=TK-002 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-13 17:00
**Finalizado:** 2026-10-13 17:40
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
~ helpdesk-frontend/src/app/features/incidents/models/incident.ts
~ helpdesk-frontend/src/app/features/incidents/utils/incident-mapper.ts
~ helpdesk-frontend/src/app/features/incidents/utils/incident-mapper.spec.ts
~ helpdesk-frontend/src/app/features/incidents/repositories/incident-repository.ts
~ helpdesk-frontend/src/app/features/incidents/repositories/incident-repository.spec.ts
~ helpdesk-frontend/src/mocks/handlers.ts
~ helpdesk-frontend/test/mothers/incident-mother.ts
```

**Notas:**
- Verificado sobre los archivos afectados: 45 pruebas de la feature `incidents` en verde, ESLint, Prettier y `tsc` (app y spec) sin errores.
- `findAll()` devuelve un recurso nuevo por llamada; la página y el tamaño no se exponen en el recurso (solo `value` y `total`), la página actual la conoce quien consulta.

**Decisiones adicionales:**
- El mapper del listado se añadió a `utils/incident-mapper.ts` (ubicación existente de la feature) en lugar de la carpeta `mappers/` que indicaba el plan.
- Se ampliaron las Object Mother (`test/mothers/incident-mother.ts`) con `listQuery()` e `incidentPage()` para los specs.

### TK-003: Página de listado de incidencias

<!-- unit:id=TK-003 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-13 17:50
**Finalizado:** 2026-10-13 18:15
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
~ helpdesk-frontend/src/app/features/incidents/incidents.routes.ts
+ helpdesk-frontend/src/app/features/incidents/pages/incident-list/incident-list.ts
+ helpdesk-frontend/src/app/features/incidents/pages/incident-list/incident-list.html
+ helpdesk-frontend/src/app/features/incidents/pages/incident-list/incident-list.css
+ helpdesk-frontend/src/app/features/incidents/pages/incident-list/incident-list.spec.ts
```

**Notas:**
- Implementada por el agente `ui-specialist`; verificado sobre los archivos afectados: 67 pruebas de la feature `incidents`, ESLint, Prettier y `tsc` sin errores.
- IT-12: no hay AXE real (`axe-core` / `vitest-axe` no están en `package.json` y `incident-form.spec.ts` tampoco lo ejecuta); se sustituyó por una prueba de accesibilidad estructural (un `h1`, landmark `main`, navegación de paginación, selectores con `label`, botones con nombre accesible).
- IT-08: la vista móvil se implementó, pero jsdom no aplica Tailwind; solo se prueba que las tarjetas se renderizan con las mismas incidencias.
- `getResourceCollection` no cancela peticiones anteriores: una respuesta lenta previa podría sobrescribir `value` (fuera del alcance de la TK).

**Decisiones adicionales:**
- Selectores nativos `<select>` con Tailwind y glifos de texto en lugar de Angular Material y `ft-icon`, igual que `incident-form`.
- Una sola paginación y un solo juego de filtros para escritorio y móvil, con visibilidad por clases Tailwind; en móvil el orden se muestra como «Orden: Recientes ↓» / «Orden: Antiguas ↑».
- Página fuera de rango: «No hay incidencias en esta página» con la paginación real; con más de 7 páginas se abrevia con «…».
- Ante un `400`, la alerta lista los parámetros con nombres en español y ofrece «Limpiar filtros» sin «Reintentar».

### TK-004: Pruebas E2E del listado de incidencias

<!-- unit:id=TK-004 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-13 18:25
**Finalizado:** 2026-10-13 18:50
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
+ helpdesk-frontend/e2e/incident-list.spec.ts
```

**Notas:**
- 7 escenarios Playwright en verde (listado, filtros combinados, orden, paginación con recarga y volver atrás, vacíos, 500 con reintento y 400 con motivo). Ejecutados en el cierre de la unidad al ser E2E de la TK, sobre el servidor de desarrollo del worktree.
- El puerto 4200 estaba ocupado por otra aplicación ajena («SDD Platform»), por lo que `playwright.config.ts` (que reutiliza un servidor existente) habría probado esa app. Se ejecutó con una configuración temporal fuera del repositorio que levanta `ng serve` en el puerto 4310; `playwright.config.ts` no se modificó.
- El mock intercepta solo peticiones `fetch`/`xhr` a `/incidents`, para no capturar la navegación a la página `/incidents`.
