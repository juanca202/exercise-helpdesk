# Cobertura de criterios — US-005: Cambiar el estado de una incidencia

**Veredicto:** ⚠️ Aprobado con observaciones
**Fecha:** 2026-10-15 14:00
**Trabajo:** [US-005](./README.md)
**Rama:** feature/US-005-cambiar-estado-incidencia (helpdesk-backend y helpdesk-frontend)

## Resumen

Los 9 criterios de aceptación tienen pruebas automatizadas que pasan. La US no tiene `test-cases/`, así que el mapeo se infirió desde las pruebas del repositorio.

**Pruebas:** resultados tomados de las corridas de `quality-check` del 2026-10-15 sobre ambos repositorios: helpdesk-backend 117 pruebas (`PASS`) y helpdesk-frontend 272 unitarias (`PASS`) más 27 e2e (`PASS`).

| Criterios de aceptación | Cubiertos | Parciales | No cubiertos |
| ----------------------- | --------- | --------- | ------------ |
| 9 | 9 | 0 | 0 |

## Cobertura por criterio

| Criterio | Descripción | Estado | Observaciones |
| -------- | ----------- | ------ | ------------- |
| AC-001 | Permitir actualizar el estado de una incidencia existente | ✅ Cubierto | Mapeo inferido, sin TC documentado |
| AC-002 | Aplicar solo las transiciones Open→In Progress, In Progress→Resolved, Resolved→In Progress | ✅ Cubierto | Mapeo inferido |
| AC-003 | Rechazar cualquier otra transición con error en español sin modificar el estado | ✅ Cubierto | Mapeo inferido |
| AC-004 | 400 con campo y motivo para estado ausente/no permitido; 404 para incidencia inexistente | ✅ Cubierto | Mapeo inferido |
| AC-005 | Devolver la incidencia actualizada tras una transición válida | ✅ Cubierto | Mapeo inferido |
| AC-006 | Estado persistido y reflejado en detalle y listado | ✅ Cubierto | Mapeo inferido |
| AC-007 | Detalle con solo las acciones válidas, sin confirmación, nuevo estado y aviso de éxito | ✅ Cubierto | Mapeo inferido; sin pruebas AXE (axe-core no instalado) |
| AC-008 | Indicador de carga, bloqueo, error comprensible con estado anterior y opción de recargar | ✅ Cubierto | Mapeo inferido |
| AC-009 | Frontend consume el endpoint conforme al contrato y presenta el error de la API | ✅ Cubierto | Mapeo inferido |

## Matriz de trazabilidad

| Criterio | TC | Tipo | Evidencia | Ejecución | Resultado |
| -------- | -- | ---- | --------- | --------- | --------- |
| AC-001 | — | API Test | `helpdesk-backend/test/incidents.status.test.ts` | quality-check | Paso |
| AC-001 | — | E2E | `helpdesk-frontend/e2e/incident-status.spec.ts` | quality-check | Paso |
| AC-002 | — | API Test | `helpdesk-backend/test/incidents.status.test.ts` (transiciones válidas) | quality-check | Paso |
| AC-002 | — | Unit | `helpdesk-frontend/src/app/features/incidents/components/incident-status-actions/incident-status-actions.spec.ts` | quality-check | Paso |
| AC-003 | — | API Test | `helpdesk-backend/test/incidents.status.test.ts` (409), `test/error-handler.test.ts` | quality-check | Paso |
| AC-004 | — | API Test | `helpdesk-backend/test/incidents.status.test.ts` (400 y 404) | quality-check | Paso |
| AC-005 | — | API Test | `helpdesk-backend/test/incidents.status.test.ts` (200 sin comments) | quality-check | Paso |
| AC-006 | — | Integration | `helpdesk-backend/test/incidents.repository.test.ts`, `incidents.status.test.ts` (detalle y listado) | quality-check | Paso |
| AC-006 | — | Unit | `helpdesk-frontend/src/app/features/incidents/pages/incident-detail/incident-detail.spec.ts` | quality-check | Paso |
| AC-007 | — | Unit | `.../incident-status-actions.spec.ts` | quality-check | Paso |
| AC-007 | — | E2E | `helpdesk-frontend/e2e/incident-status.spec.ts` (ciclo completo) | quality-check | Paso |
| AC-008 | — | Unit | `.../incident-status-actions.spec.ts` (aplicando, 409, 500, red) | quality-check | Paso |
| AC-008 | — | E2E | `helpdesk-frontend/e2e/incident-status.spec.ts` (aplicando, 500, 409 con recarga) | quality-check | Paso |
| AC-009 | — | Unit | `.../incident-repository.spec.ts`, `.../incident-mapper.spec.ts` | quality-check | Paso |
| AC-009 | — | API Test | `helpdesk-backend/test/openapi.test.ts` (ruta y respuestas en el contrato) | quality-check | Paso |

## Observaciones y pendientes

- La US no tiene casos de prueba documentados (`test-cases/`); toda la trazabilidad es inferida. Considerar `/test-define` si se quiere trazabilidad declarada.
- No hay pruebas de accesibilidad con AXE en el detalle: `axe-core` no está instalado en helpdesk-frontend.
- La corrida e2e del frontend incluyó la corrección de dos pruebas preexistentes de US-002 (`e2e/incident-list.spec.ts`), aún sin commitear.

<!-- coverage-verify:verdict=APPROVED_WITH_NOTES · fingerprint=8baf151bc6c79442dd826a4c3fb5b5dc0f8d32f6 · spec=b15dd7bb35bc41b79ddf9804dc425930a88d426a · generated=2026-10-15 -->
