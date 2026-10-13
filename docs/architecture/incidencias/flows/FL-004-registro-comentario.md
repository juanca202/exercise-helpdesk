# FL-004: Registro de un comentario

**Disparador:** el agente envía el formulario de comentario desde la pantalla de detalle de una incidencia en `helpdesk-frontend`.
**Actores / componentes:** Agente de soporte; formulario de comentario (Angular, embebido en la pantalla de detalle); repositorio de incidencias del frontend; módulo `incidents` de `helpdesk-backend`; SQLite.
**Resultado:** el comentario queda persistido con `id`, `author` (usuario de demostración) y `createdAt`, aparece al final de la lista de comentarios sin recargar la página y el campo queda vacío; o, ante error, no se persiste nada y el texto escrito se conserva.

Recorrido de RF-04 / [US-004](../../../specs/changes/user-stories/US-004-comentar-incidencia/README.md) sobre [`POST /incidents/{id}/comments`](../apis/API-001-incidencias.md#post-incidents-id-comments). Parte de la pantalla de detalle ya cargada ([FL-003](FL-003-consulta-detalle-incidencia.md)). Fuera: edición y eliminación de comentarios (no existen).

## Flujo

```mermaid
sequenceDiagram
  actor A as Agente
  participant C as Formulario de comentario (Angular)
  participant R as Repositorio de incidencias (frontend)
  participant B as Módulo incidents (Fastify + Zod)
  participant DB as SQLite

  A->>C: Escribe el comentario (contador de caracteres) y envía
  C->>C: Validación de presentación (trim, 1–500)
  alt Contenido inválido
    C-->>A: Error junto al campo; no se envía
  else Contenido válido
    C->>C: Indicador de carga y bloqueo del formulario
    C->>R: Agregar comentario (id incidencia, content)
    R->>B: POST /incidents/{id}/comments (API-001)
    B->>B: Valida body contra el esquema Zod estricto
    alt Body inválido
      B-->>R: 400 con issues (sin persistir)
      R-->>C: Errores de validación
      C-->>A: Error junto al campo; texto conservado
    else Body válido
      B->>DB: ¿Existe la incidencia? (id UUID v4)
      alt No existe
        B-->>R: 404 Not Found (sin persistir)
        R-->>C: Incidencia no encontrada
        C-->>A: Mensaje de error; texto conservado
      else Existe
        B->>B: Asigna id, author = usuario de demostración, createdAt
        B->>DB: INSERT comentario (MD-002)
        alt Persistencia correcta
          B-->>R: 201 con el comentario
          R-->>C: Comentario creado
          C-->>A: Comentario añadido al final de la lista; campo vacío
        else Fallo inesperado
          B-->>R: 500 error genérico
          R-->>C: Error de registro
          C-->>A: Mensaje de error; texto conservado
        end
      end
    end
  end
```

## Pasos

1. El **agente** escribe el comentario; el **formulario** muestra un contador de caracteres (AC-008 de US-004).
2. Al enviar, el **formulario** aplica la validación de presentación con las reglas de `content` de [MD-002](../models/MD-002-comentario.md) (recorte, 1–500); si no se cumple, muestra el error junto al campo y no envía.
3. El **formulario** muestra el indicador de carga, se bloquea para evitar envíos duplicados (AC-009) e invoca al **repositorio**, que envía [`POST /incidents/{id}/comments`](../apis/API-001-incidencias.md#post-incidents-id-comments) con `{ content }`.
4. La **ruta Fastify** valida el body con el esquema Zod estricto (recorta `content`, rechaza claves no declaradas como `id`, `createdAt` o `author`); si no cumple, `400` con `issues` y mensaje propio en español, sin persistir (AC-002, AC-004).
5. El **servicio del módulo** comprueba que la incidencia existe (un `id` que no es UUID v4 cuenta como inexistente); si no, `404` sin persistir (AC-005).
6. El **servicio** asigna `id`, `incidentId`, `author` (usuario de demostración fijo) y `createdAt`, e inserta el comentario en **SQLite** en una única operación atómica.
7. El **backend** responde `201` con el comentario completo.
8. El **formulario** añade el comentario recibido al final de la lista de la pantalla de detalle (es el más reciente, orden ascendente), sin recargar la página, y vacía el campo (AC-006, AC-008).

## Manejo de errores

| Paso | Error posible | Comportamiento esperado |
| ---- | ------------- | ----------------------- |
| 2 | Contenido vacío, solo espacios o de más de 500 caracteres | Error junto al campo; no se envía |
| 4 | Body que no cumple el contrato | `400`; nada persistido; el formulario muestra el `issue` de `/content` junto al campo (AC-010), se desbloquea y conserva el texto |
| 5 | La incidencia no existe (p. ej. id manipulado en la URL) | `404`; nada persistido; mensaje comprensible; texto conservado |
| 6 | Fallo inesperado de persistencia | `500`; nada persistido (inserción atómica); mensaje comprensible; texto conservado y formulario habilitado para reintentar (AC-009) |
| 3 | Backend no disponible o error de red | Mismo comportamiento que el `500` |

## Notas

- **No idempotente:** dos envíos producen dos comentarios; el bloqueo del formulario durante el envío evita el doble envío.
- **Actualización de la lista sin recarga:** se usa el comentario devuelto en el `201` (incluye `id`, `author` y `createdAt` reales); no hace falta volver a pedir el detalle.
- **Autor visible:** el nombre del usuario de demostración lo fija el wireframe del formulario de comentario (ver Observaciones del [README de la capability](../README.md)).
