# MD-003: Indicadores de incidencias

Proyección de solo lectura (no persistida) que resume la carga de trabajo: el total de incidencias ([MD-001](MD-001-incidencia.md)) y cuántas hay en cada estado. La calcula el backend en cada consulta a [`GET /incidents/summary`](../apis/API-001-incidencias.md#get-incidents-summary) y la consume el dashboard de [US-006](../../../specs/changes/user-stories/US-006-dashboard-indicadores/README.md).

## Campos

| Campo | Tipo | Requerido | Descripción | Validaciones / restricciones |
| ----- | ---- | --------- | ----------- | ---------------------------- |
| total | integer | Sí | Total de incidencias registradas | ≥ 0; siempre igual a `open + inProgress + resolved` |
| open | integer | Sí | Incidencias con `status = Open` | ≥ 0 |
| inProgress | integer | Sí | Incidencias con `status = In Progress` | ≥ 0 |
| resolved | integer | Sí | Incidencias con `status = Resolved` | ≥ 0 |

## Notas

- **Cálculo al consultar:** se obtiene con un recuento agrupado por `status` sobre las incidencias persistidas en el momento de la petición; no hay caché ni valores precalculados, por lo que refleja todo registro y cambio de estado previo (AC-004 de US-006).
- **Coherencia del total:** `total` se calcula como la suma de los tres recuentos por estado, de modo que la igualdad `total = open + inProgress + resolved` se cumple por construcción (BR-02 de US-006). Como `status` solo admite esos tres valores ([MD-001](MD-001-incidencia.md)), coincide con el número de incidencias.
- **Sin incidencias:** los cuatro valores son `0`; no es un error (AC-005 de US-006).
- **Etiquetas en la UI:** `open` ↔ abiertas, `inProgress` ↔ en atención, `resolved` ↔ resueltas (AC-002 de US-006); los textos visibles los fija el wireframe del dashboard.
