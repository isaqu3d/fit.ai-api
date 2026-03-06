# fit.ai — API

REST API for an AI-powered personal fitness trainer 🏋️

![Node.js](https://img.shields.io/badge/Node.js-24.x-339933?style=for-the-badge&logo=node.js&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5.9-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![Fastify](https://img.shields.io/badge/Fastify-5.x-000000?style=for-the-badge&logo=fastify&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-7.x-2D3748?style=for-the-badge&logo=prisma&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-17-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Zod](https://img.shields.io/badge/Zod-4.x-3E67B1?style=for-the-badge&logo=zod&logoColor=white)
![pnpm](https://img.shields.io/badge/pnpm-10.x-F69220?style=for-the-badge&logo=pnpm&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

---

## 🚀 About

**fit.ai API** is the backend powering an intelligent fitness application. It allows users to create personalized workout plans, track workout sessions, and interact with an AI personal trainer that generates and adjusts training programs based on individual goals and physical data.

---

## 🛠 Technologies

| Technology | Purpose |
|---|---|
| **Node.js 24** | Runtime (ES Modules) |
| **TypeScript 5.9** | Type-safe development |
| **Fastify 5** | High-performance HTTP framework |
| **Prisma 7** | ORM with PostgreSQL (pg driver adapter) |
| **PostgreSQL** | Relational database |
| **better-auth** | Authentication (email/password, sessions) |
| **Zod v4** | Schema validation and serialization |
| **AI SDK + OpenAI** | AI personal trainer integration |
| **Docker** | Local PostgreSQL via docker-compose |
| **Scalar** | Interactive API documentation UI |

---

## 📁 Project Structure

```
fit.ai-api/
├── prisma/
│   ├── schema.prisma          # Database schema and models
│   └── migrations/            # SQL migration history
├── src/
│   ├── index.tsx              # App entry point — registers plugins and routes
│   ├── lib/
│   │   ├── auth.ts            # better-auth configuration
│   │   └── db.ts              # Prisma client singleton
│   ├── routes/                # Fastify route handlers (one file per resource)
│   │   ├── ai.ts
│   │   ├── home.ts
│   │   ├── me.ts
│   │   ├── stats.ts
│   │   └── workout-plan.ts
│   ├── usecases/              # Business logic (one class per use case)
│   │   ├── CreateWorkoutPlan.ts
│   │   ├── GetHomeData.ts
│   │   ├── GetStats.ts
│   │   ├── GetUserTrainData.ts
│   │   ├── GetWorkoutDay.ts
│   │   ├── GetWorkoutPlan.ts
│   │   ├── ListWorkoutPlans.ts
│   │   ├── StartWorkoutSession.ts
│   │   ├── UpdateWorkoutSession.ts
│   │   └── UpsertUserTrainData.ts
│   ├── schemas/
│   │   └── index.ts           # Shared Zod schemas
│   ├── errors/
│   │   └── index.ts           # Custom error classes
│   └── generated/
│       └── prisma/            # Auto-generated Prisma client (do not edit)
├── docker-compose.yml
├── .env.example
└── package.json
```

### Request Flow

```
Request → Fastify Route → Auth Check → Use Case → Prisma → Response
```

---

## 🗄️ Database Diagram

```
┌─────────────────────────────┐
│            User             │
├─────────────────────────────┤
│ id (PK)                     │
│ name                        │
│ email (unique)              │
│ emailVerified               │
│ image?                      │
│ weightInGrams?              │
│ heightInCentimeters?        │
│ age?                        │
│ bodyFatPercentage?          │
│ createdAt / updatedAt       │
└──────────────┬──────────────┘
               │ 1:N
               ▼
┌─────────────────────────────┐
│         WorkoutPlan         │
├─────────────────────────────┤
│ id (PK)                     │
│ name                        │
│ userId (FK → User)          │
│ isActive                    │
│ createdAt / updatedAt       │
└──────────────┬──────────────┘
               │ 1:N
               ▼
┌─────────────────────────────┐
│          WorkoutDay         │
├─────────────────────────────┤
│ id (PK)                     │
│ name                        │
│ workoutPlanId (FK)          │
│ weekDay (enum WeekDay)      │
│ isRest                      │
│ estimatedDurationInSeconds  │
│ coverImageUrl?              │
│ createdAt / updatedAt       │
└──────┬───────────┬──────────┘
       │ 1:N       │ 1:N
       ▼           ▼
┌──────────────┐  ┌────────────────────┐
│WorkoutExercise│  │  WorkoutSession    │
├──────────────┤  ├────────────────────┤
│ id (PK)      │  │ id (PK)            │
│ name         │  │ workoutDayId (FK)  │
│ order        │  │ startedAt          │
│ workoutDayId │  │ completedAt?       │
│ sets         │  │ createdAt          │
│ reps         │  │ updatedAt          │
│ restTime(s)  │  └────────────────────┘
│ createdAt    │
│ updatedAt    │
└──────────────┘

Auth tables (managed by better-auth):
  Session · Account · Verification
```

---

## ⚙️ How to Run

### Prerequisites

- Node.js 24+
- pnpm 10+
- Docker

### 1. Clone the repository

```sh
git clone git@github.com:isaqu3d/fit.ai-api.git
cd fit.ai-api
```

### 2. Install dependencies

```sh
pnpm install
```

### 3. Set up environment variables

```sh
cp .env.example .env
```

Edit `.env` and fill in the required values:

```env
PORT=
DATABASE_URL=
BETTER_AUTH_SECRET=
BETTER_AUTH_URL=
```

### 4. Start the database

```sh
docker compose up -d
```

### 5. Run database migrations

```sh
pnpm prisma migrate dev
```

### 6. Start the development server

```sh
pnpm dev
```

The API will be available at `http://localhost:4949`.
Interactive API docs (Scalar) will be at `http://localhost:4949/docs`.

---

## 📖 API Documentation

With the server running, access the full interactive API reference at:

```
http://localhost:4949/docs
```

It includes both the application endpoints and the better-auth authentication routes.

---

## 🔐 Authentication

Authentication is handled by **better-auth** using email/password strategy. Protected routes require a valid session cookie obtained after signing in via the `/api/auth/*` endpoints.

---

## 📝 License

[MIT License](LICENSE)

Made by [Isaque de Sousa](https://github.com/isaqu3d) — give a ⭐️!
