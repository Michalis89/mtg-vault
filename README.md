# MTG Vault

MTG Vault is a full-stack Magic: The Gathering application for managing card collections, building decks, analysing deck statistics, and working with different Magic formats.

## Planned Stack

### Frontend
- Angular
- TypeScript

### Backend
- Java 17
- Spring Boot
- Spring Data JPA
- Hibernate
- Testcontainers

### Database
- PostgreSQL

### Infrastructure
- Docker
- Docker Compose

### Planned
- Flyway
- GitHub Actions
- Production deployment
- Architecture Decision Records (ADRs)

## Current Status

The project currently includes:

- Spring Boot backend
- `/api/health` endpoint
- PostgreSQL running in Docker
- Backend Docker image
- Docker Compose orchestration for backend and PostgreSQL
- JPA / Hibernate
- PostgreSQL JDBC connectivity
- Testcontainers-based PostgreSQL integration test setup
- Standalone and fully containerized backend development workflows

The Angular frontend has not been added yet.

## Prerequisites

For local development, install:

- Java 17
- Docker
- Docker Compose
- IntelliJ IDEA
- Git

Maven does not need to be installed globally because the project uses the Maven Wrapper.

## Environment Configuration

Create a local `.env` file in the repository root.

Example:

```env
POSTGRES_DB=mtg_vault
POSTGRES_USER=mtg_vault
POSTGRES_PASSWORD=mtg_vault_dev
```

The `.env` file is ignored by Git.

Use `.env.example` as the template for required environment variables.

## Development Modes

MTG Vault is designed to support two development workflows.

### 1. Standalone Development

Use this mode for normal day-to-day development and debugging.

```text
Angular      → npm run start        (once frontend is added)
Spring Boot  → IntelliJ Run / Debug
PostgreSQL   → Docker
```

This allows frontend or backend changes to be developed without rebuilding the complete Docker stack.

### Start PostgreSQL

From the repository root:

```bash
docker compose up -d postgres
```

Check its status:

```bash
docker compose ps
```

The PostgreSQL service should report:

```text
healthy
```

### Run Spring Boot from IntelliJ

Keep PostgreSQL running in Docker and stop the Docker backend if it is running:

```bash
docker compose stop backend
```

Run `BackendApplication` from IntelliJ with these environment variables:

```text
DB_URL=jdbc:postgresql://localhost:5432/mtg_vault
DB_USERNAME=mtg_vault
DB_PASSWORD=mtg_vault_dev
```

The backend will be available at:

```text
http://localhost:8080
```

Health check:

```text
http://localhost:8080/api/health
```

Expected response:

```json
{
  "status": "UP"
}
```

### 2. Full Docker Environment

Use this mode to run the application as a containerized stack.

Currently:

```text
Docker Compose
├── backend
└── postgres
```

Once the Angular frontend is added:

```text
Docker Compose
├── frontend
├── backend
└── postgres
```

Build and start the current stack:

```bash
docker compose up -d --build
```

Or rebuild/start only the backend:

```bash
docker compose up -d --build backend
```

Check running services:

```bash
docker compose ps
```

View backend logs:

```bash
docker compose logs backend
```

Follow backend logs live:

```bash
docker compose logs -f backend
```

Stop all services:

```bash
docker compose stop
```

Stop and remove the Compose containers and network:

```bash
docker compose down
```

The PostgreSQL data volume is preserved by `docker compose down`.

## Docker Networking

When Spring Boot runs from IntelliJ, it connects to PostgreSQL through the host machine:

```text
jdbc:postgresql://localhost:5432/mtg_vault
```

When Spring Boot runs inside Docker Compose, it connects using the PostgreSQL service name:

```text
jdbc:postgresql://postgres:5432/mtg_vault
```

Both modes use the same application code. Only environment-specific configuration changes.

## Backend Build and Tests

From the `backend/` directory:

```bash
./mvnw clean package
```

On Windows PowerShell:

```powershell
.\mvnw.cmd clean package
```

Backend tests use Testcontainers to start a temporary PostgreSQL container automatically.

This keeps tests isolated from the development database.

## Docker Images

Build the backend image manually from the `backend/` directory:

```bash
docker build -t mtg-vault-backend .
```

List local Docker images:

```bash
docker images
```

## Project Structure

The project uses a monorepo structure.

Current and planned structure:

```text
mtg-vault/
├── frontend/                 # Planned Angular application
│   └── Dockerfile
├── backend/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── pom.xml
│   └── src/
├── infrastructure/           # Planned infrastructure-specific configuration
│   └── postgres/
├── scripts/                  # Planned build / run helper scripts
├── docs/                     # Planned architecture documentation and ADRs
├── docker-compose.yml
├── .env.example
├── README.md
└── PROJECT_PLAN.md
```

## Development Principles

The project aims to keep:

- application code independent from the execution environment
- local development fast and easy to debug
- the full stack reproducible with Docker
- database configuration externalized through environment variables
- tests isolated from the local development database
- architecture decisions explicit and documented
