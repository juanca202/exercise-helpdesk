# Cobertura de criterios — US-003: Consultar el detalle de una incidencia

**Veredicto:** ⚠️ Aprobado con observaciones
**Fecha:** 2026-10-14 11:16
**Artefacto:** [US-003](./README.md)

## Resumen

Se trazaron los 8 criterios de aceptación contra las pruebas automatizadas de [helpdesk-backend](../../../../../helpdesk-backend) y [helpdesk-frontend](../../../../../helpdesk-frontend); la US no tiene `test-cases/`, así que la evidencia son los tests del repositorio. Pruebas: resultados tomados de las corridas de `quality-check` de esta sesión en ambos repos (unit y cobertura en verde; backend con e2e de Fastify, frontend con 11 specs Playwright en verde); no existe `quality-check-run.json` de esta corrida.

| Criterios de aceptación | Cubiertos | Parciales | No cubiertos |
| ----------------------- | --------- | --------- | ------------ |
| 8 | 7 | 1 | 0 |

## Cobertura por criterio

| Criterio | Descripción | Estado | Observaciones |
| -------- | ----------- | ------ | ------------- |
| AC-001 | Consultar una incidencia por su identificador | ✅ Cubierto | Mapeo inferido de los tests, sin TC documentado. |
| AC-002 | El detalle incluye todos los datos y comentarios | ✅ Cubierto | Mapeo inferido. |
| AC-003 | Comentarios en orden ascendente y lista vacía | ✅ Cubierto | Mapeo inferido. |
| AC-004 | 404 ante id inexistente o no UUID v4 | ✅ Cubierto | Mapeo inferido. |
| AC-005 | Dirección propia `/incidents/{id}` | ✅ Cubierto | Mapeo inferido. |
| AC-006 | Mensaje «no encontrada» con vuelta al listado | ✅ Cubierto | Mapeo inferido. «Volver al listado» apunta a `/incidents`, aún sin pantalla (US-002). |
| AC-007 | Carga, vacío y error con reintento | ✅ Cubierto | Mapeo inferido. El estado de carga solo se verifica en unitaria. |
| AC-008 | El frontend consume el contrato OpenAPI | ⚠️ Parcial | Los specs usan MSW y fixtures escritos a mano; no hay verificación automática del contrato frente al OpenAPI del backend. |

## Matriz de trazabilidad

| Criterio | TC | Tipo | Evidencia | Ejecución | Resultado |
| -------- | -- | ---- | --------- | --------- | --------- |
| AC-001 | — | Integration | `helpdesk-backend/test/incidents.get.test.ts` | quality-check | Paso |
| AC-001 | — | Unit | `helpdesk-frontend/src/app/features/incidents/repositories/incident-repository.spec.ts` | quality-check | Paso |
| AC-001 | — | E2E | `helpdesk-frontend/e2e/incident-detail.spec.ts` | quality-check | Paso |
| AC-002 | — | Integration | `helpdesk-backend/test/incidents.get.test.ts` | quality-check | Paso |
| AC-002 | — | Unit | `helpdesk-frontend/src/app/features/incidents/pages/incident-detail/incident-detail.spec.ts` | quality-check | Paso |
| AC-003 | — | Integration | `helpdesk-backend/test/incidents.get.test.ts`, `helpdesk-backend/test/comments.repository.test.ts` | quality-check | Paso |
| AC-003 | — | E2E | `helpdesk-frontend/e2e/incident-detail.spec.ts` | quality-check | Paso |
| AC-004 | — | Integration | `helpdesk-backend/test/incidents.get.test.ts`, `helpdesk-backend/test/error-handler.test.ts` | quality-check | Paso |
| AC-005 | — | E2E | `helpdesk-frontend/e2e/incident-detail.spec.ts` (apertura directa por URL) | quality-check | Paso |
| AC-006 | — | Unit | `helpdesk-frontend/.../incident-detail.spec.ts` | quality-check | Paso |
| AC-006 | — | E2E | `helpdesk-frontend/e2e/incident-detail.spec.ts` | quality-check | Paso |
| AC-007 | — | Unit | `helpdesk-frontend/.../incident-detail.spec.ts` | quality-check | Paso |
| AC-007 | — | E2E | `helpdesk-frontend/e2e/incident-detail.spec.ts` | quality-check | Paso |
| AC-008 | — | Unit | `helpdesk-frontend/.../incident-repository.spec.ts`, `helpdesk-backend/test/incidents.get.test.ts` (contrato publicado) | quality-check | Paso |

## Observaciones y pendientes

- Las pruebas de accesibilidad (AXE) previstas en TK-004 no se automatizaron: `axe-core` no está instalado. No corresponden a un criterio de aceptación, pero conviene decidirlo.
- La US no tiene test cases documentados; ejecutar `test-define` daría trazabilidad TC ↔ AC explícita.

<!-- coverage-verify:verdict=APPROVED_WITH_NOTES · fingerprint=a85e579a86abdb780d880d30f49c8fd6b6e564b2 · spec=5e740bea1f0abd97361b2023fd8c4ef03b61280f · generated=2026-10-14 -->
