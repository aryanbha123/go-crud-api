# CRUD-GO

A layered REST API in Go for managing users, built with Gin and PostgreSQL.

![Go](https://img.shields.io/badge/Go-00ADD8?style=for-the-badge&logo=go&logoColor=white)
![Gin](https://img.shields.io/badge/Gin-008ECF?style=for-the-badge&logo=gin&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

## About

CRUD-GO is a small backend service that exposes create, read, update and delete operations on a `users` table. The code follows a classic layered layout (routes, handlers, services, repository), so each layer has one job and is easy to follow. It is a compact reference for structuring a Go web API around a pgx connection pool.

## Features

- Full CRUD for users under `/api/v1/users`
- Passwords hashed with bcrypt before they are stored; the hash is never returned in responses
- Request validation through Gin binding tags (required fields, email format, minimum password length of 6)
- Clear error mapping: `404` for a missing user, `409` for a duplicate email (Postgres unique violation `23505`), `400` for invalid input
- Health check endpoint with an optional `name` query parameter
- pgx connection pool with tuned limits (max 10 / min 2 connections, idle and lifetime timeouts) and a startup ping
- CORS enabled for local frontends on `http://localhost:3000` and `http://localhost:5173`
- Configuration from environment variables or a `.env` file (via godotenv), with sensible defaults
- Two static HTML pages in `cmd/ui` (a landing page and a health-check page that calls `/api/v1/health`)

## Tech Stack

| Layer | Technology |
| --- | --- |
| Language | Go |
| HTTP framework | Gin, gin-contrib/cors |
| Database | PostgreSQL 17 (via Docker Compose) |
| DB driver | jackc/pgx v5 (pgxpool) |
| Security | golang.org/x/crypto/bcrypt |
| Config | joho/godotenv |

## Project Structure

```
go-lag-crud/
├── cmd/
│   ├── api/main.go          # entry point: config, DB pool, CORS, router
│   └── ui/                  # static index.html and health.html
├── internal/
│   ├── config/              # env loading and pgx pool setup
│   ├── routes/              # route registration
│   ├── handlers/            # HTTP handlers and request validation
│   ├── services/            # business logic (bcrypt hashing)
│   └── repository/          # SQL queries against the users table
├── schema/table.sql         # users table definition
├── docker-compose.yaml      # API + Postgres services
└── Makefile
```

## Getting Started

### Prerequisites

- Go (the module targets the version in `go.mod`)
- PostgreSQL, or Docker to run the bundled Postgres service

### 1. Start PostgreSQL

```bash
docker compose up -d postgres
```

### 2. Create the table

Apply `schema/table.sql` to your database. It uses `gen_random_uuid()` for the primary key.

### 3. Configure the environment

Create a `.env` file in the project root. Variables read by `internal/config/env.go`:

| Variable | Default |
| --- | --- |
| `APP_ENV` | `development` |
| `PORT` | `8080` |
| `DB_HOST` | `localhost` |
| `DB_PORT` | `5432` |
| `DB_USER` | `postgres` |
| `DB_PASSWORD` | (empty) |
| `DB_NAME` | (empty) |

Note: the server currently listens on `:8080` regardless of `PORT`.

### 4. Run

```bash
make run      # go run ./cmd/api
make build    # builds bin/api
make test     # go test ./...
make tidy     # go mod tidy
make clean    # removes bin/
```

The API is then available at `http://localhost:8080/api/v1`.

## API Endpoints

Base path: `/api/v1`

| Method | Endpoint | Description | Body |
| --- | --- | --- | --- |
| `GET` | `/health` | Health check; optional `?name=` | - |
| `GET` | `/users` | List all users, ordered by name | - |
| `GET` | `/users/:id` | Get a user by UUID | - |
| `POST` | `/users` | Create a user | `{ "name", "email", "password" }` |
| `PUT` | `/users/:id` | Update name and email | `{ "name", "email" }` |
| `DELETE` | `/users/:id` | Delete a user | - |

Example:

```bash
curl -X POST http://localhost:8080/api/v1/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Jane","email":"jane@example.com","password":"secret123"}'
```

## Author

**Aryan Bhandari**: [GitHub](https://github.com/aryanbha123)
