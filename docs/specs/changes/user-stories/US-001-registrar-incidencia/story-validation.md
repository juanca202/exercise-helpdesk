---
type: story-validation
story: ./README.md
updated: 2026-10-12
story_points: 5
---
# Validación — US-001: Registrar una incidencia

## Complejidad sugerida

- **Story points:** 5
- **Justificación:** Alcance acotado (un formulario y un endpoint con persistencia), pero involucra dos repositorios, el contrato compartido y la primera configuración de persistencia SQLite (ADR-007).

## INVEST

| Letra | Criterio      | Resultado | Notas |
| ----- | ------------- | --------- | ----- |
| **I** | Independiente | Met       | Es la primera historia; las demás (RF-02..RF-06) dependen de ella, no al revés. |
| **N** | Negociable    | Met       | Los valores de catálogo y límites son ajustables sin cambiar la intención. |
| **V** | Valiosa       | Met       | Habilita que el equipo de soporte reciba solicitudes. |
| **E** | Estimable     | Met       | Alcance y contrato claros; stack ya definido. |
| **S** | Pequeña       | Partial   | Abarca dos repositorios y la puesta en marcha de la persistencia; cabe en una sesión del taller. |
| **T** | Testeable     | Met       | Cada criterio es verificable por pruebas de API, de componente y e2e. |

## Definition of Ready (DoR)

| Criterio DoR                       | Estado  | Notas |
| ---------------------------------- | ------- | ----- |
| Dependencias listas                | Met     | Stack y persistencia decididos (ADR-007, ADR-008). |
| Inputs/outputs claros              | Met     | Entradas y salidas definidas en los criterios. |
| Repositorios definidos             | Met     | helpdesk-backend, helpdesk-frontend. |
| Sin decisiones técnicas pendientes | Met     | Id, prioridad, errores y ruta confirmados; la librería de datos se decide al planificar. |
| Referencias de UI                  | Met     | WF-001 aprobado. |
| Sin aclaraciones pendientes        | Met     | Nada pendiente con usuario/producto. |
