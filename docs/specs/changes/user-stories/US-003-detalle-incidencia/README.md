# US-003: Consultar el detalle de una incidencia

<!-- us:status=Ready -->
**Estado:** Ready
**Fecha de creación:** 2026-10-12
**Última actualización:** 2026-10-13
**Repositorios:** helpdesk-backend, helpdesk-frontend
**INVEST:** 🟢 6 / 6
**DoR:** 🟢 6 / 6
**Requerimiento:** [RQ-001: HelpDesk Lite](../../requirements/RQ-001-helpdesk-lite/README.md)

## Descripción

**COMO** agente de soporte (o empleado solicitante)
**QUIERO** consultar el detalle de una incidencia a partir de su identificador, con su estado, sus fechas y sus comentarios
**PARA** revisar toda la información de la solicitud antes de atenderla o darle seguimiento

## Contexto

Origen: RF-03 y RNF-06 de [RQ-001: HelpDesk Lite](../../requirements/RQ-001-helpdesk-lite/README.md). Completa la operación `GET /incidents/{id}` que [US-001](../US-001-registrar-incidencia/README.md) ya expone para recuperar la incidencia (AC-003 de esa historia): aquí se amplía con los comentarios y se cierra el comportamiento ante un id inexistente. Los comentarios los crea [US-004](../US-004-comentar-incidencia/README.md); mientras no existan, el detalle los muestra vacíos. La única fecha relevante del modelo hoy es la de creación.

## Fuera de alcance

- Agregar comentarios (US-004) y cambiar el estado (US-005); esta historia solo los muestra.
- Edición o eliminación de la incidencia.
- Historial de cambios de estado y fechas de actualización o resolución.

## Reglas de negocio

- **BR-01:** Un identificador que no corresponda a una incidencia existente, incluido uno que no sea un UUID v4 válido, DEBE tratarse como «no encontrada» y NO DEBE distinguirse por el tipo de error → verificado por AC-004
- **BR-02:** Los comentarios de la incidencia DEBEN presentarse en orden cronológico, del más antiguo al más reciente → verificado por AC-003

## Criterios de aceptación

- **AC-001 (Casos de uso):** El sistema DEBE permitir consultar una incidencia individual a partir de su identificador.
- **AC-002 (Salidas del sistema):** El detalle DEBE incluir el identificador, título, descripción, categoría, prioridad, estado actual, fecha de creación y los comentarios asociados.
- **AC-003 (Procesamiento de datos):** El detalle DEBE devolver los comentarios en orden cronológico ascendente y una lista vacía cuando la incidencia no tenga comentarios.
- **AC-004 (Reglas de negocio):** Ante un identificador inexistente o que no sea un UUID v4 válido, el sistema DEBE responder con un error 404 que comunique que la incidencia no fue encontrada.
- **AC-005 (Interacción de usuario):** La interfaz DEBE mostrar el detalle en una dirección propia por incidencia, de la forma `/incidents/{id}`, de modo que pueda abrirse desde el listado o directamente por su identificador.
- **AC-006 (Salidas del sistema):** Si la incidencia no existe, la interfaz DEBE mostrar un mensaje comprensible de «incidencia no encontrada» con una acción para volver al listado.
- **AC-007 (Usabilidad):** La interfaz DEBE mostrar un indicador de carga mientras consulta el detalle, un estado vacío comprensible cuando no haya comentarios y un mensaje de error comprensible, con opción de reintentar, si la consulta falla.
- **AC-008 (Integraciones):** El frontend DEBE consumir el endpoint de detalle conforme al contrato OpenAPI del backend.

## Referencias

- **Diseño / prototipo:** Ninguno aportado por el usuario.
- **Wireframe:** [WF-001: Detalle de incidencia](wireframes/WF-001-detalle-incidencia.md) — Approved.
- **Requerimiento:** [RQ-001: HelpDesk Lite](../../requirements/RQ-001-helpdesk-lite/README.md) — RF-03, RNF-06.
- **Modelo:** [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md) · [MD-002: Comentario](../../../../architecture/incidencias/models/MD-002-comentario.md)
- **API:** [API-001: Incidencias — GET /incidents/{id}](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents-id)
- **Flujo:** [FL-003: Consulta del detalle de una incidencia](../../../../architecture/incidencias/flows/FL-003-consulta-detalle-incidencia.md)
- **Contrato:** [ADR-008: contrato OpenAPI como fuente compartida](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md).

## Observaciones

- Depende de [US-001](../US-001-registrar-incidencia/README.md). El acceso desde el listado lo aporta [US-002](../US-002-listar-filtrar-incidencias/README.md).
- Cierra los pendientes de `GET /incidents/{id}` registrados en Observaciones de la capability [incidencias](../../../../architecture/incidencias/README.md): forma de la respuesta con comentarios y cuerpo del `404`.
- Wireframe aprobado y documentación técnica actualizada con `/design-define`.

---

## Validación

### Complejidad sugerida

- **Story points:** 3
- **Justificación:** Extiende un endpoint existente y añade una pantalla de solo lectura con sus estados de carga, vacío, error y no encontrada.

### INVEST

| Letra | Criterio      | Resultado | Notas |
| ----- | ------------- | --------- | ----- |
| **I** | Independiente | Met       | Solo requiere el registro de US-001; los comentarios vacíos no bloquean. |
| **N** | Negociable    | Met       | Disposición de la pantalla ajustable. |
| **V** | Valiosa       | Met       | Permite revisar la solicitud completa. |
| **E** | Estimable     | Met       | Alcance acotado. |
| **S** | Pequeña       | Met       | Un endpoint ampliado y una pantalla. |
| **T** | Testeable     | Met       | Verificable por pruebas de API, de componente y e2e. |

### Definition of Ready (DoR)

| Criterio DoR                       | Estado  | Notas |
| ---------------------------------- | ------- | ----- |
| Dependencias listas                | Met     | US-001 especificada. |
| Inputs/outputs claros              | Met     | Entradas y salidas definidas en los criterios. |
| Repositorios definidos             | Met     | helpdesk-backend, helpdesk-frontend. |
| Sin decisiones técnicas pendientes | Met     | Id inválido tratado como 404 (confirmado); contrato en API-001. |
| Referencias de UI                  | Met     | Wireframe aprobado. |
| Sin aclaraciones pendientes        | Met     | Nada pendiente con usuario/producto. |
