# Progreso

## US-004-comentar-incidencia

<!-- work:id=US-004 · status=In Progress -->
**Estado:** In Progress
**Tipo:** historia de usuario
**Fecha de creación:** 2026-10-15 20:36
**Ultima actualizacion:** 2026-10-15 20:48

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

<!-- unit:id=TK-002 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-15 20:37
**Finalizado:** 2026-10-15 20:39
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
~ helpdesk-frontend/src/app/features/incidents/models/comment.ts
+ helpdesk-frontend/src/app/features/incidents/utils/comment-mapper.ts
+ helpdesk-frontend/src/app/features/incidents/utils/comment-mapper.spec.ts
~ helpdesk-frontend/src/app/features/incidents/repositories/incident-repository.ts
~ helpdesk-frontend/src/app/features/incidents/repositories/incident-repository.spec.ts
~ helpdesk-frontend/src/mocks/handlers.ts
```

**Notas:**
- Pruebas de los archivos afectados (comment-mapper e incident-repository), eslint y tsc (app y spec) en verde. Código en el worktree `helpdesk-frontend/.worktrees/US-004-comentar-incidencia` (rama `feature/US-004-comentar-incidencia`), pendiente de commit.
- El worktree reutiliza `node_modules` del repo principal mediante enlace simbólico y requirió `node scripts/git-version.js` para generar `git-version.json`.

**Decisiones adicionales:**
- Además de `NewComment` se añadió `CommentSubmission` (`{ id, comment }`) como parámetro de `addComment`, siguiendo el patrón de `IncidentStatusChange`.
- Los handlers 404 y 500 se exportan como `commentNotFoundHandler` y `commentServerErrorHandler` (variantes para `server.use`), igual que los del resumen; el handler por defecto devuelve 201 o 400.

### TK-003: Formulario de comentario en el detalle de incidencia

<!-- unit:id=TK-003 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-15 20:47
**Finalizado:** 2026-10-15 20:48
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
+ helpdesk-frontend/src/app/features/incidents/components/comment-form/comment-form.ts
+ helpdesk-frontend/src/app/features/incidents/components/comment-form/comment-form.html
+ helpdesk-frontend/src/app/features/incidents/components/comment-form/comment-form.spec.ts
~ helpdesk-frontend/src/app/features/incidents/pages/incident-detail/incident-detail.ts
~ helpdesk-frontend/src/app/features/incidents/pages/incident-detail/incident-detail.html
~ helpdesk-frontend/src/app/features/incidents/pages/incident-detail/incident-detail.spec.ts
```

**Notas:**
- Pruebas de los archivos afectados (comment-form e incident-detail, 25), eslint y tsc (app y spec) en verde. Código en el worktree `helpdesk-frontend/.worktrees/US-004-comentar-incidencia`, pendiente de commit.
- IT-09: el proyecto no tiene ninguna librería AXE instalada (ni `axe-core` ni `vitest-axe`), así que la accesibilidad se verificó con aserciones sobre etiqueta, `aria-describedby`, `aria-invalid`, `role="alert"` y `aria-busy`; no se añadió dependencia.
- El `comment-form.spec.ts` y el componente se escribieron en la misma iteración, por lo que no se observó el rojo de ese archivo; las pruebas de la pantalla de detalle sí se vieron fallar antes de integrar.
- El estilo del formulario va con Tailwind en la plantilla (igual que `incident-form`), por lo que no se creó `comment-form.css` como preveía la lista de archivos del TK.

**Decisiones adicionales:**
- El destacado «Recién agregado» se retira solo a los 5 s (`RECENT_HIGHLIGHT_MS`) o al agregar otro comentario.

### TK-004: Pruebas E2E de comentarios en el detalle de incidencia

<!-- unit:id=TK-004 · status=In Progress -->
**Estado:** In Progress
**Iniciado:** 2026-10-15 20:48
**Implementador:** juanca202 / Claude / claude-sonnet-5-5
