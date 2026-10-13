# FL-001: Registro de una incidencia

**Disparador:** el empleado envía el formulario de registro de incidencia en `helpdesk-frontend`.
**Actores / componentes:** Empleado; formulario de registro (Angular); repositorio de incidencias del frontend (patrón Repository REST); módulo `incidents` de `helpdesk-backend` (ruta Fastify + esquema Zod + servicio); SQLite.
**Resultado:** la incidencia queda persistida con `id`, `status = Open` y `createdAt`, y el formulario muestra la confirmación con su identificador; o, ante datos inválidos, no se persiste nada y el formulario muestra el error junto a cada campo.

Recorrido extremo a extremo de RF-01 / [US-001](../../../specs/changes/user-stories/US-001-registrar-incidencia/README.md): validación de presentación en el formulario, envío conforme al contrato OpenAPI, validación de contrato en el backend, persistencia y respuesta. Fuera de este flujo: consulta del listado ([FL-002](FL-002-consulta-listado-incidencias.md)), detalle ([FL-003](FL-003-consulta-detalle-incidencia.md)), comentarios ([FL-004](FL-004-registro-comentario.md)) y cambios de estado ([FL-005](FL-005-cambio-estado-incidencia.md)).

## Flujo

```mermaid
sequenceDiagram
  actor E as Empleado
  participant F as Formulario de registro (Angular)
  participant R as Repositorio de incidencias (frontend)
  participant B as Módulo incidents (Fastify + Zod)
  participant DB as SQLite

  E->>F: Completa título, descripción, categoría y prioridad y envía
  F->>F: Validación de presentación (mismas reglas que MD-001)
  alt Errores de presentación
    F-->>E: Error visible junto a cada campo; no se envía (AC-008)
  else Formulario válido
    F->>F: Muestra indicador de carga y bloquea el formulario (AC-009)
    F->>R: Registrar incidencia
    R->>B: POST /incidents (API-001)
    B->>B: Valida body contra el esquema Zod
    alt Body inválido
      B-->>R: 400 con issues por campo (sin persistir)
      R-->>F: Errores de validación
      F-->>E: Error junto a cada campo devuelto por la API (AC-007, AC-010)
    else Body válido
      B->>B: Asigna id, status = Open, createdAt
      B->>DB: INSERT incidencia (MD-001)
      alt Persistencia correcta
        DB-->>B: OK
        B-->>R: 201 con la incidencia
        R-->>F: Incidencia registrada
        F-->>E: Confirmación con el identificador (AC-009)
      else Fallo inesperado
        DB-->>B: Error
        B-->>R: 500 error genérico
        R-->>F: Error de registro
        F-->>E: Alerta de error comprensible; datos conservados para reintentar (AC-009)
      end
    end
  end
```

## Pasos

1. El **empleado** completa título, descripción, categoría y prioridad en el formulario y lo envía.
2. El **formulario** aplica la validación de presentación con las mismas reglas de [MD-001](../models/MD-001-incidencia.md) (obligatoriedad, recorte de espacios, longitudes, valores de enum). El botón de envío no se deshabilita por errores: al pulsarlo, si hay errores, los muestra todos junto a cada campo y no envía la solicitud (AC-008).
3. El **formulario** muestra el indicador de carga, bloquea campos y botón mientras dura el envío (evita envíos duplicados) e invoca al **repositorio de incidencias** del frontend, que consume el contrato OpenAPI publicado por el backend (no redefine el modelo a mano — estándar `contrato-compartido`).
4. El **repositorio** envía [`POST /incidents`](../apis/API-001-incidencias.md#post-incidents) con `title`, `description`, `category` y `priority`.
5. La **ruta Fastify** del módulo `incidents` valida el body contra el esquema Zod estricto (recorta `title`/`description` antes de validar y rechaza claves no declaradas, incluidos `id`, `status` y `createdAt`). Si no cumple, el manejador de errores común responde `400` con un elemento en `issues` por problema, con mensaje propio en español, y no se ejecuta ninguna escritura (BR-04).
6. El **servicio del módulo** asigna `id`, `status = Open` y `createdAt` (BR-01) e inserta la incidencia en **SQLite** en una única operación atómica.
7. El **backend** responde `201` con la incidencia completa ([MD-001](../models/MD-001-incidencia.md)).
8. El **formulario** retira el indicador de carga y se sustituye por la confirmación con el identificador recibido (UUID v4) y el título como referencia; «Registrar otra incidencia» vuelve al formulario vacío (AC-009, [WF-001](../../../specs/changes/user-stories/US-001-registrar-incidencia/wireframes/WF-001-registro-incidencia.md)). La incidencia queda recuperable con [`GET /incidents/{id}`](../apis/API-001-incidencias.md#get-incidents-id) (AC-003), que la devuelve con `comments: []` mientras no tenga comentarios, y nace en `Open`, estado inicial del ciclo de vida de [DG-001](../diagrams/DG-001-estados-incidencia.md).

## Manejo de errores

| Paso | Error posible | Comportamiento esperado |
| ---- | ------------- | ----------------------- |
| 2 | Campo ausente, vacío (o solo espacios), fuera de longitud o con valor no permitido | Todos los errores visibles junto a su campo; no se envía la solicitud |
| 5 | Body que no cumple el contrato (validación del backend) | `400` con `issues` (mensajes propios en español); nada persistido; el formulario asocia cada `issue` a su campo (vía `instancePath`) y lo muestra junto a él, igual que los errores de presentación; se desbloquea con los datos conservados |
| 6 | Fallo inesperado de persistencia | `500` con error genérico; nada persistido (inserción atómica); el formulario muestra sobre el formulario la alerta «No se pudo registrar la incidencia» con un motivo comprensible, conserva los datos y vuelve a habilitar el botón para reintentar |
| 4 | Backend no disponible o error de red | Mismo comportamiento que el `500`: alerta sobre el formulario, datos conservados y botón habilitado para que el empleado reintente (AC-009) |

## Notas

- **Doble validación:** el formulario replica las reglas de [MD-001](../models/MD-001-incidencia.md) para no enviar solicitudes inválidas, pero la validación de contrato del backend es la autoridad (AC-004..AC-007 se verifican contra la API).
- **Concurrencia/idempotencia:** el envío no es idempotente; dos envíos producen dos incidencias. El doble envío se evita bloqueando el formulario completo mientras dura el envío.
- **Comportamiento de UI:** sigue el wireframe aprobado [WF-001](../../../specs/changes/user-stories/US-001-registrar-incidencia/wireframes/WF-001-registro-incidencia.md): tras el éxito, el formulario se sustituye por la confirmación; tras un error, los datos se conservan; los errores no asociados a un campo (`500`, red) se muestran como alerta sobre el formulario; no hay reintento automático: reintenta el empleado con el botón de envío.

## Historial de cambios

| Fecha | Cambio | Origen |
| ----- | ------ | ------ |
| 2026-10-12 | Enlazados los flujos vecinos (FL-002..FL-005) en el alcance y, en el paso 8, la respuesta de detalle con `comments` y el estado inicial de [DG-001](../diagrams/DG-001-estados-incidencia.md); sin cambios en el recorrido del registro | [US-002](../../../specs/changes/user-stories/US-002-listar-filtrar-incidencias/README.md)..[US-005](../../../specs/changes/user-stories/US-005-cambiar-estado-incidencia/README.md) |
