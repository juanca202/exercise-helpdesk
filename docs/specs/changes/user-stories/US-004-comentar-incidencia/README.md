# US-004: Agregar comentarios a una incidencia

<!-- us:status=Ready -->
**Estado:** Ready
**Fecha de creación:** 2026-10-12
**Última actualización:** 2026-10-12
**Repositorios:** helpdesk-backend, helpdesk-frontend
**INVEST:** 🟢 5 · 🟡 1 / 6
**DoR:** 🟢 6 / 6
**Requerimiento:** [RQ-001: HelpDesk Lite](../../requirements/RQ-001-helpdesk-lite/README.md)

## Descripción

**COMO** agente de soporte
**QUIERO** agregar comentarios a una incidencia existente y verlos en orden cronológico
**PARA** dejar constancia del seguimiento y compartir el contexto de la atención

## Contexto

Origen: RF-04 y RNF-06 de [RQ-001: HelpDesk Lite](../../requirements/RQ-001-helpdesk-lite/README.md). Los comentarios se agregan y se leen desde la pantalla de detalle de [US-003](../US-003-detalle-incidencia/README.md). La aplicación es de demostración: no hay autenticación real, por lo que el autor es un usuario de demostración predefinido que asigna el sistema. No hay edición ni eliminación de comentarios en esta versión.

## Fuera de alcance

- Edición y eliminación de comentarios.
- Selección del autor por el usuario, autenticación y roles reales.
- Adjuntos, menciones, formato enriquecido y notificaciones.

## Reglas de negocio

- **BR-01:** Cada comentario DEBE tener un identificador único, un contenido, una fecha de creación y un autor; el identificador, la fecha y el autor los DEBE asignar el sistema y NO DEBEN ser proporcionados por el cliente → verificado por AC-002
- **BR-02:** El contenido DEBE ser obligatorio, recortarse (trim) antes de validarse y tener entre 1 y 500 caracteres; un valor compuesto solo por espacios DEBE considerarse vacío → verificado por AC-003
- **BR-03:** Los comentarios NO DEBEN poder editarse ni eliminarse → verificado por AC-007

## Criterios de aceptación

- **AC-001 (Casos de uso):** El sistema DEBE permitir agregar un comentario a una incidencia existente.
- **AC-002 (Reglas de negocio):** Al agregar un comentario válido, el sistema DEBE asignarle un identificador único, una fecha de creación y el usuario de demostración («Agente demo») como autor, y DEBE devolverlos en la respuesta; DEBE rechazar con un error 400 toda solicitud que incluya `id`, `createdAt` o `author`, sin persistir el comentario.
- **AC-003 (Reglas de negocio):** El sistema DEBE rechazar un comentario cuyo contenido esté ausente, vacío (tras recortar espacios) o supere los 500 caracteres.
- **AC-004 (Salidas del sistema):** Ante una solicitud inválida, el sistema DEBE responder con un error 400 que indique, por cada campo inválido, el campo y un motivo comprensible redactado en español, y NO DEBE persistir ningún comentario.
- **AC-005 (Reglas de negocio):** Si la incidencia no existe, el sistema DEBE responder con un error 404 y NO DEBE persistir ningún comentario.
- **AC-006 (Procesamiento de datos):** Los comentarios DEBEN conservarse y DEBEN mostrarse en el detalle de la incidencia en orden cronológico, del más antiguo al más reciente, con su contenido, autor y fecha de creación.
- **AC-007 (Reglas de negocio):** El sistema NO DEBE exponer operaciones para editar o eliminar comentarios.
- **AC-008 (Interacción de usuario):** El formulario de comentario DEBE mostrar un contador de caracteres y el error de validación junto al campo, y NO DEBE enviar la solicitud si el contenido es inválido; tras un registro exitoso, el comentario DEBE aparecer en la lista sin recargar la página y el campo DEBE quedar vacío.
- **AC-009 (Usabilidad):** El formulario DEBE mostrar un indicador de carga mientras se envía, bloquearse para evitar envíos duplicados y, si el registro falla, mostrar un mensaje de error comprensible y conservar el texto escrito.
- **AC-010 (Integraciones):** El frontend DEBE consumir el endpoint de comentarios conforme al contrato OpenAPI del backend y DEBE presentar los errores de validación devueltos por la API.

## Referencias

- **Diseño / prototipo:** Ninguno aportado por el usuario.
- **Wireframe:** [WF-001: Formulario de comentario](wireframes/WF-001-formulario-comentario.md) — Approved.
- **Requerimiento:** [RQ-001: HelpDesk Lite](../../requirements/RQ-001-helpdesk-lite/README.md) — RF-04, RNF-06.
- **Modelo:** [MD-002: Comentario](../../../../architecture/incidencias/models/MD-002-comentario.md) · [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md)
- **API:** [API-001: Incidencias — POST /incidents/{id}/comments](../../../../architecture/incidencias/apis/API-001-incidencias.md#post-incidents-id-comments)
- **Flujo:** [FL-004: Registro de un comentario](../../../../architecture/incidencias/flows/FL-004-registro-comentario.md)
- **Contrato:** [ADR-008: contrato OpenAPI como fuente compartida](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md).

## Observaciones

- Depende de [US-003](../US-003-detalle-incidencia/README.md) (pantalla y respuesta de detalle donde se muestran los comentarios) y, por extensión, de [US-001](../US-001-registrar-incidencia/README.md).
- Modelo, contrato y wireframe generados con `/design-define`; el autor visible es «Agente demo». La cabecera muestra «Empleado demo»: en pantalla aparecen dos usuarios de demostración distintos, aceptado al aprobar el wireframe.

---

## Validación

### Complejidad sugerida

- **Story points:** 3
- **Justificación:** Un endpoint de escritura con nueva entidad persistida y un formulario sencillo embebido en una pantalla existente.

### INVEST

| Letra | Criterio      | Resultado | Notas |
| ----- | ------------- | --------- | ----- |
| **I** | Independiente | Partial   | Depende de la pantalla y la respuesta de detalle de US-003. |
| **N** | Negociable    | Met       | Límite de longitud y disposición ajustables. |
| **V** | Valiosa       | Met       | Habilita el seguimiento de la atención. |
| **E** | Estimable     | Met       | Alcance acotado. |
| **S** | Pequeña       | Met       | Un endpoint y un formulario. |
| **T** | Testeable     | Met       | Verificable por pruebas de API, de componente y e2e. |

### Definition of Ready (DoR)

| Criterio DoR                       | Estado  | Notas |
| ---------------------------------- | ------- | ----- |
| Dependencias listas                | Met     | US-003 especificada; implementar después de ella. |
| Inputs/outputs claros              | Met     | Entradas y salidas definidas en los criterios. |
| Repositorios definidos             | Met     | helpdesk-backend, helpdesk-frontend. |
| Sin decisiones técnicas pendientes | Met     | Autor fijo y longitud 1–500 confirmados; contrato en API-001. |
| Referencias de UI                  | Met     | Wireframe aprobado. |
| Sin aclaraciones pendientes        | Met     | Nada pendiente con usuario/producto. |
