# RQ-001: HelpDesk Lite - PRD

<!-- rq:status=Ready -->
**Estado:** Ready
**Fecha de creación:** 2026-10-12
**Última actualización:** 2026-10-12

### 1. Resumen ejecutivo

HelpDesk Lite es una aplicación web para gestionar incidencias de soporte interno de una organización. Permite registrar solicitudes, consultar y filtrar incidencias, gestionar su ciclo de vida y visualizar indicadores básicos de atención.

### 2. Problema y oportunidad

Las solicitudes internas de soporte requieren un mecanismo centralizado que permita registrar incidencias, consultar su estado y realizar seguimiento de su resolución.

Se plantea una solución de alcance reducido que cubra el ciclo básico de una incidencia: registro, consulta, seguimiento y visualización de la carga de trabajo.

### 3. Objetivos del producto

- Permitir el registro de incidencias de soporte.
- Facilitar la consulta y el filtrado de incidencias.
- Permitir el seguimiento de incidencias mediante comentarios y cambios de estado.
- Proporcionar una vista resumida de la carga de trabajo y del estado de las incidencias.

### 4. Alcance

#### 4.1 Incluido

- Registro de incidencias.
- Consulta, filtrado y ordenamiento de incidencias.
- Visualización del detalle de una incidencia.
- Agregado de comentarios.
- Actualización del estado de una incidencia mediante transiciones permitidas.
- Dashboard con métricas básicas.
- Validaciones funcionales y mensajes de error comprensibles.

#### 4.2 Fuera de alcance

- Autenticación real, registro de usuarios y recuperación de contraseñas.
- Sistema completo de autorización basado en roles.
- Notificaciones por correo, SMS o mensajería.
- Archivos adjuntos.
- SLA, escalamiento automático y asignación automática de agentes.
- Integración con sistemas externos de gestión de servicios.
- Aplicación móvil nativa.
- Analítica histórica avanzada y generación de informes exportables.

Para las demostraciones se podrán utilizar usuarios y roles predefinidos, sin implementar autenticación real.

### 5. Usuarios y actores

#### 5.1 Empleado solicitante

Persona que registra una incidencia y consulta su información.

#### 5.2 Agente de soporte

Persona que consulta incidencias, revisa sus detalles, agrega comentarios y actualiza sus estados.

#### 5.3 Responsable de soporte

Persona que consulta los indicadores generales para conocer la distribución de las incidencias.

Estos actores representan perspectivas funcionales. No implican un sistema real de identidad o permisos.

### 6. Requisitos funcionales

#### RF-01. Registrar una incidencia

El sistema debe permitir registrar una incidencia con los siguientes datos:

- Título.
- Descripción.
- Categoría.
- Prioridad.

Al registrar una incidencia, el sistema debe asignar un identificador único, una fecha de creación y un estado inicial.

El sistema debe validar los campos obligatorios y rechazar solicitudes inválidas con mensajes comprensibles.

**Resultado esperado:** la incidencia queda registrada y puede consultarse posteriormente.

#### RF-02. Consultar y filtrar incidencias

El sistema debe presentar un listado de incidencias registradas.

El usuario debe poder:

- Consultar el identificador, título, categoría, prioridad, estado y fecha de creación.
- Filtrar por estado, prioridad y categoría.
- Ordenar por fecha de creación.
- Consultar resultados vacíos de manera comprensible.

El listado debe ofrecer paginación básica.

**Resultado esperado:** el usuario puede localizar las incidencias relevantes utilizando los filtros disponibles.

#### RF-03. Consultar el detalle de una incidencia

El sistema debe permitir consultar una incidencia individual a partir de su identificador.

El detalle debe incluir sus datos principales, estado actual, fechas relevantes y comentarios asociados.

Si el identificador no corresponde a una incidencia existente, el sistema debe comunicar que no fue encontrada.

#### RF-04. Agregar comentarios

El sistema debe permitir agregar comentarios a una incidencia existente.

Cada comentario debe contener, como mínimo:

- Identificador único.
- Contenido.
- Fecha de creación.
- Autor o usuario de demostración.

Los comentarios deben conservarse y mostrarse en orden cronológico.

No se contempla la edición ni eliminación de comentarios en esta versión.

#### RF-05. Cambiar el estado de una incidencia

El sistema debe permitir actualizar el estado de una incidencia únicamente mediante transiciones válidas.

Para la versión inicial se proponen los siguientes estados:

- Open: incidencia registrada y pendiente de atención.
- In Progress: incidencia en atención.
- Resolved: incidencia resuelta.

Transiciones permitidas:

- Open → In Progress.
- In Progress → Resolved.
- Resolved → In Progress, para reabrir una incidencia que requiera atención adicional.

Las transiciones no contempladas deben rechazarse sin modificar el estado de la incidencia.

El sistema debe reflejar el nuevo estado en el detalle y en el listado.

#### RF-06. Consultar el dashboard

El sistema debe presentar indicadores básicos de las incidencias registradas:

- Total de incidencias.
- Cantidad de incidencias abiertas.
- Cantidad de incidencias en atención.
- Cantidad de incidencias resueltas.

Los indicadores deben corresponder a los datos disponibles en el sistema y actualizarse después de las operaciones que modifiquen esos datos.

El dashboard no contempla gráficos analíticos avanzados ni comparaciones históricas.

### 7. Requisitos no funcionales

#### RNF-06. Usabilidad

La interfaz debe presentar formularios comprensibles, validaciones visibles, indicadores de carga, mensajes de error y estados vacíos.

### 8. Criterios de aceptación del producto

El producto se considera funcionalmente aceptable cuando cumple los siguientes criterios:

| Código | Criterio | Origen |
| ------ | -------- | ------ |
| AC-001 | El sistema DEBE registrar una incidencia cuando los datos obligatorios son válidos. | RF-01 |
| AC-002 | El sistema DEBE rechazar con un mensaje comprensible el registro de una incidencia con datos obligatorios ausentes o inválidos. | RF-01 |
| AC-003 | El sistema DEBE permitir filtrar el listado por estado, prioridad y categoría, ordenarlo por fecha de creación y paginarlo. | RF-02 |
| AC-004 | El sistema DEBE mostrar el detalle de una incidencia existente, con sus comentarios, y comunicar que no fue encontrada cuando el identificador no existe. | RF-03 |
| AC-005 | El sistema DEBE permitir agregar comentarios a una incidencia y mostrarlos en orden cronológico. | RF-04 |
| AC-006 | El sistema DEBE aplicar las transiciones de estado válidas (Open → In Progress, In Progress → Resolved, Resolved → In Progress). | RF-05 |
| AC-007 | El sistema NO DEBE modificar el estado de una incidencia cuando la transición solicitada es inválida. | RF-05 |
| AC-008 | El dashboard DEBE presentar indicadores coherentes con las incidencias registradas y sus estados actuales. | RF-06 |

### 9. Restricciones y supuestos

- El producto es una aplicación de demostración, no un sistema de soporte listo para producción.
- Los datos de prueba podrán ser sintéticos.
- No se requiere autenticación real.
