# TK-003: Formulario de comentario en el detalle de incidencia

<!-- tk:status=Ready -->
**Estado:** Ready
**Historia:** [US-004](./README.md)
**Repositorio:** helpdesk-frontend

## Descripción

Implementar el formulario «Nuevo comentario» embebido en la tarjeta «Comentarios» de la pantalla de detalle (`/incidents/:id`) según el wireframe aprobado: campo multilínea con ayuda y contador `n / 500`, error de validación junto al campo sin enviar si el contenido es inválido, indicador de carga y bloqueo durante el envío, el comentario creado añadido al final de la lista sin recargar la página y el campo vacío tras el éxito; ante un fallo, mensaje de error comprensible conservando el texto escrito, y los errores `400` de la API mostrados en el campo.

## Dependencias

- Pantalla de detalle de incidencia ([TK-004 de US-003](../US-003-detalle-incidencia/TK-004-pagina-detalle-incidencia.md)) — aloja el formulario bajo la lista de comentarios y mantiene la lista y el contador de la tarjeta.
- Mutación `addComment` y `CommentMapper` ([TK-002](./TK-002-repositorio-comentarios.md)) — envío del comentario.
- Modelo `Comment` ([TK-003 de US-003](../US-003-detalle-incidencia/TK-003-repositorio-detalle-incidencia.md)) — comentario devuelto por el `201` que se añade a la lista.
- Signal Forms (`@angular/forms/signals`) — estado y validación del formulario.
- Angular Material y Tailwind CSS — campo, botón, indicador y estilos.
- `msw` y handlers de [TK-002](./TK-002-repositorio-comentarios.md) — respuestas de la API en las pruebas unitarias.

## Referencias

- **Arquitectura:** [ADR-001: Arquitectura por capas y features](../../../../../helpdesk-frontend/docs/adr/ADR-001-hybrid-layered-feature-architecture.md) · [ADR-006: Presentación con Tailwind CSS](../../../../../helpdesk-frontend/docs/adr/ADR-006-presentation-tailwind-css.md) · [ADR-003: Pruebas unitarias con Vitest y Testing Library](../../../../../helpdesk-frontend/docs/adr/ADR-003-unit-testing-vitest-testing-library.md) · [ADR-007: Mocks HTTP con MSW](../../../../../helpdesk-frontend/docs/adr/ADR-007-http-api-mocks-msw.md) · [ADR-013: Componente de icono unificado](../../../../../helpdesk-frontend/docs/adr/ADR-013-unified-ft-icon-component.md)
- **Estándares:** [Frontend Standards](../../../../../helpdesk-frontend/docs/standards/frontend.md) · [Architecture Standards](../../../../../helpdesk-frontend/docs/standards/architecture.md) · [Testing Standards](../../../../../helpdesk-frontend/docs/standards/testing.md)
- **Documentación técnica:** [FL-004: Registro de un comentario](../../../../architecture/incidencias/flows/FL-004-registro-comentario.md) · [API-001: POST /incidents/{id}/comments](../../../../architecture/incidencias/apis/API-001-incidencias.md#post-incidents-id-comments) · [MD-002: Comentario](../../../../architecture/incidencias/models/MD-002-comentario.md)
- **Diseño:** [WF-001: Formulario de comentario](./wireframes/WF-001-formulario-comentario.md) — Approved · [WF-001 de US-003: Detalle de incidencia](../US-003-detalle-incidencia/wireframes/WF-001-detalle-incidencia.md) — pantalla contenedora y vista móvil

## Archivos afectados

```text
helpdesk-frontend/
└── src/app/features/incidents/
    ├── components/comment-form/
    │   ├── + comment-form.ts                                # componente del formulario; emite el comentario creado
    │   ├── + comment-form.html                              # plantilla con los estados del wireframe
    │   ├── + comment-form.css                               # estilos
    │   └── + comment-form.spec.ts                           # unit con Testing Library, MSW y AXE
    └── pages/incident-detail/
        ├── ~ incident-detail.ts                             # añade el comentario recibido a la lista
        ├── ~ incident-detail.html                           # inserta el formulario bajo la lista de comentarios
        └── ~ incident-detail.spec.ts                        # comentario añadido sin recargar y contador de la tarjeta
```

## Plan de implementación

- [ ] **IT-01** — Crear el componente `comment-form` con Signal Forms: campo «Nuevo comentario» obligatorio y multilínea, ayuda «Entre 1 y 500 caracteres» y botón primario «Agregar comentario»; recibe el id de la incidencia
- [ ] **IT-02** — Aplicar la validación de presentación con las reglas de `content` de MD-002 (recorte, 1–500; solo espacios cuenta como vacío) y mostrar el contador `n / 500` sobre el contenido recortado
- [ ] **IT-03** — Al pulsar «Agregar comentario» con contenido inválido, mostrar el error junto al campo y no enviar la petición
  Mensajes: «El comentario no puede estar vacío» y «El comentario no puede superar los 500 caracteres». El botón no se deshabilita por errores, solo durante el envío.
- [ ] **IT-04** — Enviar con `addComment`: el botón muestra el indicador y «Agregando…», y campo y botón quedan bloqueados mientras dura la petición para evitar envíos duplicados
- [ ] **IT-05** — Ante `201`, emitir el comentario devuelto, vaciar el campo y reiniciar la validación; la pantalla de detalle lo añade al final de la lista, lo destaca brevemente como «Recién agregado» y aumenta el contador de la tarjeta, sin volver a pedir el detalle
- [ ] **IT-06** — Ante `400`, mostrar en el campo el `issue` con `instancePath` `/content` y desbloquear el formulario conservando el texto
- [ ] **IT-07** — Ante `404`, `500` o fallo de red, mostrar junto al botón el aviso «No se pudo agregar el comentario» (con el motivo de la API en el `404`), conservar el texto y rehabilitar el formulario para reintentar
- [ ] **IT-08** — Ajustar la disposición responsiva: en móvil el formulario ocupa el ancho completo al final del detalle, con el botón a ancho completo
- [ ] **IT-09** — Pruebas unitarias del formulario: contador, errores de vacío, solo espacios y 501 caracteres sin petición, bloqueo e indicador durante el envío, éxito con campo vacío, `400` en el campo, `404`/`500` con texto conservado y accesibilidad (AXE)
- [ ] **IT-10** — Pruebas unitarias de la pantalla de detalle: el comentario creado aparece al final de la lista con autor y fecha, sin recargar, y el contador de la tarjeta aumenta
