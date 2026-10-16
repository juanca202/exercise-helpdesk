# Cobertura de criterios — US-004: Agregar comentarios a una incidencia

**Veredicto:** ⚠️ Aprobado con observaciones
**Fecha:** 2026-10-15 21:06
**Trabajo:** [US-004](./README.md)
**Rama:** feature/US-004-comentar-incidencia (helpdesk-backend y helpdesk-frontend)

## Resumen

Los 10 criterios de aceptación tienen pruebas automatizadas que pasan. La US no tiene `test-cases/`, así que el mapeo se infirió desde las pruebas del repositorio.

**Pruebas:** resultados tomados de las corridas de `quality-check` del 2026-10-15 sobre ambos repositorios: helpdesk-backend 134 pruebas (`PASS`) y helpdesk-frontend 295 unitarias (`PASS`) más 31 e2e (`PASS`).

| Criterios de aceptación | Cubiertos | Parciales | No cubiertos |
| ----------------------- | --------- | --------- | ------------ |
| 10 | 10 | 0 | 0 |

## Cobertura por criterio

| Criterio | Descripción | Estado | Observaciones |
| -------- | ----------- | ------ | ------------- |
| AC-001 | Permitir agregar un comentario a una incidencia existente | ✅ Cubierto | Mapeo inferido, sin TC documentado |
| AC-002 | Asignar id, fecha y autor «Agente demo»; rechazar con 400 `id`, `createdAt` o `author` sin persistir | ✅ Cubierto | Mapeo inferido; también se rechaza `incidentId` |
| AC-003 | Rechazar contenido ausente, vacío (tras recortar) o de más de 500 caracteres | ✅ Cubierto | Mapeo inferido |
| AC-004 | 400 con campo y motivo en español, sin persistir | ✅ Cubierto | Mapeo inferido |
| AC-005 | 404 si la incidencia no existe, sin persistir | ✅ Cubierto | Mapeo inferido; incluye id no UUID |
| AC-006 | Comentarios conservados y mostrados en orden cronológico con contenido, autor y fecha | ✅ Cubierto | Mapeo inferido; el orden ascendente se verifica con las pruebas del repositorio de US-003 |
| AC-007 | Sin operaciones de edición o eliminación de comentarios | ✅ Cubierto | Mapeo inferido |
| AC-008 | Contador, error junto al campo sin enviar si es inválido, comentario añadido sin recargar y campo vacío | ✅ Cubierto | Mapeo inferido |
| AC-009 | Indicador de carga, bloqueo contra envíos duplicados y error comprensible conservando el texto | ✅ Cubierto | Mapeo inferido; sin pruebas AXE (axe-core no instalado) |
| AC-010 | Frontend consume el endpoint conforme al contrato y presenta los errores de validación de la API | ✅ Cubierto | Mapeo inferido |

## Matriz de trazabilidad

| Criterio | TC | Tipo | Evidencia | Ejecución | Resultado |
| -------- | -- | ---- | --------- | --------- | --------- |
| AC-001 | — | API Test | `helpdesk-backend/test/incidents.comments.test.ts` (alta válida) | quality-check | Paso |
| AC-001 | — | E2E | `helpdesk-frontend/e2e/incident-comments.spec.ts` (alta válida) | quality-check | Paso |
| AC-002 | — | API Test | `helpdesk-backend/test/incidents.comments.test.ts` (201 y claves `id`, `incidentId`, `author`, `createdAt`) | quality-check | Paso |
| AC-003 | — | API Test | `helpdesk-backend/test/incidents.comments.test.ts` (ausente, vacío, solo espacios, 501) | quality-check | Paso |
| AC-003 | — | Unit | `helpdesk-frontend/src/app/features/incidents/components/comment-form/comment-form.spec.ts` | quality-check | Paso |
| AC-004 | — | API Test | `helpdesk-backend/test/incidents.comments.test.ts` (`issues` con `instancePath` y mensaje; nada persistido) | quality-check | Paso |
| AC-004 | — | Unit | `.../comment-form.spec.ts` (issue `400` en el campo) | quality-check | Paso |
| AC-005 | — | API Test | `helpdesk-backend/test/incidents.comments.test.ts` (404 inexistente y no UUID) | quality-check | Paso |
| AC-006 | — | API Test | `helpdesk-backend/test/incidents.comments.test.ts` (aparece en GET), `comments.repository.test.ts` (orden) | quality-check | Paso |
| AC-006 | — | Unit | `helpdesk-frontend/src/app/features/incidents/pages/incident-detail/incident-detail.spec.ts` | quality-check | Paso |
| AC-007 | — | API Test | `helpdesk-backend/test/openapi.test.ts` (solo `post` sobre rutas de comentarios) | quality-check | Paso |
| AC-008 | — | Unit | `.../comment-form.spec.ts` (contador, error sin petición, campo vacío tras éxito), `.../incident-detail.spec.ts` | quality-check | Paso |
| AC-008 | — | E2E | `helpdesk-frontend/e2e/incident-comments.spec.ts` (validación sin petición y alta sin recargar) | quality-check | Paso |
| AC-009 | — | Unit | `.../comment-form.spec.ts` (bloqueo e indicador, 404/500/red con texto conservado) | quality-check | Paso |
| AC-009 | — | E2E | `helpdesk-frontend/e2e/incident-comments.spec.ts` (500 y reintento) | quality-check | Paso |
| AC-010 | — | Unit | `.../incident-repository.spec.ts` (`addComment`), `.../comment-mapper.spec.ts` | quality-check | Paso |
| AC-010 | — | API Test | `helpdesk-backend/test/openapi.test.ts` (ruta y respuestas en el contrato) | quality-check | Paso |

## Observaciones y pendientes

- La US no tiene casos de prueba documentados (`test-cases/`); toda la trazabilidad es inferida. Considerar `/test-define` si se quiere trazabilidad declarada.
- No hay pruebas de accesibilidad con AXE en el formulario: `axe-core` no está instalado en helpdesk-frontend; se verificó con aserciones ARIA.

<!-- coverage-verify:verdict=APPROVED_WITH_NOTES · fingerprint=ddb50f8894e12aa32b89e3d7ce852c0d00d27bd3 · spec=3c65db189a995518cbd171626665f34ab9e67489 · generated=2026-10-15 -->
