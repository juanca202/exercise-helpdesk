# FL-005: Cambio de estado de una incidencia

**Disparador:** el agente pulsa en la pantalla de detalle una de las acciones de estado disponibles (iniciar atención, resolver o reabrir).
**Actores / componentes:** Agente de soporte; acciones de estado de la pantalla de detalle (Angular); repositorio de incidencias del frontend; módulo `incidents` de `helpdesk-backend`; SQLite.
**Resultado:** la incidencia queda persistida con el nuevo estado y el detalle lo muestra con las acciones del nuevo estado; o, si la transición se rechaza o falla, el estado no cambia y la pantalla conserva el anterior con un mensaje de error.

Recorrido de RF-05 / [US-005](../../../specs/changes/user-stories/US-005-cambiar-estado-incidencia/README.md) sobre [`PATCH /incidents/{id}/status`](../apis/API-001-incidencias.md#patch-incidents-id-status), con las transiciones de [DG-001](../diagrams/DG-001-estados-incidencia.md). El nuevo estado se refleja en el listado ([FL-002](FL-002-consulta-listado-incidencias.md)) y en los indicadores en la siguiente consulta.

## Flujo

```mermaid
sequenceDiagram
  actor A as Agente
  participant D as Acciones de estado (pantalla de detalle)
  participant R as Repositorio de incidencias (frontend)
  participant B as Módulo incidents (Fastify + Zod)
  participant DB as SQLite

  D->>D: Ofrece solo las acciones válidas desde el estado actual (DG-001)
  A->>D: Pulsa la acción (sin confirmación)
  D->>D: Indicador de carga y bloqueo de las acciones
  D->>R: Cambiar estado (id, status destino)
  R->>B: PATCH /incidents/{id}/status (API-001)
  B->>B: Valida body contra el esquema Zod estricto
  alt Body inválido
    B-->>R: 400 con issues
  else Body válido
    B->>DB: SELECT incidencia por id (id UUID v4)
    alt No existe
      B-->>R: 404 Not Found
    else Existe
      B->>B: ¿(estado actual → destino) está permitida?
      alt No permitida (incluido mismo estado)
        B-->>R: 409 Conflict (sin cambios)
      else Permitida
        B->>DB: UPDATE status = destino WHERE id = ? AND status = actual
        alt 1 fila actualizada
          B-->>R: 200 incidencia actualizada
        else 0 filas (cambio concurrente)
          B-->>R: 409 Conflict (sin cambios)
        end
      end
    end
  end
  alt 200
    R-->>D: Incidencia actualizada
    D-->>A: Nuevo estado y acciones del nuevo estado
  else 400 / 404 / 409 / 500 / red
    R-->>D: Error
    D-->>A: Mensaje de error comprensible; estado anterior conservado
  end
```

## Pasos

1. La **pantalla de detalle** ofrece únicamente las acciones válidas desde el estado actual según [DG-001](../diagrams/DG-001-estados-incidencia.md): `Open` → «Iniciar atención»; `In Progress` → «Resolver»; `Resolved` → «Reabrir» (AC-007 de US-005).
2. El **agente** pulsa una acción; no se pide confirmación.
3. La **pantalla** muestra el indicador de carga, bloquea las acciones (AC-008) e invoca al **repositorio**, que envía [`PATCH /incidents/{id}/status`](../apis/API-001-incidencias.md#patch-incidents-id-status) con `{ status: <destino> }`.
4. La **ruta Fastify** valida el body con el esquema Zod estricto; un `status` ausente o fuera de los valores permitidos responde `400` (AC-004).
5. El **servicio del módulo** busca la incidencia ([MD-001](../models/MD-001-incidencia.md)); si no existe (incluido un id que no es UUID v4) responde `404` (AC-004).
6. El **servicio** comprueba que la transición estado actual → destino está permitida; si no lo está (incluido el mismo estado), responde `409` con un mensaje en español que indica que la transición no está permitida desde el estado actual, sin modificar nada (BR-02, AC-003).
7. El **servicio** aplica la transición con una única actualización condicionada al estado leído; si afecta una fila responde `200` con la incidencia actualizada (AC-005); si no afecta ninguna (otro cambio concurrente) responde `409`.
8. La **pantalla** retira el indicador de carga y muestra el nuevo estado y las acciones que le corresponden (AC-006, AC-007).

## Manejo de errores

| Paso | Error posible | Comportamiento esperado |
| ---- | ------------- | ----------------------- |
| 4 | `status` ausente o no permitido | `400` con `issue` en `/status`; nada modificado; la pantalla muestra el error y conserva el estado anterior. En uso normal no ocurre: las acciones envían valores válidos |
| 5 | Incidencia inexistente | `404`; nada modificado; mensaje comprensible |
| 6 | Transición no permitida (p. ej. el estado cambió en otra pestaña) | `409`; nada modificado; la pantalla presenta el mensaje devuelto por la API (AC-009) y conserva el estado anterior |
| 7 | Cambio concurrente entre lectura y escritura | `409`; nada modificado (actualización condicionada) |
| 7 | Fallo inesperado de persistencia | `500`; nada modificado (sentencia única atómica); mensaje comprensible; estado anterior conservado (AC-008) |
| 3 | Backend no disponible o error de red | Mismo comportamiento que el `500` |

## Notas

- **Sin cambios parciales:** la transición es una sola sentencia condicionada, por lo que una transición rechazada o fallida no deja datos modificados ([persistence.md](../../../../helpdesk-backend/docs/standards/persistence.md), CR-002).
- **Tras un `409`**, la pantalla puede volver a pedir el detalle ([FL-003](FL-003-consulta-detalle-incidencia.md)) para mostrar el estado real; US-005 solo exige conservar el estado anterior y mostrar el error, así que se decide en la planificación técnica.
- **Idempotencia:** repetir la misma transición tras aplicarse responde `409` (el estado ya no es el origen); no produce un segundo cambio.
