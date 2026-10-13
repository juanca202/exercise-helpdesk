# Progreso

## US-001-registrar-incidencia

<!-- work:id=US-001 · status=Done -->
**Estado:** Done
**Tipo:** historia de usuario
**Fecha de creación:** 2026-10-13 09:52
**Ultima actualizacion:** 2026-10-13 15:25

## Unidades

### TK-001: Persistencia SQLite de incidencias

<!-- unit:id=TK-001 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-13 10:05
**Finalizado:** 2026-10-13 10:40
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
~ helpdesk-backend/.env.example
~ helpdesk-backend/src/config/env.ts
~ helpdesk-backend/src/plugins/database.ts
~ helpdesk-backend/src/types/fastify.d.ts
+ helpdesk-backend/src/modules/incidents/incidents.repository.ts
~ helpdesk-backend/test/setup.ts
+ helpdesk-backend/test/env.test.ts
+ helpdesk-backend/test/database.test.ts
+ helpdesk-backend/test/incidents.repository.test.ts
+ helpdesk-backend/docs/adr/ADR-009-node-sqlite-acceso-a-datos.md
~ helpdesk-backend/docs/adr/ADR-007-sqlite-persistencia-local.md
~ helpdesk-backend/docs/adr/README.md
```

**Notas:**
- `npm run format:check` global falla por `AGENTS.md`, `docs/standards/api.md` y `docs/standards/persistence.md`, anteriores a esta TK; los archivos de la unidad están formateados.
- `id` se declara `TEXT NOT NULL PRIMARY KEY`: en SQLite un `TEXT PRIMARY KEY` admite NULL salvo que se declare `NOT NULL` (lo detectó la prueba del esquema).

**Decisiones adicionales:**
- IT-06: ADR-007 está `Accepted` y no se reescribe; la elección de `node:sqlite` se registró en el nuevo ADR-009 (sin `emits`) y ADR-007 lo referencia en su sección de referencias. Decisores: Equipo.
- El tipo `Incident` y sus enums viven en `incidents.repository.ts` hasta que TK-002 introduzca los esquemas Zod.

### TK-002: Endpoint de creación de incidencias

<!-- unit:id=TK-002 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-13 11:00
**Finalizado:** 2026-10-13 11:35
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
~ helpdesk-backend/src/app.ts
~ helpdesk-backend/src/plugins/openapi.ts
~ helpdesk-backend/src/plugins/error-handler.ts
+ helpdesk-backend/src/modules/incidents/incidents.schemas.ts
+ helpdesk-backend/src/modules/incidents/incidents.service.ts
+ helpdesk-backend/src/modules/incidents/incidents.routes.ts
+ helpdesk-backend/test/incidents.create.test.ts
~ helpdesk-backend/test/openapi.test.ts
```

**Notas:**
- IT-06: el manejador común ya devolvía `instancePath` y los mensajes propios en español en `issues`; solo se ajustó para que los errores 4xx que no son de validación (p. ej. un cuerpo que no es JSON) usen la frase HTTP estándar en `error` (`Bad Request`) en lugar del nombre de la excepción (`FastifyError`).
- Las respuestas `400` y `500` se declaran en el contrato OpenAPI de la ruta (CR-004 de api.md); `issues` es opcional en el esquema porque un cuerpo no JSON no genera `issues`.
- Para claves no declaradas, `issues[].instancePath` es `/` (raíz) y el mensaje lista los campos rechazados.

**Decisiones adicionales:**
- Los esquemas de error (`errorSchema`, `validationErrorSchema`) viven en `incidents.schemas.ts` por ahora; si otro módulo los necesita se subirán al kernel.

### TK-003: Endpoint de recuperación de incidencia por id

<!-- unit:id=TK-003 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-13 11:45
**Finalizado:** 2026-10-13 12:00
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
~ helpdesk-backend/src/modules/incidents/incidents.schemas.ts
~ helpdesk-backend/src/modules/incidents/incidents.service.ts
~ helpdesk-backend/src/modules/incidents/incidents.routes.ts
+ helpdesk-backend/test/incidents.get.test.ts
```

**Notas:**
- Un id que no es UUID responde `400` (validación del parámetro), tal como define esta TK; el tratamiento de ese caso como `404` lo cierra la historia de detalle (US-003 TK-002).
- El `404` usa la estructura común `{statusCode, error: "Not Found", message}` gracias al manejador de errores ajustado en TK-002.

**Decisiones adicionales:**
- La excepción de dominio `IncidentNotFoundError` (con `statusCode = 404`) vive en `incidents.service.ts`, sin importar tipos de Fastify.

### TK-004: Repositorio y modelo de incidencias en el cliente

<!-- unit:id=TK-004 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-13 12:20
**Finalizado:** 2026-10-13 12:55
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
+ helpdesk-frontend/src/app/features/incidents/models/incident.ts
+ helpdesk-frontend/src/app/features/incidents/utils/incident-mapper.ts
+ helpdesk-frontend/src/app/features/incidents/utils/incident-mapper.spec.ts
+ helpdesk-frontend/src/app/features/incidents/repositories/incident-repository.ts
+ helpdesk-frontend/src/app/features/incidents/repositories/incident-repository.spec.ts
~ helpdesk-frontend/src/mocks/handlers.ts
+ helpdesk-frontend/test/mothers/incident-mother.ts
```

**Notas:**
- `npm run arch` en verde (19 criterios PASS); ESLint y tsc (app y spec) sin errores.
- Los archivos nuevos se formatearon con Prettier según la configuración del repo (`trailingComma` por defecto); varios archivos existentes del repo no pasan `prettier --check`, anterior a esta TK.
- `ng test` requiere `git-version.json`, generado por `scripts/git-version.js` (ignorado por git).

**Decisiones adicionales:**
- El mapper vive en `features/incidents/utils/incident-mapper.ts` (objeto `IncidentMapper`), no en `mappers/` como indicaba el árbol de archivos de la TK, para cumplir CR-014 de Architecture Standards y ADR-012 (decisión confirmada por el usuario). La TK no se modificó.
- Object Mother de pruebas en `test/mothers/incident-mother.ts` (alias `@/test/*`), para reutilizarlo desde TK-005.
- `IncidentRepository` usa `@Service()` (ADR-015) y notifica `create` con el id creado vía `BaseRepository.notifyChange`.
- Las etiquetas de categoría y prioridad usan `$localize` y reutilizan `Option` de `@/shared/models/option`.

### TK-005: Formulario de registro de incidencia

<!-- unit:id=TK-005 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-13 13:10
**Finalizado:** 2026-10-13 14:05
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
~ helpdesk-frontend/src/app/app.routes.ts
+ helpdesk-frontend/src/app/features/incidents/incidents.routes.ts
+ helpdesk-frontend/src/app/features/incidents/pages/incident-form/incident-form.ts
+ helpdesk-frontend/src/app/features/incidents/pages/incident-form/incident-form.html
+ helpdesk-frontend/src/app/features/incidents/pages/incident-form/incident-form.spec.ts
```

**Notas:**
- 12 pruebas unitarias del formulario en verde; `npm run arch` en verde (19 criterios), ESLint, tsc y `ng build` sin errores. El build avisa que el bundle inicial supera el presupuesto de 500 kB en 15,6 kB (el formulario va en un chunk diferido; el aviso no depende de esta TK).
- Pruebas de accesibilidad: la TK pedía AXE, pero `axe-core` / `jest-axe` no están en `package.json`. Se cubrió con aserciones sobre `aria-invalid`, `aria-describedby`, `role="alert"` y etiquetas asociadas; añadir AXE exige incorporar una dependencia nueva (pendiente de decisión).
- Los selectores de Signal Forms escuchan el evento `input` (también en `select`), por lo que los tests lo disparan con `fireEvent.input`.

**Decisiones adicionales:**
- Controles nativos (`input`, `textarea`, `select`, radios) estilizados con Tailwind en lugar de componentes Angular Material: `@angular/material` solo llega como dependencia transitiva y Material + Signal Forms no está probado en el proyecto; se respeta Tailwind como framework de presentación (ADR-006).
- No se creó `incident-form.css`: no hay estilos propios, solo utilidades Tailwind (CR-003 de Frontend Standards).
- `'' → incidents/new` como pantalla de inicio (redirect) según la TK; US-006 la reemplazará por el dashboard.
- Errores de la API por campo: se muestran sobre el campo y se limpian al editarlo; un `400` sin `issues` reconocibles se trata como error general.

### TK-006: Pruebas E2E del registro de incidencia

<!-- unit:id=TK-006 · status=Done -->
**Estado:** Done
**Iniciado:** 2026-10-13 14:20
**Finalizado:** 2026-10-13 15:10
**Implementador:** juanca202 / Claude / claude-sonnet-5-5

**Archivos:**
```
+ helpdesk-frontend/e2e/incident-registration.spec.ts
~ helpdesk-frontend/angular.json
~ helpdesk-frontend/AGENTS.md
~ helpdesk-frontend/src/app/core/services/app-manager.ts
~ helpdesk-frontend/src/app/core/services/app-manager.spec.ts
~ helpdesk-frontend/src/app/core/services/session.ts
~ helpdesk-frontend/src/app/core/services/session.spec.ts
~ helpdesk-frontend/src/app/core/components/error/error.ts
~ helpdesk-frontend/src/app/core/components/error/error.html
~ helpdesk-frontend/src/app/core/components/error/error.spec.ts
~ helpdesk-frontend/src/app/shared/components/main-layout/main-layout.ts
~ helpdesk-frontend/src/app/shared/components/main-layout/main-layout.spec.ts
~ helpdesk-frontend/test/mocks/service-mocks.ts
```

**Notas:**
- Los 4 escenarios E2E pasan (4/4) contra `ng serve` en el puerto 4300. Los specs unitarios afectados (97 pruebas), ESLint de los archivos tocados y `npm run arch` (19 criterios) en verde.
- La aplicación base no arrancaba en el navegador por dos defectos anteriores a US-001, corregidos aquí porque impedían verificar la TK:
  1. `$localize is not defined`: se añadió `polyfills: ["@angular/localize/init"]` a `build.options` de `angular.json`.
  2. `NG0201: No provider found for AuthProvider`: por indicación del usuario, la aplicación no usa `AuthProvider`; se eliminó toda referencia (`AppManager`, `Session`, página `Error`, `MainLayout`, mocks de prueba y `AGENTS.md`).
- En el puerto 4200 había un `ng serve` de otra sesión que `playwright.config.ts` reutiliza (`reuseExistingServer`); la verificación se hizo en el puerto 4300 con una configuración temporal ya eliminada. Quien ejecute `npm run e2e` debe cerrar ese servidor o usar el suyo.
- Prettier señala formato anterior en `session.ts`, `app-manager.ts` y otros archivos existentes; no se reformatearon para mantener el diff acotado.

**Decisiones adicionales:**
- Sin autenticación (decisión del usuario): `Session.getSettings` devuelve los ajustes locales si existen y, si no, los pide a la red; la página de error muestra «Go home» salvo `unknown` («Reload»); `AppManager.init` ya no carga ajustes por usuario autenticado.
- Los escenarios simulan `POST /incidents` con `page.route`, sin backend ni MSW (ADR-005).
- `docs/audits/arch-audit-2026-09-12.md` (informe histórico) todavía menciona `AuthProvider`; no se editó.

