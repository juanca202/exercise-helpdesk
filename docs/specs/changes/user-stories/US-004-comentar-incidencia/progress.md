# Progreso

## US-004-comentar-incidencia

<!-- work:id=US-004 · status=In Progress -->
**Estado:** In Progress
**Tipo:** historia de usuario
**Fecha de creación:** 2026-10-15 20:36
**Ultima actualizacion:** 2026-10-15 20:37

## Unidades

### TK-001: Endpoint para agregar comentarios a una incidencia

<!-- unit:id=TK-001 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-15 20:36
**Finalizado:** 2026-10-15 20:37
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
~ helpdesk-backend/src/modules/incidents/incidents.schemas.ts
~ helpdesk-backend/src/modules/incidents/incidents.service.ts
~ helpdesk-backend/src/modules/incidents/incidents.routes.ts
+ helpdesk-backend/test/incidents.comments.test.ts
~ helpdesk-backend/test/openapi.test.ts
```

**Notas:**
- Pruebas de los archivos afectados (comments, openapi, get, status), lint, typecheck y prettier en verde. Código en el worktree `helpdesk-backend/.worktrees/US-004-comentar-incidencia` (rama `feature/US-004-comentar-incidencia`), pendiente de commit.
- La US no tiene `test-cases/`: las pruebas se derivaron de los AC-001 a AC-005 y AC-007.

### TK-002: Repositorio y mapper de comentarios en el cliente

<!-- unit:id=TK-002 · status=Pending -->
**Estado:** Pending

### TK-003: Formulario de comentario en el detalle de incidencia

<!-- unit:id=TK-003 · status=Pending -->
**Estado:** Pending

### TK-004: Pruebas E2E de comentarios en el detalle de incidencia

<!-- unit:id=TK-004 · status=Pending -->
**Estado:** Pending
