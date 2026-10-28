---
type: story-validation
story: ./README.md
updated: 2026-10-12
story_points: 3
---
# Validación — US-004: Agregar comentarios a una incidencia

## Complejidad sugerida

- **Story points:** 3
- **Justificación:** Un endpoint de escritura con nueva entidad persistida y un formulario sencillo embebido en una pantalla existente.

## INVEST

| Letra | Criterio      | Resultado | Notas |
| ----- | ------------- | --------- | ----- |
| **I** | Independiente | Partial   | Depende de la pantalla y la respuesta de detalle de US-003. |
| **N** | Negociable    | Met       | Límite de longitud y disposición ajustables. |
| **V** | Valiosa       | Met       | Habilita el seguimiento de la atención. |
| **E** | Estimable     | Met       | Alcance acotado. |
| **S** | Pequeña       | Met       | Un endpoint y un formulario. |
| **T** | Testeable     | Met       | Verificable por pruebas de API, de componente y e2e. |

## Definition of Ready (DoR)

| Criterio DoR                       | Estado  | Notas |
| ---------------------------------- | ------- | ----- |
| Dependencias listas                | Met     | US-003 especificada; implementar después de ella. |
| Inputs/outputs claros              | Met     | Entradas y salidas definidas en los criterios. |
| Repositorios definidos             | Met     | helpdesk-backend, helpdesk-frontend. |
| Sin decisiones técnicas pendientes | Met     | Autor fijo y longitud 1–500 confirmados; contrato en API-001. |
| Referencias de UI                  | Met     | Wireframe aprobado. |
| Sin aclaraciones pendientes        | Met     | Nada pendiente con usuario/producto. |
