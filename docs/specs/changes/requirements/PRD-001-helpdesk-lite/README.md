---
id: PRD-001
type: prd
status: Ready
created: 2026-10-12
updated: 2026-10-28
business_approval: "Aceptado por el equipo (autor del documento), 2026-10-28"
---

# PRD-001: HelpDesk Lite

## 1. Introducción

### 1.1 Problema y oportunidad

Las solicitudes internas de soporte requieren un mecanismo centralizado que permita registrar incidencias, consultar su estado y realizar seguimiento de su resolución. Se plantea una solución de alcance reducido que cubra el ciclo básico de una incidencia: registro, consulta, seguimiento y visualización de la carga de trabajo.

**Evidencia:** Sin medición disponible.

**Costo de no hacerlo:** Sin medición disponible; las solicitudes siguen sin un punto único de registro y seguimiento.

### 1.2 Alcance del producto

HelpDesk Lite es una aplicación web para gestionar incidencias de soporte interno de una organización. Permite registrar solicitudes, consultar y filtrar incidencias, gestionar su ciclo de vida y visualizar indicadores básicos de atención.

**Incluido:**

- Registro de incidencias.
- Consulta, filtrado y ordenamiento de incidencias.
- Visualización del detalle de una incidencia.
- Agregado de comentarios.
- Actualización del estado de una incidencia mediante transiciones permitidas.
- Dashboard con métricas básicas.
- Validaciones funcionales y mensajes de error comprensibles.

**Fuera de alcance:**

- Autenticación real, registro de usuarios y recuperación de contraseñas.
- Sistema completo de autorización basado en roles.
- Notificaciones por correo, SMS o mensajería.
- Archivos adjuntos.
- SLA, escalamiento automático y asignación automática de agentes.
- Integración con sistemas externos de gestión de servicios.
- Aplicación móvil nativa.
- Analítica histórica avanzada y generación de informes exportables.

### 1.3 Definiciones, acrónimos y abreviaturas

- **Estados de una incidencia:** «abierta» = `Open`, «en atención» = `In Progress`, «resuelta» = `Resolved`. Los indicadores del dashboard usan la forma en español y el resto del documento, los valores de dominio en inglés.

### 1.4 Objetivos y métricas de éxito

Objetivos tomados de la sección «Objetivos del producto» del [requerimiento original](references/raw-requirement.md). Las métricas son proxy; el negocio confirmó los objetivos el 2026-10-28.

| ID | Objetivo de negocio | Métrica | Valor objetivo | Plazo |
| -- | ------------------- | ------- | -------------- | ----- |
| OB-01 | Permitir el registro de incidencias de soporte. | Proxy: flujo de registro completable de extremo a extremo (AC-001) | Por definir (proxy) | Por definir |
| OB-02 | Facilitar la consulta y el filtrado de incidencias. | Proxy: filtros por estado, prioridad y categoría disponibles (AC-003) | Por definir (proxy) | Por definir |
| OB-03 | Permitir el seguimiento mediante comentarios y cambios de estado. | Proxy: comentarios y transiciones válidas operativas (AC-005, AC-006) | Por definir (proxy) | Por definir |
| OB-04 | Proporcionar una vista resumida de la carga de trabajo y del estado de las incidencias. | Proxy: indicadores coherentes con los datos (AC-008) | Por definir (proxy) | Por definir |

## 2. Descripción general del producto

### 2.1 Perspectiva del producto

Producto nuevo e independiente. Es una aplicación de demostración, no un sistema de soporte listo para producción.

### 2.2 Funciones del producto

Registro de incidencias, listado con filtros y paginación, detalle con comentarios, cambio de estado mediante transiciones válidas y dashboard de indicadores básicos.

### 2.3 Restricciones del producto

- El producto es una aplicación de demostración, no un sistema de soporte listo para producción.
- Los datos de prueba podrán ser sintéticos.
- No se requiere autenticación real. Para las demostraciones se podrán utilizar usuarios y roles predefinidos.

### 2.4 Características de los usuarios y escenarios de uso

Estos actores representan perspectivas funcionales. No implican un sistema real de identidad o permisos.

- **Empleado solicitante:** persona que registra una incidencia y consulta su información.
- **Agente de soporte:** persona que consulta incidencias, revisa sus detalles, agrega comentarios y actualiza sus estados.
- **Responsable de soporte:** persona que consulta los indicadores generales para conocer la distribución de las incidencias.

Todos los actores pueden ver y operar todas las incidencias; no hay restricciones por actor en esta versión.

Escenarios derivados de la descripción de cada actor; confirmados por el negocio el 2026-10-28.

| ID | Actor | Escenario | Resultado esperado | FR-XXX relacionados |
| -- | ----- | --------- | ------------------ | ------------------- |
| SC-01 | Empleado solicitante | Registra una incidencia con título, descripción, categoría y prioridad, y luego consulta su información. | La incidencia queda registrada y puede consultarse posteriormente. | RF-01, RF-03 |
| SC-02 | Agente de soporte | Localiza incidencias con los filtros, revisa su detalle, agrega un comentario y avanza su estado o lo reabre (Resolved → In Progress). | El comentario queda visible en orden cronológico y el nuevo estado se refleja en el detalle y en el listado. | RF-02, RF-03, RF-04, RF-05 |
| SC-03 | Responsable de soporte | Consulta los indicadores generales. | Conoce el total de incidencias y su distribución por estado, calculada al abrir o recargar el dashboard. | RF-06 |

### 2.5 Supuestos y dependencias

**Funcionalidad transversal asumida**

| Capacidad | Supuesto | Origen |
| --------- | -------- | ------ |
| Autenticación y roles | No existe ni está en el alcance; se usan usuarios de demostración predefinidos | Fuera de alcance (sección 1.2) |

## 3. Stack tecnológico

Pendiente — se completa al promover a SRS.

## 4. Repositorios e implementación

Pendiente — se completa al promover a SRS.

## 5. Requisitos funcionales

Los ids `RF-XX` y `RNF-XX` se conservan tal como los citan las historias de usuario derivadas.

| ID | Categoría | Prioridad | Estado | Enunciado |
| -- | --------- | --------- | ------ | --------- |
| RF-01 | Casos de uso | Esencial | Approved | El sistema DEBE permitir registrar una incidencia con título, descripción, categoría y prioridad, asignándole un identificador único, una fecha de creación y el estado inicial `Open`. El título DEBE tener entre 5 y 100 caracteres y la descripción entre 10 y 2000; la categoría DEBE ser una de Hardware, Software, Accesos y permisos, Red y conectividad, Correo y comunicación u Otro, y la prioridad una de Baja, Media o Alta. El sistema DEBE validar los campos obligatorios y rechazar las solicitudes inválidas con mensajes comprensibles. |
| RF-02 | Casos de uso | Esencial | Approved | El sistema DEBE presentar un listado de incidencias con identificador, título, categoría, prioridad, estado y fecha de creación; DEBE permitir filtrar por estado, prioridad y categoría, ordenar por fecha de creación (por defecto, las más recientes primero) y paginar en páginas de 20 incidencias, y DEBE comunicar de manera comprensible un resultado vacío. |
| RF-03 | Casos de uso | Esencial | Approved | El sistema DEBE permitir consultar una incidencia individual por su identificador, con sus datos principales, estado actual, fechas relevantes y comentarios asociados, y DEBE comunicar que no fue encontrada cuando el identificador no existe. |
| RF-04 | Casos de uso | Alta | Approved | El sistema DEBE permitir agregar comentarios a una incidencia existente, cada uno con identificador único, contenido (entre 1 y 500 caracteres), fecha de creación y autor o usuario de demostración, y DEBE mostrarlos en orden cronológico. El sistema NO DEBE permitir editar ni eliminar comentarios en esta versión. |
| RF-05 | Flujos de proceso | Esencial | Approved | El sistema DEBE permitir cambiar el estado de una incidencia únicamente mediante las transiciones Open → In Progress, In Progress → Resolved y Resolved → In Progress; DEBE rechazar cualquier otra sin modificar el estado, y DEBE reflejar el nuevo estado en el detalle y en el listado. |
| RF-06 | Salidas del sistema | Alta | Approved | El sistema DEBE presentar el total de incidencias y las cantidades abiertas, en atención y resueltas, correspondientes a los datos disponibles; los indicadores DEBEN calcularse al abrir o recargar la página, sin actualización en tiempo real. El dashboard NO DEBE incluir gráficos analíticos avanzados ni comparaciones históricas. |

Estados de una incidencia: `Open` (registrada y pendiente de atención), `In Progress` (en atención) y `Resolved` (resuelta).

## 6. Requisitos no funcionales (expectativas de calidad)

| ID | Categoría | Prioridad | Estado | Expectativa |
| -- | --------- | --------- | ------ | ----------- |
| RNF-06 | Usabilidad | Alta | Proposed | La interfaz DEBE presentar formularios comprensibles, validaciones visibles, indicadores de carga, mensajes de error y estados vacíos. |

> Revisado con el negocio el 2026-10-28: al ser una aplicación de demostración, no se declaran expectativas de Seguridad, Rendimiento ni Fiabilidad/Disponibilidad.

## 7. Reglas de negocio

- **BR-01:** Las transiciones de estado válidas DEBEN ser únicamente Open → In Progress, In Progress → Resolved y Resolved → In Progress → relacionado con RF-05.
- **BR-02:** Los comentarios NO DEBEN poder editarse ni eliminarse en esta versión → relacionado con RF-04.

## 8. Interfaces externas

### 8.1 Interfaces de usuario

- Aplicación web con listado, registro, detalle (con comentarios y acciones de estado) y dashboard. Formularios comprensibles, validaciones visibles, indicadores de carga, mensajes de error y estados vacíos (RNF-06).

### 8.2 Interfaces de hardware

Pendiente — se completa al promover a SRS.

### 8.3 Interfaces de software

Pendiente — se completa al promover a SRS.

### 8.4 Interfaces de comunicaciones

Pendiente — se completa al promover a SRS.

## 9. Requisitos de datos (obligaciones de negocio)

No aplica: sin obligaciones de negocio sobre retención, sensibilidad o auditoría de datos identificadas. Los datos de prueba podrán ser sintéticos.

Entidades, volumen y modelo: Pendiente — se completa al promover a SRS.

## 10. Cumplimiento normativo

No aplica: no se identificó normativa aplicable.

Cómo se cumple cada una: Pendiente — se completa al promover a SRS.

## 11. Diseño de interfaz (wireframes)

<!-- srs:ui-required=true -->
**¿Requiere diseño de interfaz?** Sí

**Tipo de solución:** Aplicación web

**¿Debe ser responsiva?** Sí (según los wireframes aprobados de cada historia)

Los wireframes se generaron por historia de usuario y viven en la carpeta de cada una:

| Pantalla | Wireframe | Estado de revisión | Observaciones ya incorporadas |
| -------- | --------- | ------------------ | ----------------------------- |
| Registro de incidencia | [WF-001](../../user-stories/US-001-registrar-incidencia/wireframes/WF-001-registro-incidencia.md) | Approved | Ninguna |
| Listado de incidencias | [WF-001](../../user-stories/US-002-listar-filtrar-incidencias/wireframes/WF-001-listado-incidencias.md) | Approved | Ninguna |
| Detalle de incidencia | [WF-001](../../user-stories/US-003-detalle-incidencia/wireframes/WF-001-detalle-incidencia.md) | Approved | Ninguna |
| Formulario de comentario | [WF-001](../../user-stories/US-004-comentar-incidencia/wireframes/WF-001-formulario-comentario.md) | Approved | Ninguna |
| Acciones de estado | [WF-001](../../user-stories/US-005-cambiar-estado-incidencia/wireframes/WF-001-acciones-estado.md) | Approved | Ninguna |
| Dashboard | [WF-001](../../user-stories/US-006-dashboard-indicadores/wireframes/WF-001-dashboard.md) | Approved | Ninguna |

## 12. Criterios de aceptación y trazabilidad

| ID | AC | Origen | Depende de | Método de verificación | Criterio de aceptación |
| -- | -- | ------ | ---------- | ---------------------- | ---------------------- |
| RF-01 | AC-001 | [raw-requirement.md](references/raw-requirement.md) | Ninguna | Pendiente | El sistema DEBE registrar una incidencia cuando los datos obligatorios son válidos. |
| RF-01 | AC-002 | [raw-requirement.md](references/raw-requirement.md) | Ninguna | Pendiente | El sistema DEBE rechazar con un mensaje comprensible el registro de una incidencia con datos obligatorios ausentes o inválidos. |
| RF-02 | AC-003 | [raw-requirement.md](references/raw-requirement.md) | RF-01 | Pendiente | El sistema DEBE permitir filtrar el listado por estado, prioridad y categoría, ordenarlo por fecha de creación y paginarlo. |
| RF-03 | AC-004 | [raw-requirement.md](references/raw-requirement.md) | RF-01 | Pendiente | El sistema DEBE mostrar el detalle de una incidencia existente, con sus comentarios, y comunicar que no fue encontrada cuando el identificador no existe. |
| RF-04 | AC-005 | [raw-requirement.md](references/raw-requirement.md) | RF-03 | Pendiente | El sistema DEBE permitir agregar comentarios a una incidencia y mostrarlos en orden cronológico. |
| RF-05 | AC-006 | [raw-requirement.md](references/raw-requirement.md) | RF-03 | Pendiente | El sistema DEBE aplicar las transiciones de estado válidas (Open → In Progress, In Progress → Resolved, Resolved → In Progress). |
| RF-05 | AC-007 | [raw-requirement.md](references/raw-requirement.md) | RF-03 | Pendiente | El sistema NO DEBE modificar el estado de una incidencia cuando la transición solicitada es inválida. |
| RF-06 | AC-008 | [raw-requirement.md](references/raw-requirement.md) | RF-01, RF-05 | Pendiente | El dashboard DEBE presentar indicadores coherentes con las incidencias registradas y sus estados actuales. |
| RNF-06 | AC-009 | [raw-requirement.md](references/raw-requirement.md) (RNF-06) | Ninguna | Pendiente | La interfaz DEBE mostrar validaciones visibles, indicadores de carga, mensajes de error y estados vacíos en los formularios y listados. |
| RF-02 | AC-010 | [raw-requirement.md](references/raw-requirement.md) | RF-01 | Pendiente | El sistema DEBE comunicar de manera comprensible que no hay incidencias que mostrar cuando el listado, con o sin filtros, no arroja resultados. |
| RF-04 | AC-011 | [raw-requirement.md](references/raw-requirement.md) | RF-03 | Pendiente | El sistema NO DEBE permitir editar ni eliminar un comentario ya agregado, y cada comentario DEBE mostrar su contenido, fecha de creación y autor. |
| RF-05 | AC-012 | [raw-requirement.md](references/raw-requirement.md) | RF-03 | Pendiente | El sistema DEBE reflejar el nuevo estado de la incidencia tanto en su detalle como en el listado tras una transición válida. |
| RF-06 | AC-013 | [raw-requirement.md](references/raw-requirement.md) | RF-01, RF-05 | Pendiente | El dashboard NO DEBE incluir gráficos analíticos avanzados ni comparaciones históricas. |
| RF-01 | AC-014 | [raw-requirement.md](references/raw-requirement.md) | Ninguna | Pendiente | El sistema DEBE rechazar un título de menos de 5 o más de 100 caracteres, una descripción de menos de 10 o más de 2000, y una categoría o prioridad fuera de los valores admitidos. |
| RF-04 | AC-015 | [raw-requirement.md](references/raw-requirement.md) | RF-03 | Pendiente | El sistema DEBE rechazar un comentario vacío o de más de 500 caracteres. |
| RF-06 | AC-016 | [raw-requirement.md](references/raw-requirement.md) | RF-01, RF-05 | Pendiente | El total del dashboard DEBE ser igual a la suma de las incidencias abiertas, en atención y resueltas, y DEBE reflejar los datos vigentes al abrir o recargar la página. |

## 13. Riesgos de producto

Ninguno identificado (revisado con el negocio el 2026-10-28).

## 14. Enlaces y archivos de apoyo

- **Requerimiento original:** [references/raw-requirement.md](references/raw-requirement.md) (conservado tal cual, con su título anterior «RQ-001: HelpDesk Lite - PRD»)
- **Wireframes:** en la carpeta de cada historia (sección 11)
- **RQ relacionado:** Ninguno

## 15. Historias de usuario derivadas

| US-XXX | Título | FR-XXX cubiertos |
| ------ | ------ | ---------------- |
| [US-001](../../user-stories/US-001-registrar-incidencia/README.md) | Registrar una incidencia | RF-01, RNF-06 |
| [US-002](../../user-stories/US-002-listar-filtrar-incidencias/README.md) | Consultar y filtrar incidencias | RF-02, RNF-06 |
| [US-003](../../user-stories/US-003-detalle-incidencia/README.md) | Consultar el detalle de una incidencia | RF-03, RNF-06 |
| [US-004](../../user-stories/US-004-comentar-incidencia/README.md) | Agregar comentarios a una incidencia | RF-04, RNF-06 |
| [US-005](../../user-stories/US-005-cambiar-estado-incidencia/README.md) | Cambiar el estado de una incidencia | RF-05, RNF-06 |
| [US-006](../../user-stories/US-006-dashboard-indicadores/README.md) | Consultar el dashboard de indicadores | RF-06, RNF-06 |

## Observaciones

- **Para promover a SRS falta:** stack, repositorios, umbrales de RNF, interfaces 8.2-8.4, entidades de datos, cómo se cumple la normativa, método de verificación de los AC y riesgos técnicos.
- Este documento se creó como `RQ-001` pero su contenido es un PRD (la definición es del propio equipo, que además implementa). Se renombró a `PRD-001` conservando el número; los ids `RF-XX`, `RNF-06` y `AC-001`…`AC-008` se mantienen. `AC-009` es nuevo (cubre `RNF-06`, que no tenía criterio).
- Los `OB-XX`, `SC-XX`, `BR-01`/`BR-02` y las prioridades se derivaron del texto original y fueron confirmados por el negocio el 2026-10-28.
- Aprobación de negocio registrada (equipo, 2026-10-28); riesgos y expectativas de calidad revisados. La validación final (Ola 3) se superó el 2026-10-28.
- Los ids `RF-XX`/`RNF-XX` no siguen el prefijo `FR-`/`NFR-` de la plantilla; se conservaron porque las historias derivadas los citan.

- **Ola 3 — decisiones de negocio (2026-10-28):** se incorporaron al PRD las reglas de datos de los wireframes (límites, catálogos, página de 20, orden por defecto); todos los actores ven y operan todas las incidencias; los indicadores del dashboard se calculan al abrir o recargar la página.
- **Ola 3 — resueltos:** añadidos `AC-010` a `AC-016` (resultado vacío, comentarios inmutables y sus campos, reflejo del estado, dashboard sin analítica avanzada, reglas de validación, total coherente), reapertura en `SC-02`, definición de estados en 1.3.
- **Ola 3 — aceptados sin cambio (no bloquean):** `RNF-06` queda en `Proposed` (cualitativo, se cuantifica al promover); los seis `WF-001` comparten id por vivir en carpetas de historia distintas; los wireframes citan los `AC-XXX` de su historia, no los del PRD; «mensajes comprensibles» se mantiene cualitativo; el autor de los comentarios («Agente demo») y el usuario del encabezado («Empleado demo») son usuarios de demostración.

## Validación

### Definition of Ready (DoR) del PRD

| Criterio DoR | Estado | Notas |
| ------------ | ------ | ----- |
| Problema y evidencia descritos (1.1) | Met | Problema descrito; evidencia y costo declarados como «Sin medición disponible». |
| Objetivos y métricas de éxito definidos (1.4) | Met | `OB-01` a `OB-04` con métrica proxy, confirmados por el negocio. |
| Alcance y fuera de alcance claros (1.2) | Met | |
| Actores y escenarios de uso (2.4) | Met | Cada actor tiene un `SC-XX`. |
| Requisitos funcionales completos y priorizados (5) | Met | `RF-01` a `RF-06` priorizados y `Approved`. |
| Expectativas de calidad revisadas (6) | Met | `RNF-06` (Usabilidad); Seguridad, Rendimiento y Fiabilidad declaradas sin expectativas (app de demostración). |
| Interfaz de usuario y wireframes revisados (8.1, 11) | Met | Los seis wireframes están `Approved`. |
| Obligaciones de datos identificadas (9) | Met | No aplica: sin obligaciones identificadas. |
| Normativa aplicable identificada (10) | Met | No aplica: sin normativa identificada. |
| Criterios de aceptación completos (12) | Met | `AC-001` a `AC-016` cubren `RF-01` a `RF-06` y `RNF-06`. |
| Riesgos de producto identificados (13) | Met | «Ninguno identificado», revisado con el negocio. |
| Sin aclaraciones pendientes de negocio | Met | Confirmados `OB-XX`, `SC-XX`, `BR-XX` y prioridades. |
| Validación final (Ola 3, PO+UX) superada | Met | Ejecutada el 2026-10-28 con subagente sin contexto; hallazgos resueltos o aceptados (ver Observaciones). |
| Aprobación de negocio registrada | Met | Equipo (autor del documento), 2026-10-28, en la cabecera. |
