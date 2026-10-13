# Cobertura de criterios de aceptación — US-006: Consultar el dashboard de indicadores

**Veredicto:** ✅ Aprobado
**Fecha:** 2026-10-13 17:40
**Artefacto:** [US-006](./README.md)
**Alcance de esta corrida:** ramas `feature/US-006-dashboard-indicadores` de `helpdesk-backend` (integrada, commit 5686f50) y `helpdesk-frontend` (commit e223785).

## Resumen

Pruebas: resultados tomados de las corridas de `quality-check` del 2026-10-13 en ambos repositorios (backend: unit `PASS`, 61 passed; frontend: unit `PASS`, 185 passed, e2e `PASS`, 10 passed). La US no tiene `test-cases/` documentados, así que el mapeo se infirió desde los tests del repositorio.

| Criterios de aceptación | Cubiertos | Parciales | No cubiertos |
| ----------------------- | --------- | --------- | ------------ |
| 9 | 9 | 0 | 0 |

## Cobertura por criterio

| Criterio | Descripción | Estado | Observaciones |
| -------- | ----------- | ------ | ------------- |
| AC-001 | Consultar un dashboard con indicadores | ✅ Cubierto | Backend (API) y pantalla (unit + E2E). Mapeo inferido. |
| AC-002 | Total y recuento por estado | ✅ Cubierto | Backend y tarjetas del dashboard (unit). Mapeo inferido. |
| AC-003 | Total igual a la suma de los tres estados | ✅ Cubierto | Backend y verificación en pantalla (unit). Mapeo inferido. |
| AC-004 | Indicadores calculados al consultar | ✅ Cubierto | Backend con registro y cambio de estado entre consultas. Mapeo inferido. |
| AC-005 | Sin incidencias: ceros y mensaje | ✅ Cubierto | Ceros en backend y mensaje en pantalla (unit + E2E). Mapeo inferido. |
| AC-006 | Página principal con menú de navegación | ✅ Cubierto | Menú y rutas: main-layout.spec y E2E de navegación. Mapeo inferido. |
| AC-007 | Sin gráficos analíticos ni comparaciones | ✅ Cubierto | Aserción en el E2E del dashboard. Mapeo inferido. |
| AC-008 | Indicador de carga y error con reintento | ✅ Cubierto | Carga y error con reintento: unit y E2E. Mapeo inferido. |
| AC-009 | Consumo conforme al contrato OpenAPI | ✅ Cubierto | Contrato en backend (openapi.test) y consumo en el repositorio del frontend. Mapeo inferido. |

## Matriz de trazabilidad

| Criterio | TC | Tipo | Evidencia | Ejecución | Resultado |
| -------- | -- | ---- | --------- | --------- | --------- |
| AC-001 | — | API Test | `helpdesk-backend/test/incidents.summary.test.ts` | quality-check | PASS |
| AC-001 | — | E2E | `helpdesk-frontend/e2e/dashboard.spec.ts` | quality-check | PASS |
| AC-002 | — | API Test | `helpdesk-backend/test/incidents.summary.test.ts`, `helpdesk-backend/test/incidents.repository.test.ts` | quality-check | PASS |
| AC-002 | — | Unit | `helpdesk-frontend/src/app/features/incidents/pages/incident-dashboard/incident-dashboard.spec.ts` | quality-check | PASS |
| AC-003 | — | API Test | `helpdesk-backend/test/incidents.summary.test.ts` | quality-check | PASS |
| AC-003 | — | Unit | `helpdesk-frontend/src/app/features/incidents/pages/incident-dashboard/incident-dashboard.spec.ts` | quality-check | PASS |
| AC-004 | — | API Test | `helpdesk-backend/test/incidents.summary.test.ts` | quality-check | PASS |
| AC-005 | — | API Test | `helpdesk-backend/test/incidents.summary.test.ts` | quality-check | PASS |
| AC-005 | — | Unit | `helpdesk-frontend/src/app/features/incidents/pages/incident-dashboard/incident-dashboard.spec.ts` | quality-check | PASS |
| AC-006 | — | Unit | `helpdesk-frontend/src/app/shared/components/main-layout/main-layout.spec.ts` | quality-check | PASS |
| AC-006 | — | E2E | `helpdesk-frontend/e2e/dashboard.spec.ts` | quality-check | PASS |
| AC-007 | — | E2E | `helpdesk-frontend/e2e/dashboard.spec.ts` | quality-check | PASS |
| AC-008 | — | Unit | `helpdesk-frontend/src/app/features/incidents/pages/incident-dashboard/incident-dashboard.spec.ts` | quality-check | PASS |
| AC-008 | — | E2E | `helpdesk-frontend/e2e/dashboard.spec.ts` | quality-check | PASS |
| AC-009 | — | API Test | `helpdesk-backend/test/openapi.test.ts` | quality-check | PASS |
| AC-009 | — | Unit | `helpdesk-frontend/src/app/features/incidents/repositories/incident-repository.spec.ts` | quality-check | PASS |

## Observaciones y pendientes

- Ningún criterio tiene `TC-XXX` documentado: el mapeo es inferido desde los tests; considerar `/test-define` si se quiere trazabilidad declarada.
- Las pruebas AXE de accesibilidad no existen: `axe-core` no está instalado en `helpdesk-frontend`.
- Las suites `coverage` de ambos repositorios son métricas de `quality-check` y no se mapean a criterios.

<!-- coverage-verify:verdict=APPROVED · fingerprint=db816412ace47baf0dc9421e8236c64ccb6fd56d · spec=77870e212e836a30bdfc98becf6cd0aeb0dd0be7 · generated=2026-10-13 -->
