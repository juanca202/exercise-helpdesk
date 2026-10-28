---
id: US-001
type: user-story
status: Ready
created: 2026-10-12
updated: 2026-10-12
repositories: [helpdesk-backend, helpdesk-frontend]
invest: { met: 5, partial: 1, not_met: 0, total: 6 }
dor: { met: 6, partial: 0, not_met: 0, total: 6 }
requirement: ../../requirements/PRD-001-helpdesk-lite/README.md
---
# US-001: Registrar una incidencia

## Descripción

**COMO** empleado solicitante
**QUIERO** registrar una solicitud de soporte indicando título, descripción, categoría y prioridad
**PARA** que el equipo de soporte pueda atenderla y yo pueda consultarla posteriormente

## Contexto

Origen: US-01 / RF-01 del PRD, registrado como [PRD-001: HelpDesk Lite](../../requirements/PRD-001-helpdesk-lite/README.md) (RF-01). La aplicación es de demostración: no hay autenticación real, el actor se representa con un usuario predefinido. Comprende el formulario de registro (frontend) y el endpoint de creación con su persistencia (backend), acordados mediante el contrato OpenAPI compartido.

## Fuera de alcance

- Consulta, filtrado y paginación del listado de incidencias (RF-02).
- Detalle, comentarios y cambios de estado de la incidencia (RF-03, RF-04, RF-05).
- Edición o eliminación de una incidencia registrada.
- Autenticación y roles reales; archivos adjuntos; notificaciones.

## Reglas de negocio

- **BR-01:** El sistema DEBE asignar a cada incidencia registrada un identificador único (UUID v4), una fecha de creación y el estado inicial `Open`; estos datos NO DEBEN ser proporcionados por el solicitante → verificado por AC-002, AC-011
- **BR-02:** El título, la descripción, la categoría y la prioridad DEBEN ser obligatorios → verificado por AC-004, AC-005, AC-006
- **BR-03:** La categoría DEBE ser una de `HARDWARE`, `SOFTWARE`, `ACCESS_PERMISSIONS`, `NETWORK_CONNECTIVITY`, `EMAIL_COMMUNICATION`, `OTHER`, y la prioridad DEBE ser una de `LOW`, `MEDIUM`, `HIGH` (mostradas como Baja, Media, Alta) → verificado por AC-006
- **BR-04:** Una solicitud inválida NO DEBE producir ninguna incidencia persistida → verificado por AC-007
- **BR-05:** El título y la descripción DEBEN recortarse (trim) antes de validarse; un valor compuesto solo por espacios DEBE considerarse vacío → verificado por AC-004, AC-005

## Criterios de aceptación

- **AC-001 (Casos de uso):** El sistema DEBE permitir al empleado registrar una incidencia indicando título, descripción, categoría y prioridad.
- **AC-002 (Reglas de negocio):** Al registrar una incidencia válida, el sistema DEBE asignarle un identificador único, una fecha de creación y el estado inicial `Open`, y DEBE devolverlos en la respuesta.
- **AC-003 (Procesamiento de datos):** La incidencia registrada DEBE quedar persistida y DEBE poder recuperarse posteriormente por su identificador con los datos con que fue registrada.
- **AC-004 (Reglas de negocio):** El sistema DEBE rechazar una incidencia cuyo título esté ausente, vacío (tras recortar espacios) o tenga una longitud fuera de 5 a 100 caracteres.
- **AC-005 (Reglas de negocio):** El sistema DEBE rechazar una incidencia cuya descripción esté ausente, vacía (tras recortar espacios) o tenga una longitud fuera de 10 a 2000 caracteres.
- **AC-006 (Reglas de negocio):** El sistema DEBE rechazar una incidencia cuya categoría o prioridad esté ausente o no pertenezca a los valores permitidos (BR-03).
- **AC-007 (Salidas del sistema):** Ante una solicitud inválida, el sistema DEBE responder con un error 400 que indique, por cada campo inválido, el campo y un motivo comprensible redactado en español, y NO DEBE persistir ninguna incidencia.
- **AC-008 (Interacción de usuario):** El formulario DEBE mostrar de forma visible, junto a cada campo, su error de validación; al intentar enviar con errores DEBE mostrarlos todos y NO DEBE enviar la solicitud. El formulario DEBE mostrar contadores de caracteres para título y descripción.
- **AC-009 (Usabilidad):** El formulario DEBE mostrar un indicador de carga mientras se envía la solicitud, una confirmación con el identificador de la incidencia registrada al tener éxito, y un mensaje de error comprensible si el registro falla. Mientras se envía, el formulario DEBE bloquearse para evitar envíos duplicados, y si el registro falla DEBE conservar los datos escritos.
- **AC-010 (Integraciones):** El frontend DEBE consumir el endpoint de registro conforme al contrato OpenAPI del backend y DEBE presentar los errores de validación devueltos por la API.
- **AC-011 (Reglas de negocio):** El sistema DEBE rechazar con un error 400 toda solicitud de registro que incluya `id`, `status` o `createdAt`, sin persistir ninguna incidencia.

## Referencias

- **Diseño / prototipo:** Ninguno aportado por el usuario.
- **Wireframe:** [WF-001: Registro de incidencia](wireframes/WF-001-registro-incidencia.md) — Approved.
- **Modelo:** [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md)
- **API:** [API-001: Incidencias — POST /incidents](../../../../architecture/incidencias/apis/API-001-incidencias.md#post-incidents) · [GET /incidents/{id}](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents-id)
- **Flujo:** [FL-001: Registro de una incidencia](../../../../architecture/incidencias/flows/FL-001-registro-incidencia.md)
- **Requerimiento:** [PRD-001: HelpDesk Lite](../../requirements/PRD-001-helpdesk-lite/README.md) — RF-01, RNF-06.
- **Contrato:** [ADR-008: contrato OpenAPI como fuente compartida](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md).

## Observaciones

- AC-003 (persistencia y recuperación por id) no tiene interfaz en esta historia: se verifica por API.
- Se resuelven al planificar las tareas técnicas (no condicionan el comportamiento): nombres de tabla y columnas, representación de `createdAt` en SQLite y librería de acceso a datos.
- El estilo de `status` (`Open|In Progress|Resolved`) se normaliza, si hace falta, antes de RF-05 (US de ciclo de vida).
