# HelpDesk Lite — especificaciones

HelpDesk Lite es una aplicación web para gestionar incidencias de soporte interno: permite registrar solicitudes, consultarlas y filtrarlas, darles seguimiento mediante comentarios y cambios de estado, y visualizar indicadores básicos de la carga de trabajo. Se utiliza como ejercicio práctico de un taller de desarrollo AI-Native con Spec-Driven Development (SDD) y SDD Devkit.

Este repositorio concentra las especificaciones, el diseño y el harness de la solución, y agrega como submódulos los dos repositorios de código: [helpdesk-backend](helpdesk-backend/README.md) (API REST y persistencia) y [helpdesk-frontend](helpdesk-frontend/README.md) (interfaz web). El requerimiento de origen está registrado en [PRD-001: HelpDesk Lite](docs/specs/changes/requirements/PRD-001-helpdesk-lite/README.md).

## Cómo correr la aplicación

Requisitos: Node.js ≥ 22 y npm.

1. Clonar el repositorio con sus submódulos (o inicializarlos si ya está clonado):

   ```bash
   git clone --recurse-submodules <url-del-repositorio>
   # si ya estaba clonado:
   git submodule update --init --recursive
   ```

2. Backend (API REST en `http://localhost:3000`):

   ```bash
   cd helpdesk-backend
   npm install
   cp .env.example .env      # PORT=3000, DATABASE_PATH=./data/helpdesk.db
   npm run dev
   ```

   La base SQLite se crea en `DATABASE_PATH` al arrancar. La documentación OpenAPI se consulta en `http://localhost:3000/docs`.

3. Frontend (interfaz web en `http://localhost:4200`), en otra terminal:

   ```bash
   cd helpdesk-frontend
   npm install
   npm start
   ```

   Abrir `http://localhost:4200/`; el listado de incidencias está en `/incidents`.

> **Conexión entre frontend y backend.** En desarrollo el frontend llama a la API con rutas relativas (`apiRestBaseUrl` vacío en `src/environments/environment.development.ts`) y el repositorio no incluye un proxy de desarrollo ni CORS en el backend. Para que la interfaz hable con la API en local hay que apuntar `apiRestBaseUrl` a `http://localhost:3000` y habilitar CORS en el backend, o publicar ambos tras un mismo origen con un proxy inverso.

### Pruebas y verificaciones

| Repositorio | Comando | Qué hace |
| ----------- | ------- | -------- |
| `helpdesk-backend` | `npm test` · `npm run lint` · `npm run typecheck` | Pruebas (Vitest), linter y tipado |
| `helpdesk-frontend` | `npm test` · `npm run lint` · `npm run e2e` | Pruebas unitarias, linter y E2E (Playwright) |

