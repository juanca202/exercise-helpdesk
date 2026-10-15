# Progreso

## US-005-cambiar-estado-incidencia

<!-- work:id=US-005 · status=In Progress -->
**Estado:** In Progress
**Tipo:** historia de usuario
**Fecha de creación:** 2026-10-15 12:30
**Ultima actualizacion:** 2026-10-15 12:47

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

<!-- unit:id=TK-002 · status=Pending -->
**Estado:** Pending

### TK-003: Acciones de estado en el detalle de incidencia

<!-- unit:id=TK-003 · status=Pending -->
**Estado:** Pending

### TK-004: Pruebas E2E del cambio de estado de una incidencia

<!-- unit:id=TK-004 · status=Pending -->
**Estado:** Pending

## Decisiones adicionales

- Base de la rama: tras integrar US-002 y US-003 en `main`, a petición del usuario («Integrar todos antes»). Las puertas de cierre (quality-check, code-review, coverage-verify) quedaron omitidas en esas dos integraciones por decisión del usuario.
