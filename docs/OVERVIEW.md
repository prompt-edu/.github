# prompt-edu Organization Overview

> This document provides a high-level overview of the **prompt-edu** GitHub organization: what we build, how the pieces fit together, and where to find what you need.

PROMPT is developed at the [Research Group Applied Education Technologies (AET)](https://www.aet.cit.tum.de/) at the Technical University of Munich. It is an open-source, modular course management platform for project-based university teaching.

---

## Organization Structure

The organization is split into four repositories, each with a focused responsibility:

```text
prompt-edu/
├── prompt                  # Main platform: shell app + core platform services
├── prompt-intro-course     # Standalone intro-course module (client + server)
├── prompt-lib              # Shared frontend NPM packages
└── prompt-sdk              # Shared Go SDK for backend services
```

---

## Repository Details

### [prompt](https://github.com/prompt-edu/prompt)

The core PROMPT platform. Contains:

- **`clients/core/`** — React shell application (Webpack Module Federation host, port 3000)
- **`clients/shared_library/`** — Shared hooks, components, API utilities for all micro-frontends
- **`clients/*_component/`** — Independent micro-frontend modules (interview, matching, assessment, team allocation, etc.)
- **`servers/core/`** — Main Go service (course management, routing, auth, port 8080)
- **`servers/interview/`**, **`servers/team_allocation/`**, **`servers/assessment/`**, etc. — Independent backend microservices
- **`docs/`** — Docusaurus documentation site

**Live instance**: [https://prompt.aet.cit.tum.de/](https://prompt.aet.cit.tum.de/)
**Docs**: [https://prompt-edu.github.io/prompt/](https://prompt-edu.github.io/prompt/) _(hosted under `ls1intum.github.io` for historical reasons; the project org is `prompt-edu`)_
**Development guide**: [`AGENTS.md`](https://github.com/prompt-edu/prompt/blob/main/AGENTS.md)

---

### [prompt-intro-course](https://github.com/prompt-edu/prompt-intro-course)

A standalone repository for the intro programming course module, extracted from the main platform:

- **`client/`** — Intro-course developer micro-frontend (Webpack Module Federation remote, port 3005)
- **`server/`** — Intro-course Go backend service (port 8082): developer profiles, seat plans, tutor management, GitLab infrastructure setup

This repo has its own CI/CD pipeline and Docker Compose for independent deployment.

**Development guide**: [`AGENTS.md`](https://github.com/prompt-edu/prompt-intro-course/blob/main/AGENTS.md)

---

### [prompt-lib](https://github.com/prompt-edu/prompt-lib)

Shared frontend NPM packages used by all PROMPT micro-frontends:

| Package                        | Purpose                                                                                       |
| ------------------------------ | --------------------------------------------------------------------------------------------- |
| `@tumaet/prompt-ui-components` | shadcn/ui-based design system: buttons, tables, dialogs, editors, page layouts                |
| `@tumaet/prompt-shared-state`  | Domain types, enums (`Role`, `PassStatus`), Zustand stores (`useAuthStore`, `useCourseStore`) |

Both packages are published to npm on GitHub release.

---

### [prompt-sdk](https://github.com/prompt-edu/prompt-sdk)

Shared Go SDK used by all PROMPT backend services:

| Feature             | Description                                                               |
| ------------------- | ------------------------------------------------------------------------- |
| Auth middleware     | Keycloak-based JWT verification for Gin routes, role-aware access control |
| Resolution helpers  | Fetch and merge participation/phase data from the Core service            |
| CORS middleware     | Configurable CORS setup for Gin                                           |
| Standard endpoints  | Uniform config and copy endpoints for course phase modules                |
| Shared domain types | `Person`, `Student`, `Team`, `MetaData`, and more                         |
| Utilities           | `GetEnv`, `DeferDBRollback`, structured JSON fetching, custom validators  |

---

## Technology Stack at a Glance

| Layer              | Technology                           |
| ------------------ | ------------------------------------ |
| Frontend framework | React 19, TypeScript 5.9             |
| Frontend build     | Webpack 5, Module Federation         |
| UI                 | Tailwind CSS v4, shadcn/ui, Radix UI |
| State / data       | Zustand, TanStack React Query        |
| Backend language   | Go 1.26                              |
| Backend framework  | Gin                                  |
| Database           | PostgreSQL (pgx driver)              |
| DB queries         | sqlc (type-safe query generation)    |
| Migrations         | golang-migrate                       |
| Auth               | Keycloak (OIDC / RBAC)               |
| Container          | Docker Compose v2                    |

---

## Architecture Overview

PROMPT uses a **micro-frontend + microservices** architecture:

```text
Browser
  └── Core Shell (React, Module Federation host)
        ├── Dynamically loads micro-frontends (Module Federation remotes)
        │     ├── interview_component (port 3002)
        │     ├── matching_component (port 3003)
        │     ├── assessment_component (port 3007)
        │     ├── intro_course_developer_component (port 3005)  ← from prompt-intro-course
        │     └── ...
        └── shared_library (hooks, UI, API utils)

API Gateway / Reverse Proxy
  ├── /api/core/*             → servers/core (port 8080)
  ├── /api/interview/*        → servers/interview (port 8087)
  ├── /api/intro_course/*     → prompt-intro-course server (port 8082)
  └── ...

Identity Provider: Keycloak
```

Each micro-frontend is an independently deployable Webpack Module Federation remote. Each backend service has its own PostgreSQL database and uses the shared `prompt-sdk` for authentication.

---

## Quick Start

### Prerequisites

- Docker & Docker Compose v2
- Node.js + Yarn (for frontend work)
- Go 1.26+ (for backend work)

### Run the full platform locally

```bash
# From the prompt/ repository

# 1. Set up environment
cp .env.template .env
cp .env.dev.template .env.dev

# 2. Start database + Keycloak
make db

# 3. Start all micro-frontends
make clients

# 4. Start core server
make server
```

For module-specific setup, see the README or AGENTS.md in the relevant repository.

---

## Key Conventions

- **Backend routes** must include `/api/course_phase/:coursePhaseID/` for the SDK auth middleware to resolve course-phase roles.
- **Frontend imports** from shared_library use the `@/` alias; imports from prompt-lib use `@tumaet/prompt-ui-components` or `@tumaet/prompt-shared-state`.
- **New course phase modules** should follow the template in `clients/template_component/` and `servers/template_server/`.
- Generated sqlc code must be kept in sync with SQL query files after any schema or query changes.

---

## Further Reading

- [PROMPT AGENTS.md](https://github.com/prompt-edu/prompt/blob/main/AGENTS.md) — Full architecture, code conventions, patterns, commands
- [PROMPT Intro Course AGENTS.md](https://github.com/prompt-edu/prompt-intro-course/blob/main/AGENTS.md) — Intro-course specific conventions
- [PROMPT Documentation Portal](https://prompt-edu.github.io/prompt/) — User and contributor documentation
- [AET Research Group](https://www.aet.cit.tum.de/) — Parent research group at TUM
- [Discord](https://discord.gg/eybNUqD8gf) — Community and support
