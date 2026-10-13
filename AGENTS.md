# Fuentes de contexto

- @.agents/MEMORY.md — Preferencias del proyecto
- @README.md — Acerca del proyecto

# Reglas generales

<!-- Normas de trabajo para agentes que no son preferencias (MEMORY), decisiones de arquitectura (ADR/estándares) ni stack: convenciones de código, comandos del repo, restricciones operativas. Ejemplo: "Seguir el estilo del código vecino; no reformatear líneas que no formen parte del cambio." -->

# Stack tecnológico

Repositorio de especificaciones — sin stack de aplicación propio (harness y documentación). Stack de cada repositorio de la solución:

| Repositorio | Stack | Detalle |
| ----------- | ----- | ------- |
| `helpdesk-backend` | Node.js ≥ 22, TypeScript, Fastify 5 + Zod + OpenAPI 3.1, SQLite; Vitest | [helpdesk-backend/AGENTS.md](helpdesk-backend/AGENTS.md#stack-tecnológico) |
| `helpdesk-frontend` | Angular 22, TypeScript, Tailwind CSS 4, Angular Material; Vitest + Testing Library, Playwright | [helpdesk-frontend/AGENTS.md](helpdesk-frontend/AGENTS.md#stack-tecnológico) |
