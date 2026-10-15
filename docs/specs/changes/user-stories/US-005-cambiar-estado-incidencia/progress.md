# Progreso

## US-005-cambiar-estado-incidencia

<!-- work:id=US-005 · status=In Progress -->
**Estado:** In Progress
**Tipo:** historia de usuario
**Fecha de creación:** 2026-10-15 12:30
**Ultima actualizacion:** 2026-10-15 13:02

## Unidades

### TK-001: Endpoint de cambio de estado de una incidencia

<!-- unit:id=TK-001 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-15 12:44
**Finalizado:** 2026-10-15 12:47
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
~ helpdesk-backend/src/modules/incidents/incidents.repository.ts
~ helpdesk-backend/src/modules/incidents/incidents.schemas.ts
~ helpdesk-backend/src/modules/incidents/incidents.service.ts
~ helpdesk-backend/src/modules/incidents/incidents.routes.ts
+ helpdesk-backend/test/incidents.status.test.ts
~ helpdesk-backend/test/incidents.repository.test.ts
~ helpdesk-backend/test/error-handler.test.ts
~ helpdesk-backend/test/openapi.test.ts
```

**Notas:**
- Pruebas de los archivos afectados (84 en `incidents.*`, más error-handler y openapi), lint, typecheck, build, formato y `test:arch` en verde. La US no tiene test cases (`test-cases/`); las pruebas se derivaron de los AC-XXX.
- IT-07: `src/plugins/error-handler.ts` ya devolvía `error: "Conflict"` para un `409` (frase HTTP del código); no requirió cambios y quedó cubierto con una prueba en `error-handler.test.ts`.
- IT-11: la prueba de actualización concurrente está en `incidents.status.test.ts` (simula el cambio entre lectura y escritura).
- Antes de arrancar se integraron US-002 y US-003 en `main` de ambos submódulos, resolviendo los conflictos con US-006; la rama `feature/US-005-cambiar-estado-incidencia` parte de ese `main`. Efecto colateral: `openapi.test.ts` dejó de esperar `400` en `GET /incidents/{id}` (US-003 resuelve un id mal formado como `404`).

### TK-002: Mutación de cambio de estado en el repositorio de incidencias

<!-- unit:id=TK-002 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-15 12:49
**Finalizado:** 2026-10-15 12:55
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
~ helpdesk-frontend/src/app/features/incidents/models/incident.ts
~ helpdesk-frontend/src/app/features/incidents/utils/incident-mapper.ts
~ helpdesk-frontend/src/app/features/incidents/utils/incident-mapper.spec.ts
~ helpdesk-frontend/src/app/features/incidents/repositories/incident-repository.ts
~ helpdesk-frontend/src/app/features/incidents/repositories/incident-repository.spec.ts
~ helpdesk-frontend/src/mocks/handlers.ts
```

**Notas:**
- Specs del mapper y del repositorio (59 pruebas), eslint, tsc, prettier y `npm run arch` en verde.
- El mapper vive en `utils/incident-mapper.ts` (no en `mappers/` como indica el árbol del TK). `IncidentStatus` ya existía en el modelo; se añadieron `IncidentStatusChange`, `IncidentStatusChangeFailure` e `IncidentStatusChangeError`.
- La respuesta `200` se convierte con `mapResponseToIncident` existente; el nuevo mapper aporta `mapStatusToRequest` y `mapErrorToStatusChangeError`. Los handlers `400`/`404`/`409`/`500` no se exportan: las pruebas los sobrescriben con `server.use(...)`.

### TK-003: Acciones de estado en el detalle de incidencia

<!-- unit:id=TK-003 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-15 12:56
**Finalizado:** 2026-10-15 13:02
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
+ helpdesk-frontend/src/app/features/incidents/models/incident-status-transition.ts
+ helpdesk-frontend/src/app/features/incidents/components/incident-status-actions/incident-status-actions.ts
+ helpdesk-frontend/src/app/features/incidents/components/incident-status-actions/incident-status-actions.html
+ helpdesk-frontend/src/app/features/incidents/components/incident-status-actions/incident-status-actions.spec.ts
~ helpdesk-frontend/src/app/features/incidents/pages/incident-detail/incident-detail.ts
~ helpdesk-frontend/src/app/features/incidents/pages/incident-detail/incident-detail.html
~ helpdesk-frontend/src/app/features/incidents/pages/incident-detail/incident-detail.spec.ts
```

**Notas:**
- Specs del componente y de la página de detalle (66 pruebas de `components/` y `pages/`), eslint, tsc, prettier, `ng build` y `npm run arch` en verde.
- IT-08/IT-09 (AXE): `axe-core` no está instalado en el repositorio, así que no se escribieron pruebas con AXE; el marcado usa `aria-busy`, `role="alert"` y nombres accesibles. Pendiente decidir si se añade la dependencia (mismo pendiente que US-006 TK-003).
- No se creó `incident-status-actions.css`: los estilos van con Tailwind en la plantilla. El aviso de éxito se publica con `notify` desde el componente.
- Se actualizó la prueba de la página que comprobaba el área reservada `status-actions-area`; ahora verifica el encabezado «Acciones de estado».
- «Recargar incidencia» vuelve a pedir el detalle (`detail.reload()`) sin pasar por la pantalla de carga, para conservar la sección visible.

### TK-004: Pruebas E2E del cambio de estado de una incidencia

<!-- unit:id=TK-004 · status=Pending -->
**Estado:** Pending

## Decisiones adicionales

- Base de la rama: tras integrar US-002 y US-003 en `main`, a petición del usuario («Integrar todos antes»). Las puertas de cierre (quality-check, code-review, coverage-verify) quedaron omitidas en esas dos integraciones por decisión del usuario.
