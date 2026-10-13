# Reporte de cobertura — US-001-registrar-incidencia

**Veredicto:** ⚠️ APPROVED_WITH_NOTES
**Fecha:** 2026-10-13
**Rama:** main (especificaciones) · código en `feature/US-001-registrar-incidencia` de helpdesk-backend y helpdesk-frontend
**Commit:** 5660d9e (especificaciones) · backend 0121912 · frontend b4c9a2a
**Trabajo:** [US-001](./README.md)

## Resumen

Los once criterios tienen pruebas automatizadas que pasan; diez quedan cubiertos por completo y AC-010 queda parcial porque la conformidad del frontend con el contrato OpenAPI no se comprueba de forma automática entre repositorios. No hay criterios sin cobertura.

**Pruebas:** resultados tomados de la caché fresca de `quality-check` de cada worktree (backend: commit 0121912, 2026-10-13; frontend: commit b4c9a2a, 2026-10-13); el `fingerprint` de ambas coincide con el recalculado ahora. Backend: unit `PASS` (53 passed) · coverage `PASS`. Frontend: unit `PASS` (173 passed) · coverage `PASS` · e2e `PASS` (5 passed). La US no tiene carpeta `test-cases/`: la evidencia son las pruebas automatizadas de ambos repositorios y el mapeo es inferido, no declarado por TC.

**Cobertura de criterios de aceptación**

| Total | ✅ COVERED | ⚠️ PARTIAL | ❌ UNCOVERED |
| ----- | --------- | --------- | ------------ |
| 11    | 10        | 1         | 0            |

## Cobertura por criterio

Vista de veredicto: un criterio por fila. El detalle de qué lo prueba está en la matriz de abajo.

| Criterio | Descripción | Estado | Observaciones |
| -------- | ----------- | ------ | ------------- |
| AC-001 | Registrar una incidencia con título, descripción, categoría y prioridad | ✅ COVERED | Mapeo inferido (sin TC). Backend (API) + formulario + e2e. |
| AC-002 | Asignar id único, fecha de creación y estado `Open` y devolverlos | ✅ COVERED | Mapeo inferido (sin TC). Id validado contra UUID v4 y `createdAt` como ISO. |
| AC-003 | Persistencia y recuperación por id con los mismos datos | ✅ COVERED | Mapeo inferido (sin TC). Verificado por API (`GET /incidents/:id`), según lo acordado en la US. |
| AC-004 | Rechazar título ausente, vacío tras trim o fuera de 5–100 | ✅ COVERED | Mapeo inferido (sin TC). Backend cubre ausente, solo espacios, 4 y 101 caracteres y los límites; frontend cubre vacío, solo espacios y corto. |
| AC-005 | Rechazar descripción ausente, vacía tras trim o fuera de 10–2000 | ✅ COVERED | Mapeo inferido (sin TC). Backend cubre ausente, solo espacios, 9 y 2001 caracteres; frontend cubre solo espacios y >2000. |
| AC-006 | Rechazar categoría o prioridad ausente o fuera del catálogo | ✅ COVERED | Mapeo inferido (sin TC). Backend cubre ausente y valor fuera de catálogo en ambos campos; frontend cubre el caso ausente en el envío vacío. |
| AC-007 | Error 400 por campo con motivo en español y sin persistir | ✅ COVERED | Mapeo inferido (sin TC). La aserción del idioma es una expresión regular amplia (`/[áéíóúñ]\|debe\|obligatori\|válid/i`): comprueba español de forma indicativa, no el texto exacto. |
| AC-008 | Errores visibles por campo, sin envío con errores y contadores | ✅ COVERED | Mapeo inferido (sin TC). Accesibilidad verificada con aserciones `aria-*` (sin axe-core, decisión acordada). |
| AC-009 | Indicador de carga, confirmación con id, error comprensible, bloqueo y conservación de datos | ✅ COVERED | Mapeo inferido (sin TC). |
| AC-010 | Consumo del endpoint conforme al contrato OpenAPI y presentación de errores de la API | ⚠️ PARTIAL | La presentación de errores está probada; la conformidad con el contrato se apoya en mocks (MSW y `page.route`) escritos a mano según API-001, sin verificación automática contra el documento OpenAPI del backend. |
| AC-011 | Rechazar con 400 `id`, `status` o `createdAt` sin persistir | ✅ COVERED | Mapeo inferido (sin TC). Backend rechaza también claves no declaradas; frontend no las envía. |

## Matriz de trazabilidad

Vista auditable: **una fila por cada combinación criterio × caso de prueba × tipo de prueba declarado**. No hay TC documentados: las filas se derivan de las pruebas halladas (`TC = —`).

Rutas relativas a `helpdesk-backend/.worktrees/US-001-registrar-incidencia/` (BE) y `helpdesk-frontend/.worktrees/US-001-registrar-incidencia/` (FE).

| Criterio | TC | Tipo | Evidencia | Ejecución | Resultado |
| -------- | -- | ---- | --------- | --------- | --------- |
| AC-001 | — | API Test | BE `test/incidents.create.test.ts` («registra una incidencia válida con id, estado Open y fecha de creación») | quality-check | ✅ PASS |
| AC-001 | — | Unit | FE `src/app/features/incidents/pages/incident-form/incident-form.spec.ts` («should show the confirmation with the identifier assigned by the API») | quality-check | ✅ PASS |
| AC-001 | — | E2E | FE `e2e/incident-registration.spec.ts` («registra una incidencia válida y muestra la confirmación con el identificador») | quality-check | ✅ PASS |
| AC-002 | — | API Test | BE `test/incidents.create.test.ts` («registra una incidencia válida…», «persiste la incidencia con los datos devueltos») | quality-check | ✅ PASS |
| AC-002 | — | Unit | FE `src/app/features/incidents/repositories/incident-repository.spec.ts` («should register the incident and return it with the data assigned by the API») | quality-check | ✅ PASS |
| AC-003 | — | API Test | BE `test/incidents.get.test.ts` («recupera la incidencia registrada con los mismos datos») | quality-check | ✅ PASS |
| AC-003 | — | Unit | BE `test/incidents.repository.test.ts` («recupera la incidencia insertada con los mismos datos») | quality-check | ✅ PASS |
| AC-004 | — | API Test | BE `test/incidents.create.test.ts` (`it.each` «título ausente / de solo espacios / de 4 / de 101 caracteres», «acepta los límites…», «recorta los espacios…») | quality-check | ✅ PASS |
| AC-004 | — | Unit | FE `incident-form.spec.ts` («should reject a title shorter than 5 characters after trimming», «should treat a title made only of spaces as empty and send nothing») | quality-check | ✅ PASS |
| AC-005 | — | API Test | BE `test/incidents.create.test.ts` (`it.each` «descripción ausente / de solo espacios / de 9 / de 2001 caracteres») | quality-check | ✅ PASS |
| AC-005 | — | Unit | FE `incident-form.spec.ts` («should treat a description made only of spaces as empty and send nothing», «should reject a description longer than 2000 characters») | quality-check | ✅ PASS |
| AC-006 | — | API Test | BE `test/incidents.create.test.ts` (`it.each` «categoría ausente / fuera del catálogo», «prioridad ausente / fuera del catálogo») | quality-check | ✅ PASS |
| AC-006 | — | Unit | FE `incident-form.spec.ts` («should show every error and send nothing when submitting an empty form») | quality-check | ✅ PASS |
| AC-007 | — | API Test | BE `test/incidents.create.test.ts` (`it.each` «rechaza con 400 y no persiste: %s», «informa todos los campos inválidos en una sola respuesta») | quality-check | ✅ PASS |
| AC-007 | — | API Test | BE `test/error-handler.test.ts` («responde 400 cuando el cuerpo no cumple el contrato») | quality-check | ✅ PASS |
| AC-008 | — | Unit | FE `incident-form.spec.ts` («should show every error and send nothing…», «should expose the invalid state to assistive technology», «should update the counters while typing», «should count the characters after trimming…») | quality-check | ✅ PASS |
| AC-008 | — | E2E | FE `e2e/incident-registration.spec.ts` («muestra los errores por campo y no envía nada al enviar el formulario vacío») | quality-check | ✅ PASS |
| AC-009 | — | Unit | FE `incident-form.spec.ts` («should lock the form and show the loading text while the request is in flight», «should show the confirmation with the identifier…», «should show a general alert and keep the data when the API fails», «should keep the data and allow retrying after a network error») | quality-check | ✅ PASS |
| AC-009 | — | E2E | FE `e2e/incident-registration.spec.ts` («muestra la alerta general si la API falla y conserva los datos para reintentar») | quality-check | ✅ PASS |
| AC-010 | — | Unit | FE `incident-repository.spec.ts` («should send only the fields provided by the requester, trimmed», «should expose the validation issues when the API answers 400») | quality-check | ✅ PASS |
| AC-010 | — | Unit | FE `incident-form.spec.ts` («should show the field errors returned by the API with a 400», «should show a general alert when a 400 carries issues without a known field») | quality-check | ✅ PASS |
| AC-010 | — | E2E | FE `e2e/incident-registration.spec.ts` («presenta el error de validación de la API en su campo y conserva los datos») | quality-check | ✅ PASS |
| AC-010 | — | API Test | BE `test/openapi.test.ts` («publica POST /incidents con sus respuestas 201, 400 y 500») | quality-check | ✅ PASS |
| AC-011 | — | API Test | BE `test/incidents.create.test.ts` (`it.each` «rechaza con 400 si el cliente envía id / status / createdAt / una clave no declarada») | quality-check | ✅ PASS |
| AC-011 | — | Unit | FE `src/app/features/incidents/utils/incident-mapper.spec.ts` («should not send fields assigned by the system») | quality-check | ✅ PASS |

## Observaciones y pendientes

- Ambos worktrees tienen `workingTreeClean: false` en su corrida de `quality-check`; los resultados corresponden al estado del árbol con cambios sin commitear (el `fingerprint` coincide con el actual).
- Sin carpeta `test-cases/` en la US: todo el mapeo es inferido desde los tests del repositorio; si se ejecuta `test-define`, el reporte debe revalidarse.
- AC-010: para pasar a `COVERED` haría falta una verificación automática de los mocks del frontend contra el documento OpenAPI del backend (por ejemplo, un test de contrato).
- Las suites `architecture` de ambos repositorios dieron `PASS` y no se mapean a criterios.

<!-- coverage-verify:verdict=APPROVED_WITH_NOTES · fingerprint=c9270e9d1557a7e0e512a3ccf06015202df10353+c80a6cabe75b6ebdf38201e881dbc550f5fbfbc4 · spec=0e899162e74c90858f0a141abe6e1ea68d9cbaef · generated=2026-10-13 -->
