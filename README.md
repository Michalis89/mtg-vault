# MTG Vault

MTG Vault is a full-stack Magic: The Gathering application for managing card collections, building decks, analysing deck statistics, and working with different Magic formats.

## Planned Stack

### Frontend
- Angular
- TypeScript

### Backend
- Java 17
- Spring Boot

### Database
- PostgreSQL

### Infrastructure
- Docker
- Docker Compose

## Development Modes

MTG Vault is designed to support two development workflows.

### Standalone Development

Individual services can be run independently for faster development and debugging.

```text
Angular      → npm run start
Spring Boot  → IntelliJ Run / Debug
PostgreSQL   → Docker
```

This allows frontend or backend changes to be developed without rebuilding the entire Docker environment.

### Full Docker Environment

The complete application can also run as a containerized stack using Docker Compose.

```text
Docker Compose
├── frontend
├── backend
└── postgres
```

This provides a reproducible environment for full-stack integration testing and a setup closer to deployment.

Application code remains the same between both modes. Environment-specific configuration determines how services connect to each other.

## Project Structure

The project uses a monorepo structure:

```text
mtg-vault/
├── frontend/
│   └── Dockerfile
├── backend/
│   └── Dockerfile
├── infrastructure/
│   └── postgres/
│       ├── init/
│       └── config/
├── scripts/
├── docs/
├── docker-compose.yml
├── README.md
└── PROJECT_PLAN.md
```
