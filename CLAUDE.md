# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Start development server with hot reload
pnpm dev

# Lint
npx eslint .

# Database
docker compose up -d                        # Start PostgreSQL
npx prisma migrate dev                      # Run migrations
npx prisma generate                         # Regenerate Prisma client (after schema changes)
npx prisma studio                           # Database GUI
```

> No test suite is configured yet.

## Architecture

Fastify REST API with Zod validation, Prisma ORM (PostgreSQL), and better-auth for authentication.

### Request Flow

```
Request → Fastify route → better-auth session check → Use Case class → Prisma → Response
```

- **`src/index.tsx`** — App entry point. Registers plugins (CORS, Swagger, Scalar docs), mounts route groups, and proxies all `/api/auth/*` requests to better-auth.
- **`src/routes/`** — Fastify route handlers. Each file exports an async plugin function registered with a URL prefix. Routes use `ZodTypeProvider` for typed request/response validation.
- **`src/usecases/`** — Business logic classes with a single `execute(dto)` method. They interact directly with `prisma`.
- **`src/schemas/index.ts`** — Shared Zod schemas (`WorkoutPlanSchema`, `ErrorSchema`) used in both route definitions and response serialization.
- **`src/errors/index.ts`** — Custom error classes (e.g., `NotFoundError`) caught in route handlers and mapped to HTTP status codes.
- **`src/lib/auth.ts`** — better-auth configuration (email/password, Prisma adapter, OpenAPI plugin).
- **`src/lib/db.ts`** — Singleton Prisma client using `@prisma/adapter-pg` (driver adapters mode).
- **`src/generated/prisma/`** — Auto-generated Prisma client output. Do not edit manually.

### Key Conventions

- **Adding a route**: create a file in `src/routes/`, export an async Fastify plugin, register it in `src/index.tsx` with `app.register(...)`.
- **Business logic**: goes in a use case class under `src/usecases/`, not inline in route handlers.
- **Validation/serialization**: always declare `schema.body` and `schema.response` on routes using schemas from `src/schemas/` or inline Zod objects. `fastify-type-provider-zod` handles both validation and serialization.
- **Auth**: routes requiring authentication call `auth.api.getSession({ headers: fromNodeHeaders(request.headers) })` and return 401 if no session.
- **Error handling**: routes catch known error types (e.g., `NotFoundError`) and return structured `{ error, code }` responses matching `ErrorSchema`.
- **Prisma client**: generated to `src/generated/prisma/` (not the default location). Import from `../generated/prisma/client.js`.
- **Imports**: must use `.js` extensions (ESM with `"type": "module"`). Import sorting is enforced by ESLint (`simple-import-sort`).

### Data Model

`User` → `WorkoutPlan` (one active per user) → `WorkoutDay` → `WorkoutExercise`
`WorkoutDay` also has `WorkoutSession` records tracking actual workout completions.
Auth tables (`Session`, `Account`, `Verification`) are managed by better-auth.

### Environment Variables

```
PORT=
DATABASE_URL=
BETTER_AUTH_SECRET=
BETTER_AUTH_URL=
```

### API Docs

Available at `http://localhost:8081/docs` when running locally (Scalar UI with both app and auth OpenAPI specs).
