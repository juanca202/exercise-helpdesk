---
id: US-006
type: user-story
status: Ready
created: 2026-10-12
updated: 2026-10-12
repositories: [helpdesk-backend, helpdesk-frontend]
invest: { met: 5, partial: 1, not_met: 0, total: 6 }
dor: { met: 6, partial: 0, not_met: 0, total: 6 }
requirement: ../../requirements/PRD-001-helpdesk-lite/README.md
---
# US-006: Consultar el dashboard de indicadores

## Descripción

**COMO** responsable de soporte
**QUIERO** consultar un dashboard con indicadores básicos de las incidencias registradas
**PARA** conocer la distribución de las incidencias por estado y la carga de trabajo actual

## Contexto

Origen: RF-06 y RNF-06 de [PRD-001: HelpDesk Lite](../../requirements/PRD-001-helpdesk-lite/README.md). El dashboard es la página principal de la aplicación y desde él se accede al listado y al registro. Los indicadores se calculan sobre los datos que generan [US-001](../US-001-registrar-incidencia/README.md) y [US-005](../US-005-cambiar-estado-incidencia/README.md). La aplicación es de demostración: no hay roles, cualquier actor puede consultarlo.

## Fuera de alcance

- Gráficos analíticos avanzados, series temporales y comparaciones históricas.
- Indicadores por categoría, prioridad, agente o antigüedad.
- Exportación de indicadores e informes.
- Actualización automática en tiempo real (los indicadores se actualizan al consultar).

## Reglas de negocio

- **BR-01:** Los indicadores DEBEN ser el total de incidencias y la cantidad de incidencias en cada estado (`Open`, `In Progress`, `Resolved`) → verificado por AC-002
- **BR-02:** El total DEBE ser igual a la suma de las incidencias de los tres estados → verificado por AC-003
- **BR-03:** El dashboard NO DEBE incluir gráficos analíticos avanzados ni comparaciones históricas → verificado por AC-007

## Criterios de aceptación

- **AC-001 (Casos de uso):** El sistema DEBE permitir consultar un dashboard con indicadores de las incidencias registradas.
- **AC-002 (Procesamiento de datos):** El dashboard DEBE presentar el total de incidencias y la cantidad de incidencias abiertas, en atención y resueltas.
- **AC-003 (Reglas de negocio):** Los indicadores DEBEN corresponder a los datos registrados en el sistema, de modo que el total sea igual a la suma de las incidencias abiertas, en atención y resueltas.
- **AC-004 (Procesamiento de datos):** Los indicadores DEBEN calcularse al consultarlos, de modo que reflejen los registros y los cambios de estado realizados antes de la consulta.
- **AC-005 (Salidas del sistema):** Cuando no existan incidencias, el dashboard DEBE mostrar los cuatro indicadores en cero y un mensaje comprensible que lo explique.
- **AC-006 (Interacción de usuario):** La interfaz DEBE presentar el dashboard como página principal de la aplicación, con un menú de navegación (Dashboard · Incidencias · Nueva incidencia) y acceso al listado y al registro.
- **AC-007 (Idoneidad funcional):** El dashboard NO DEBE incluir gráficos analíticos avanzados ni comparaciones históricas.
- **AC-008 (Usabilidad):** La interfaz DEBE mostrar un indicador de carga mientras consulta los indicadores y un mensaje de error comprensible, con opción de reintentar, si la consulta falla.
- **AC-009 (Integraciones):** El frontend DEBE consumir el endpoint de indicadores conforme al contrato OpenAPI del backend.

## Referencias

- **Diseño / prototipo:** Ninguno aportado por el usuario.
- **Wireframe:** [WF-001: Dashboard](wireframes/WF-001-dashboard.md) — Approved.
- **Requerimiento:** [PRD-001: HelpDesk Lite](../../requirements/PRD-001-helpdesk-lite/README.md) — RF-06, RNF-06.
- **Modelo:** [MD-003: Indicadores de incidencias](../../../../architecture/incidencias/models/MD-003-indicadores-incidencias.md) · [MD-001: Incidencia](../../../../architecture/incidencias/models/MD-001-incidencia.md)
- **API:** [API-001: Incidencias — GET /incidents/summary](../../../../architecture/incidencias/apis/API-001-incidencias.md#get-incidents-summary)
- **Contrato:** [ADR-008: contrato OpenAPI como fuente compartida](../../../../../helpdesk-backend/docs/adr/ADR-008-contrato-openapi-fuente-compartida.md).

## Observaciones

- Depende de [US-001](../US-001-registrar-incidencia/README.md) y de [US-005](../US-005-cambiar-estado-incidencia/README.md) para que los indicadores varíen por estado; los accesos al listado y al registro enlazan con [US-002](../US-002-listar-filtrar-incidencias/README.md) y US-001.
- Wireframe y contrato del endpoint de indicadores generados con `/design-define`. Las tarjetas son solo informativas.
