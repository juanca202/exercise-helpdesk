PRD — HelpDesk Lite

Versión: 0.1.0
Estado: Draft
Tipo: Producto para taller AI-Native SDLC con SDD
Modalidad: Aplicación web con arquitectura multirepo

1. Resumen ejecutivo

HelpDesk Lite es una aplicación web para gestionar incidencias de soporte interno de una organización. Permite registrar solicitudes, consultar y filtrar incidencias, gestionar su ciclo de vida y visualizar indicadores básicos de atención.

El proyecto se utilizará como ejercicio práctico para un taller de desarrollo de software nativo de IA (AI-Native SDLC), utilizando Spec-Driven Development (SDD) y SDD Devkit.

A diferencia de un ejercicio de implementación convencional, los participantes deberán producir y mantener los artefactos de especificación, diseño, implementación y validación necesarios para construir una solución coherente entre dos repositorios independientes.

2. Problema y oportunidad

Las solicitudes internas de soporte requieren un mecanismo centralizado que permita registrar incidencias, consultar su estado y realizar seguimiento de su resolución.

Para este ejercicio se plantea una solución de alcance reducido que permita demostrar cómo los requisitos funcionales se transforman en modelos, diagramas, contratos API, wireframes, tareas de implementación y pruebas verificables.

3. Objetivos del producto

3.1 Objetivos funcionales

Permitir el registro de incidencias de soporte.

Facilitar la consulta y el filtrado de incidencias.

Permitir el seguimiento de incidencias mediante comentarios y cambios de estado.

Proporcionar una vista resumida de la carga de trabajo y del estado de las incidencias.

3.2 Objetivos de aprendizaje

Configurar un proyecto SDD con múltiples repositorios.

Diseñar una solución utilizando modelos, diagramas, flujos, contratos API y wireframes.

Mantener trazabilidad entre requisitos, User Stories, tareas, implementación y pruebas.

Coordinar el desarrollo de frontend y backend mediante contratos explícitos.

Validar la implementación respecto de las especificaciones acordadas.

4. Alcance

4.1 Incluido

Registro de incidencias.

Consulta, filtrado y ordenamiento de incidencias.

Visualización del detalle de una incidencia.

Agregado de comentarios.

Actualización del estado de una incidencia mediante transiciones permitidas.

Dashboard con métricas básicas.

API REST para las capacidades funcionales.

Persistencia de los datos.

Validaciones funcionales y manejo de errores.

Pruebas unitarias, de contrato, integración y end-to-end según corresponda.

Diseño documentado mediante artefactos SDD.

Separación del frontend y backend en repositorios independientes.

4.2 Fuera de alcance

Autenticación real, registro de usuarios y recuperación de contraseñas.

Integración con proveedores de identidad.

Sistema completo de autorización basado en roles.

Notificaciones por correo, SMS o mensajería.

Archivos adjuntos.

SLA, escalamiento automático y asignación automática de agentes.

Integración con sistemas externos de gestión de servicios.

Aplicación móvil nativa.

Despliegue productivo, alta disponibilidad y observabilidad avanzada.

Analítica histórica avanzada y generación de informes exportables.

Para las demostraciones se podrán utilizar usuarios y roles predefinidos, sin implementar autenticación real.

5. Usuarios y actores

5.1 Empleado solicitante

Persona que registra una incidencia y consulta su información.

5.2 Agente de soporte

Persona que consulta incidencias, revisa sus detalles, agrega comentarios y actualiza sus estados.

5.3 Responsable de soporte

Persona que consulta los indicadores generales para conocer la distribución de las incidencias.

Estos actores representan perspectivas funcionales. No implican la implementación de un sistema real de identidad o permisos.

6. Requisitos funcionales

RF-01. Registrar una incidencia

El sistema debe permitir registrar una incidencia con los siguientes datos:

Título.

Descripción.

Categoría.

Prioridad.

Al registrar una incidencia, el sistema debe asignar un identificador único, una fecha de creación y un estado inicial.

El sistema debe validar los campos obligatorios y rechazar solicitudes inválidas con mensajes comprensibles.

Resultado esperado: la incidencia queda persistida y puede consultarse posteriormente.

RF-02. Consultar y filtrar incidencias

El sistema debe presentar un listado de incidencias registradas.

El usuario debe poder:

Consultar el identificador, título, categoría, prioridad, estado y fecha de creación.

Filtrar por estado, prioridad y categoría.

Ordenar por fecha de creación.

Consultar resultados vacíos de manera comprensible.

La consulta debe soportar paginación básica para evitar recuperar todos los registros en una sola solicitud.

Resultado esperado: el usuario puede localizar las incidencias relevantes utilizando los filtros disponibles.

RF-03. Consultar el detalle de una incidencia

El sistema debe permitir consultar una incidencia individual a partir de su identificador.

El detalle debe incluir sus datos principales, estado actual, fechas relevantes y comentarios asociados.

Si el identificador no corresponde a una incidencia existente, el sistema debe comunicar que el recurso no fue encontrado.

RF-04. Agregar comentarios

El sistema debe permitir agregar comentarios a una incidencia existente.

Cada comentario debe contener, como mínimo:

Identificador único.

Contenido.

Fecha de creación.

Autor o usuario de demostración.

Los comentarios deben persistirse y mostrarse en orden cronológico.

No se contempla la edición ni eliminación de comentarios en esta versión.

RF-05. Cambiar el estado de una incidencia

El sistema debe permitir actualizar el estado de una incidencia únicamente mediante transiciones válidas.

Para la versión inicial se proponen los siguientes estados:

Open: incidencia registrada y pendiente de atención.

In Progress: incidencia en atención.

Resolved: incidencia resuelta.

Transiciones permitidas:

Open → In Progress.

In Progress → Resolved.

Resolved → In Progress, para reabrir una incidencia que requiera atención adicional.

Las transiciones no contempladas deben rechazarse sin modificar el estado persistido.

El sistema debe reflejar el nuevo estado en el detalle y en el listado.

RF-06. Consultar el dashboard

El sistema debe presentar indicadores básicos de las incidencias persistidas:

Total de incidencias.

Cantidad de incidencias abiertas.

Cantidad de incidencias en atención.

Cantidad de incidencias resueltas.

Los indicadores deben corresponder a los datos disponibles en el sistema y actualizarse después de las operaciones que modifiquen esos datos.

El dashboard no contempla gráficos analíticos avanzados ni comparaciones históricas.

7. Requisitos no funcionales

RNF-01. Separación de responsabilidades

El frontend y el backend deben desarrollarse en repositorios independientes, con responsabilidades claramente delimitadas.

RNF-02. Contrato de integración

La comunicación entre frontend y backend debe realizarse mediante una API REST documentada.

El contrato debe definir endpoints, esquemas de datos, códigos HTTP, validaciones y respuestas de error.

RNF-03. Consistencia de datos

Las operaciones de creación y actualización deben persistir sus resultados. Las transiciones inválidas no deben producir cambios parciales en el estado de una incidencia.

RNF-04. Mantenibilidad

La solución debe mantener una separación clara entre presentación, lógica de aplicación, reglas de negocio y persistencia, según corresponda a la arquitectura seleccionada.

RNF-05. Testabilidad

Las capacidades funcionales deben poder verificarse mediante pruebas automatizadas. El entorno de desarrollo debe permitir ejecutar el frontend, el backend y las pruebas sin depender de servicios externos de negocio.

RNF-06. Usabilidad

La interfaz debe presentar formularios comprensibles, validaciones visibles, indicadores de carga, mensajes de error y estados vacíos.

RNF-07. Tiempo de ejecución del taller

El alcance debe poder implementarse y verificarse durante tres sesiones de dos horas, utilizando agentes de IA y un entorno de desarrollo previamente preparado.

Este requisito constituye un objetivo de planificación, no una garantía de finalización de todas las funcionalidades en cualquier entorno.

8. User Stories y trazabilidad

El proyecto se organiza en un máximo de cuatro User Stories.

US-01. Registrar una incidencia

Como empleado, quiero registrar una solicitud de soporte para que el equipo pueda atenderla.

Requisitos asociados: RF-01.

Resultado verificable: una incidencia válida se registra y puede recuperarse posteriormente.

US-02. Consultar y filtrar incidencias

Como agente de soporte, quiero consultar y filtrar las incidencias para priorizar mi trabajo.

Requisitos asociados: RF-02.

Resultado verificable: el listado refleja los datos persistidos y aplica correctamente los filtros, el ordenamiento y la paginación.

US-03. Gestionar el ciclo de vida de una incidencia

Como agente de soporte, quiero consultar el detalle, agregar comentarios y cambiar el estado para dar seguimiento a una solicitud.

Requisitos asociados: RF-03, RF-04 y RF-05.

Resultado verificable: los comentarios quedan registrados, las transiciones válidas se aplican y las transiciones inválidas son rechazadas.

US-04. Visualizar el estado del soporte

Como responsable de soporte, quiero consultar un resumen de las incidencias para conocer la carga de trabajo y su estado.

Requisitos asociados: RF-06.

Resultado verificable: los indicadores reflejan las incidencias registradas y sus estados actuales.

9. Requisitos de diseño

Antes de completar la implementación, se deben elaborar y revisar los siguientes artefactos:

Modelo de dominio y modelo de datos.

Diagrama de componentes y límites entre frontend y backend.

Diagrama de estados de una incidencia.

Diagramas de secuencia para operaciones representativas.

Flujos funcionales de registro, consulta y actualización.

Contrato OpenAPI de la API REST.

Wireframes del listado, formulario, detalle y dashboard.

ADRs para las decisiones arquitectónicas relevantes.

Criterios de aceptación y casos de prueba vinculados a las User Stories.

Los artefactos compartidos deben contar con una ubicación canónica y ser referenciados por las historias que los utilizan, evitando duplicar definiciones de modelos y contratos.

10. Organización multirepo

10.1 helpdesk-frontend

Responsable de:

Implementar las vistas y componentes de interfaz.

Gestionar formularios y validaciones de presentación.

Consumir la API REST.

Representar estados de carga, éxito, error y ausencia de datos.

Implementar pruebas de componentes y flujos de usuario.

10.2 helpdesk-backend

Responsable de:

Exponer los endpoints REST.

Validar las solicitudes recibidas.

Aplicar las reglas de negocio y transiciones de estado.

Persistir incidencias y comentarios.

Calcular los indicadores del dashboard.

Implementar pruebas unitarias, de integración y de contrato según corresponda.

10.3 Artefactos compartidos

Los modelos, decisiones de arquitectura y contratos que afectan a ambos repositorios deben mantenerse en una ubicación acordada y versionada.

La configuración concreta de repositorios, dependencias, worktrees y flujos de trabajo se definirá durante la preparación del ejercicio en SDD Devkit.

11. Criterios de aceptación del producto

El producto se considera funcionalmente aceptable cuando:

Se puede registrar una incidencia válida y persistirla.

Se rechazan los datos obligatorios ausentes o inválidos.

El listado permite filtrar, ordenar y paginar los resultados.

Se puede consultar el detalle de una incidencia.

Se pueden agregar y consultar comentarios persistidos.

Las transiciones válidas de estado se aplican correctamente.

Las transiciones inválidas no modifican el estado persistido.

El dashboard presenta indicadores coherentes con los datos registrados.

El frontend consume la API documentada y maneja sus respuestas de error.

Las pruebas definidas para las funcionalidades implementadas pueden ejecutarse y sus resultados quedan disponibles.

Los artefactos SDD mantienen trazabilidad con las historias y la implementación entregada.

Las funcionalidades incompletas al finalizar el taller deben quedar identificadas explícitamente, sin declarar satisfechos criterios que no hayan sido verificados.

12. Restricciones y supuestos

El producto es una aplicación de demostración, no un sistema de soporte listo para producción.

Se utilizará un entorno de desarrollo local previamente preparado.

Los datos de prueba podrán ser sintéticos.

No se requiere autenticación real.

El stack tecnológico se confirmará antes de la implementación.

Los participantes utilizarán agentes de IA como apoyo para diseñar, implementar y validar.

Las decisiones arquitectónicas deben documentarse antes de que se conviertan en dependencias difíciles de cambiar.

El proyecto debe mantener un alcance compatible con las seis horas de trabajo práctico.