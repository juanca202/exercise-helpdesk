---
id: US-002
type: user-story
status: Ready
created: 2026-10-12
updated: 2026-10-12
repositories: [helpdesk-backend, helpdesk-frontend]
invest: { met: 5, partial: 1, not_met: 0, total: 6 }
dor: { met: 6, partial: 0, not_met: 0, total: 6 }
requirement: ../../requirements/PRD-001-helpdesk-lite/README.md
---
# US-002: Consultar y filtrar incidencias

## Descripción

**COMO** agente de soporte (o empleado solicitante)
**QUIERO** consultar el listado de incidencias registradas y filtrarlas por estado, prioridad y categoría, ordenarlas por fecha de creación y paginarlas
**PARA** localizar rápidamente las incidencias relevantes sin revisar todas las solicitudes

## Contexto

Origen: RF-02 y RNF-06 de [PRD-001: HelpDesk Lite](../../requirements/PRD-001-helpdesk-lite/README.md). Parte de las incidencias que registra [US-001](../US-001-registrar-incidencia/README.md). La aplicación es de demostración: no hay autenticación real y todos los actores ven el mismo listado. Las etiquetas visibles de categoría y prioridad son las de [WF-001](../US-001-registrar-incidencia/wireframes/WF-001-registro-incidencia.md); el estado se muestra con sus valores literales `Open`, `In Progress` y `Resolved`.

## Fuera de alcance

- Detalle de la incidencia, comentarios y cambio de estado (US-003, US-004, US-005).
- Búsqueda de texto libre, filtros por fecha y ordenamiento por otros campos.
- Tamaño de página configurable por el usuario.
- Exportación del listado.

## Reglas de negocio

- **BR-01:** Los filtros por estado, prioridad y categoría DEBEN poder combinarse; una incidencia aparece en el resultado solo si cumple todos los filtros aplicados → verificado por AC-003
- **BR-02:** El listado DEBE ordenarse por fecha de creación descendente (más recientes primero) por defecto y DEBE poder invertirse → verificado por AC-004
- **BR-03:** El listado DEBE paginarse con un tamaño de página fijo de 20 incidencias → verificado por AC-005

## Criterios de aceptación

- **AC-001 (Casos de uso):** El sistema DEBE permitir consultar un listado de las incidencias registradas.
- **AC-002 (Salidas del sistema):** Cada incidencia del listado DEBE mostrar su identificador, título, categoría, prioridad, estado y fecha de creación, y DEBE permitir navegar a su detalle; la interfaz PUEDE mostrar el identificador abreviado (8 primeros caracteres) en el listado.
- **AC-003 (Reglas de negocio):** El sistema DEBE permitir filtrar el listado por estado, por prioridad y por categoría, de forma individual o combinada, con un solo valor por filtro, y sin filtros DEBE devolver todas las incidencias.
- **AC-004 (Reglas de negocio):** El sistema DEBE ordenar el listado por fecha de creación, descendente por defecto, y DEBE permitir ordenarlo de forma ascendente.
- **AC-005 (Procesamiento de datos):** El sistema DEBE paginar el listado en páginas de 20 incidencias e informar el total de resultados, la página actual y el número de páginas; una página fuera de rango DEBE devolver una lista vacía sin error.
- **AC-006 (Salidas del sistema):** Ante un valor de filtro, de orden o de paginación inválido, el sistema DEBE responder con un error 400 que indique, por cada parámetro inválido, el parámetro y un motivo comprensible redactado en español.
- **AC-007 (Interacción de usuario):** Al cambiar un filtro o el orden, la interfaz DEBE volver a la primera página y DEBE mostrar los filtros aplicados y DEBE ofrecer una acción para limpiar los filtros; los filtros, el orden y la página DEBEN conservarse en la dirección de modo que recargar o volver atrás mantenga la vista.
- **AC-008 (Usabilidad):** Cuando no haya resultados, la interfaz DEBE mostrar un estado vacío comprensible que distinga entre «no hay incidencias registradas» y «ninguna incidencia coincide con los filtros», este último con la acción de limpiar filtros.
- **AC-009 (Usabilidad):** La interfaz DEBE mostrar un indicador de carga mientras consulta el listado y un mensaje de error comprensible, con opción de reintentar, si la consulta falla.
- **AC-010 (Integraciones):** El frontend DEBE consumir el endpoint de listado conforme al contrato OpenAPI del backend y DEBE presentar los errores de validación devueltos por la API.

## Referencias

- **Diseño / prototipo:** Ninguno aportado por el usuario.
- **Wireframe:** [WF-001: Listado de incidencias](wireframes/WF-001-listado-incidencias.md) — Approved.
- **Requerimiento:** [PRD-001: HelpDesk Lite](../../requirements/PRD-001-helpdesk-lite/README.md) — RF-02, RNF-06.
- **Modelo:** [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md)
- **API:** [API-001: Incidencias — GET /incidents](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents)
- **Flujo:** [FL-002: Consulta del listado de incidencias](../../../../architecture/incidencias/flows/FL-002-consulta-listado-incidencias.md)
- **Contrato:** [ADR-008: contrato OpenAPI como fuente compartida](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md).

## Observaciones

- Depende de [US-001](../US-001-registrar-incidencia/README.md) (las incidencias que se listan). Navegar al detalle (AC-002) se completa con [US-003](../US-003-detalle-incidencia/README.md).
- Wireframe aprobado y documentación técnica generados con `/design-define`. Parámetros de query no declarados: se ignoran (decidir en la planificación si se rechazan con 400).
