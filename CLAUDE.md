# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Ktor backend server for **Mytho Compendium**, providing the REST API consumed by the mobile app. Connects to two databases populated by `mytho-compendium-content`: **Neo4j** for graph relationships between mythological entities, and **PostgreSQL** for structured long-form content. Docker configuration for local development and cloud deployment lives directly in this repository.

## Build Commands

```bash
./gradlew run                                    # Run the server locally (requires local DBs)
./gradlew build                                  # Build the project
./gradlew test                                   # Run all unit tests
./gradlew detekt                                 # Run Detekt static analysis
docker-compose -f docker/docker-compose.yml up   # Start full local stack (server + Neo4j + PostgreSQL)
```

> A `.env` file with database credentials is required to run the server. Copy `.env.example` and fill in your values. It is never committed to the repository.

## Architecture

```text
src/
└── main/
    └── kotlin/
        └── com/mythocompendium/
            ├── Application.kt           # Entry point, plugin setup
            ├── plugins/                 # Ktor plugins (routing, serialization, etc.)
            ├── routing/                 # Route definitions grouped by feature
            │   ├── MythologyRoutes.kt
            │   ├── EntityRoutes.kt
            │   └── SearchRoutes.kt
            ├── domain/                  # Use cases and domain models
            ├── data/
            │   ├── neo4j/               # Graph queries and Neo4j repositories
            │   └── postgresql/          # SQL queries, Exposed tables, PG repositories
            └── di/                      # Koin modules
```

## Tech Stack

| Layer | Technology |
|:---|:---|
| Language | Kotlin |
| Framework | Ktor |
| Graph database | Neo4j |
| Relational database | PostgreSQL |
| ORM / query | Exposed (PostgreSQL) + Neo4j Java Driver |
| Serialization | kotlinx.serialization |
| DI | Koin |
| Containerization | Docker + Docker Compose |
| Static analysis | Detekt |
| CI | GitHub Actions |

## API Overview

| Method | Endpoint | Description |
|:---|:---|:---|
| `GET` | `/mythologies` | List all mythologies |
| `GET` | `/mythologies/{id}` | Get a mythology with its entity graph summary |
| `GET` | `/entities/{id}` | Get full entity detail (content + relations) |
| `GET` | `/entities/{id}/relations` | Get the graph of relations for an entity |
| `GET` | `/search?q=` | Cross-mythology full-text search |
| `GET` | `/health` | Health check endpoint |

## Docker Setup

```text
docker/
  ├── docker-compose.yml        # Full local stack: server + Neo4j + PostgreSQL
  ├── docker-compose.prod.yml   # Production overrides
  └── Dockerfile                # Server image build
```

The local stack spins up:
- Ktor server on `http://localhost:8080`
- Neo4j Browser on `http://localhost:7474`
- PostgreSQL on port `5432`

## Environment Variables

```bash
# Server
SERVER_PORT=8080
ENVIRONMENT=development

# PostgreSQL
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=mythocompendium
POSTGRES_USER=your_user
POSTGRES_PASSWORD=your_password

# Neo4j
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password
```

## CI/CD

```text
.github/workflows/
  ├── pr-checks.yml   # Runs on every PR: build, unit tests, detekt
  └── deploy.yml      # Runs on merge to main: build Docker image, deploy
```

## Related Repositories

- **mytho-compendium-app** — Android app, future KMP; consumes content from this server
- **mytho-compendium-content** — Content pipeline that populates the Neo4j and PostgreSQL databases this server reads from