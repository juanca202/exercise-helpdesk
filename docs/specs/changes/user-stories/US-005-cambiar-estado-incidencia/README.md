# US-005: Cambiar el estado de una incidencia

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
**QUIERO** actualizar el estado de una incidencia mediante las transiciones permitidas
**PARA** reflejar el avance de su atención y que el estado actual sea visible en el detalle y en el listado

## Contexto

Origen: RF-05 y RNF-06 de [RQ-001: HelpDesk Lite](../../requirements/RQ-001-helpdesk-lite/README.md). La acción se ejecuta desde la pantalla de detalle de [US-003](../US-003-detalle-incidencia/README.md) y el nuevo estado se refleja también en el listado de [US-002](../US-002-listar-filtrar-incidencias/README.md). Los valores de estado se mantienen literales: `Open`, `In Progress` y `Resolved`. La aplicación es de demostración: no hay roles ni permisos, cualquier actor puede ejecutar las transiciones.

## Fuera de alcance

- Historial o auditoría de cambios de estado y fechas de resolución.
- Estados adicionales y asignación de agentes.
- Restricción de transiciones por rol.
- Confirmación previa al aplicar una transición.

## Reglas de negocio

- **BR-01:** Una incidencia solo DEBE cambiar de estado mediante las transiciones `Open → In Progress`, `In Progress → Resolved` y `Resolved → In Progress` (reabrir) → verificado por AC-002
- **BR-02:** Una transición no contemplada, incluida la que mantiene el mismo estado, DEBE rechazarse y NO DEBE modificar el estado de la incidencia → verificado por AC-003

## Criterios de aceptación

- **AC-001 (Casos de uso):** El sistema DEBE permitir actualizar el estado de una incidencia existente.
- **AC-002 (Reglas de negocio):** El sistema DEBE aplicar únicamente las transiciones `Open → In Progress`, `In Progress → Resolved` y `Resolved → In Progress`.
- **AC-003 (Reglas de negocio):** El sistema DEBE rechazar cualquier otra transición con un error que indique, en español, que la transición no está permitida desde el estado actual, y NO DEBE modificar el estado de la incidencia.
- **AC-004 (Reglas de negocio):** El sistema DEBE rechazar con un error 400, indicando el campo y el motivo en español, un estado ausente o que no pertenezca a los valores permitidos, y con un error 404 una incidencia inexistente, sin modificar ningún dato.
- **AC-005 (Salidas del sistema):** Tras una transición válida, el sistema DEBE devolver la incidencia actualizada con su nuevo estado.
- **AC-006 (Procesamiento de datos):** El nuevo estado DEBE quedar persistido y DEBE reflejarse en el detalle y en el listado de incidencias.
- **AC-007 (Interacción de usuario):** El detalle DEBE ofrecer únicamente las acciones correspondientes a las transiciones válidas desde el estado actual (iniciar atención, resolver o reabrir), sin solicitar confirmación, y tras aplicarla DEBE mostrar el nuevo estado y un aviso temporal de éxito.
- **AC-008 (Usabilidad):** La interfaz DEBE mostrar un indicador de carga mientras aplica la transición, bloquear las acciones para evitar envíos duplicados y, si falla, mostrar un mensaje de error comprensible y conservar el estado anterior; si la transición es rechazada por el estado actual, DEBE ofrecer recargar la incidencia.
- **AC-009 (Integraciones):** El frontend DEBE consumir el endpoint de cambio de estado conforme al contrato OpenAPI del backend y DEBE presentar el error devuelto por la API cuando una transición sea rechazada.

## Referencias

- **Diseño / prototipo:** Ninguno aportado por el usuario.
- **Wireframe:** [WF-001: Acciones de estado](wireframes/WF-001-acciones-estado.md) — Approved.
- **Requerimiento:** [RQ-001: HelpDesk Lite](../../requirements/RQ-001-helpdesk-lite/README.md) — RF-05, RNF-06.
- **Modelo:** [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md)
- **API:** [API-001: Incidencias — PATCH /incidents/{id}/status](../../../../architecture/incidencias/apis/API-001-incidencias.md#patch-incidents-id-status)
- **Flujo:** [FL-005: Cambio de estado de una incidencia](../../../../architecture/incidencias/flows/FL-005-cambio-estado-incidencia.md)
- **Diagrama:** [DG-001: Estados de la incidencia](../../../../architecture/incidencias/diagrams/DG-001-estados-incidencia.md)
- **Contrato:** [ADR-008: contrato OpenAPI como fuente compartida](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md).

## Observaciones

- Depende de [US-003](../US-003-detalle-incidencia/README.md) (pantalla de detalle donde se ofrecen las acciones); el reflejo en el listado requiere [US-002](../US-002-listar-filtrar-incidencias/README.md).
- La decisión de mantener los valores literales de `status` cierra el pendiente de normalización de la capability [incidencias](../../../../architecture/incidencias/README.md).
- Contrato, diagrama de estados (DG-001) y wireframe generados con `/design-define`. La transición no permitida responde `409` (decisión técnica documentada en API-001).

---

## Validación

### Complejidad sugerida

- **Story points:** 3
- **Justificación:** Un endpoint con una máquina de estados pequeña y acciones condicionales en la pantalla de detalle.

### INVEST

| Letra | Criterio      | Resultado | Notas |
| ----- | ------------- | --------- | ----- |
| **I** | Independiente | Partial   | Depende de la pantalla de detalle de US-003. |
| **N** | Negociable    | Met       | Etiquetas y disposición de las acciones ajustables. |
| **V** | Valiosa       | Met       | Refleja el avance de la atención. |
| **E** | Estimable     | Met       | Transiciones cerradas y alcance acotado. |
| **S** | Pequeña       | Met       | Un endpoint y unas acciones en una pantalla existente. |
| **T** | Testeable     | Met       | Verificable por pruebas de API, de componente y e2e. |

### Definition of Ready (DoR)

| Criterio DoR                       | Estado  | Notas |
| ---------------------------------- | ------- | ----- |
| Dependencias listas                | Met     | US-003 especificada; implementar después de ella. |
| Inputs/outputs claros              | Met     | Transiciones y errores definidos en los criterios. |
| Repositorios definidos             | Met     | helpdesk-backend, helpdesk-frontend. |
| Sin decisiones técnicas pendientes | Met     | Valores de `status` literales y sin confirmación (confirmado); contrato en API-001. |
| Referencias de UI                  | Met     | Wireframe aprobado. |
| Sin aclaraciones pendientes        | Met     | Nada pendiente con usuario/producto. |
