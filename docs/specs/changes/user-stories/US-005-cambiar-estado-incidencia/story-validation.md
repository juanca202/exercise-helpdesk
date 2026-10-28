---
type: story-validation
story: ./README.md
updated: 2026-10-12
story_points: 3
---
# Validación — US-005: Cambiar el estado de una incidencia

## Complejidad sugerida

- **Story points:** 3
- **Justificación:** Un endpoint con una máquina de estados pequeña y acciones condicionales en la pantalla de detalle.

## INVEST

| Letra | Criterio      | Resultado | Notas |
| ----- | ------------- | --------- | ----- |
| **I** | Independiente | Partial   | Depende de la pantalla de detalle de US-003. |
| **N** | Negociable    | Met       | Etiquetas y disposición de las acciones ajustables. |
| **V** | Valiosa       | Met       | Refleja el avance de la atención. |
| **E** | Estimable     | Met       | Transiciones cerradas y alcance acotado. |
| **S** | Pequeña       | Met       | Un endpoint y unas acciones en una pantalla existente. |
| **T** | Testeable     | Met       | Verificable por pruebas de API, de componente y e2e. |

## Definition of Ready (DoR)

| Criterio DoR                       | Estado  | Notas |
| ---------------------------------- | ------- | ----- |
| Dependencias listas                | Met     | US-003 especificada; implementar después de ella. |
| Inputs/outputs claros              | Met     | Transiciones y errores definidos en los criterios. |
| Repositorios definidos             | Met     | helpdesk-backend, helpdesk-frontend. |
| Sin decisiones técnicas pendientes | Met     | Valores de `status` literales y sin confirmación (confirmado); contrato en API-001. |
| Referencias de UI                  | Met     | Wireframe aprobado. |
| Sin aclaraciones pendientes        | Met     | Nada pendiente con usuario/producto. |
