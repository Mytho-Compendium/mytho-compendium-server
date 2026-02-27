# mytho-compendium-server
> Ktor backend server for **Mytho Compendium**, providing the REST API consumed by the mobile app, with Docker-based infrastructure for local development and cloud deployment.

---

## Project Overview

This repository contains the entire backend for Mytho Compendium. It is built with **Ktor**, a Kotlin-native async web framework, and connects to two databases populated by the `mytho-compendium-content` repository: **Neo4j** for graph relationships between mythological entities, and **PostgreSQL** for structured long-form content.

Docker configuration for local development and cloud deployment lives directly in this repository, infrastructure and server are the same deployable unit.

---

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
| CI | GitHub Actions |

---

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

---

## API Overview

| Method | Endpoint | Description |
|:---|:---|:---|
| `GET` | `/mythologies` | List all mythologies |
| `GET` | `/mythologies/{id}` | Get a mythology with its entity graph summary |
| `GET` | `/entities/{id}` | Get full entity detail (content + relations) |
| `GET` | `/entities/{id}/relations` | Get the graph of relations for an entity |
| `GET` | `/search?q=` | Cross-mythology full-text search |
| `GET` | `/health` | Health check endpoint |

---

## Database Connections

This server reads from two databases populated independently by the `mytho-compendium-content` repository.

**Neo4j** - stores entities and their relationships (gods, creatures, artifacts, locations, events) and the connections between them across and within mythologies.

**PostgreSQL** - stores all structured long-form content for each entity: descriptions, epithets, cultural context, and source citations.

The two databases are joined at the API layer by shared entity UUIDs. Neo4j provides the graph traversal; PostgreSQL provides the content payload.

---

## Docker Setup

Docker Compose is used for both local development and production deployment.

```text
docker/
  ├── docker-compose.yml        # Full local stack: server + Neo4j + PostgreSQL
  ├── docker-compose.prod.yml   # Production overrides
  └── Dockerfile                # Server image build
```

**Start the full local stack:**

```bash
docker-compose -f docker/docker-compose.yml up
```

This spins up:
- The Ktor server on `http://localhost:8080`
- Neo4j Browser on `http://localhost:7474`
- PostgreSQL on port `5432`

> ⚠️ Database credentials and environment variables are managed via `.env` files. A `.env.example` is provided. Never commit a real `.env` file.

---

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

---

## CI/CD

```text
.github/workflows/
  ├── pr-checks.yml   # Runs on every PR: build, unit tests, detekt
  └── deploy.yml      # Runs on merge to main: build Docker image, deploy
```

---

## Getting Started

```bash
# Clone the repository
git clone https://github.com/your-username/mytho-compendium-server.git

# Copy environment file
cp .env.example .env
# Fill in your local credentials

# Start everything with Docker Compose
docker-compose -f docker/docker-compose.yml up

# Or run the server locally without Docker (requires local Neo4j and PostgreSQL)
./gradlew run
```

---

## Related Repositories

| Repository | Purpose |
|:---|:---|
| **mytho-compendium-app** | Android app, future KMP |
| **mytho-compendium-server** _(this repo)_ | Ktor backend, Docker, databases |
| **mytho-compendium-content** | Content pipeline, Claude Code skills, data ingestion |