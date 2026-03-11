# My Time Garden — CLAUDE.md

This is the project constitution for Claude Code. Read this file before starting any task.
Keep it lean: refer to `agent_docs/` for deeper detail on specific topics.

---

## Project Overview

**My Time Garden** is a SaaS time-allocation web app.
Users plan and track time across five life categories: Housekeeping, English Learning,
Professional Learning, Gardening, and Child-Rearing.

Full specification: [`agent_docs/specification.md`](agent_docs/specification.md)

---

## Repository Structure

```
/
├── backend/                  # .NET 8 Web API (C#)
│   ├── MyTimeGarden.Api/     # Controllers, Program.cs, appsettings
│   ├── MyTimeGarden.Core/    # Domain entities, interfaces, DTOs
│   ├── MyTimeGarden.Infrastructure/  # EF Core, repositories, AI providers
│   └── MyTimeGarden.Tests/   # xUnit tests
├── frontend/                 # React 18 + TypeScript (Vite)
│   ├── src/
│   │   ├── features/         # Feature-based folders (auth, planner, ai, etc.)
│   │   ├── components/       # Shared UI components
│   │   ├── hooks/            # Custom React hooks
│   │   ├── services/         # Axios API client, push service
│   │   └── types/            # Shared TypeScript interfaces
│   └── public/
├── agent_docs/               # Deep-dive docs for Claude to read on demand
│   ├── specification.md      # Full app spec
│   ├── database_schema.md    # Full DB schema with column types
│   ├── api_contracts.md      # All REST endpoint shapes
│   ├── frontend_components.md # Component contracts and state design
│   └── deployment.md         # Cloud infra and CI/CD details
├── docker-compose.yml        # Local dev: API + DB + Redis
└── CLAUDE.md                 # ← you are here
```

---

## Common Commands

### Backend

```bash
# Restore and build
cd backend && dotnet restore && dotnet build

# Run API locally (uses docker-compose for DB + Redis)
cd backend && dotnet run --project MyTimeGarden.Api

# Run all tests
cd backend && dotnet test

# Run tests with coverage
cd backend && dotnet test --collect:"XPlat Code Coverage"

# Add EF Core migration
cd backend && dotnet ef migrations add <MigrationName> --project MyTimeGarden.Infrastructure --startup-project MyTimeGarden.Api

# Apply migrations
cd backend && dotnet ef database update --project MyTimeGarden.Infrastructure --startup-project MyTimeGarden.Api
```

### Frontend

```bash
# Install dependencies
cd frontend && npm install

# Start dev server
cd frontend && npm run dev

# Type check
cd frontend && npm run typecheck

# Run tests
cd frontend && npm run test

# Build for production
cd frontend && npm run build
```

### Docker (local dev)

```bash
# Start DB + Redis
docker-compose up -d db redis

# Start everything
docker-compose up

# Reset DB (drops volume)
docker-compose down -v
```

---

## Architecture Decisions

- **Backend**: .NET 8, ASP.NET Core Web API, EF Core (code-first), PostgreSQL, Redis
- **Frontend**: React 18, TypeScript strict mode, Vite, Zustand, TanStack Query, shadcn/ui, Tailwind CSS
- **Auth**: OAuth 2.0 (Google primary), JWT access tokens (15 min) + refresh tokens (30 days)
- **AI**: Pluggable via `IAiProvider` — never call AI APIs directly from frontend; always proxy through backend
- **API style**: REST only; versioned under `/v1`
- **No raw SQL**: use EF Core and LINQ; only add raw SQL if a query cannot be expressed otherwise, and document why

---

## Code Conventions

### Backend (C#)

- Follow standard C# naming: `PascalCase` for types and methods, `camelCase` for locals
- Use `record` types for DTOs and value objects
- Repositories return domain entities; controllers receive/return DTOs
- All controller actions must be `async`
- Validate all incoming DTOs with FluentValidation — never validate in controllers directly
- Use `Result<T>` or throw domain exceptions — do not return `null` from service methods
- Log with `ILogger<T>` using structured logging (Serilog); never use `Console.WriteLine`
- Every public service method must have a corresponding unit test

### Frontend (TypeScript / React)

- Functional components only; no class components
- All files in `features/` follow the pattern: `FeatureName/index.tsx`, `FeatureName/hooks.ts`, `FeatureName/types.ts`
- Use TanStack Query for all server state — do not store API responses in Zustand
- Zustand is for UI-only global state (auth user, theme, etc.)
- Prefer `zod` schemas co-located with form components for validation
- No `any` — use `unknown` and narrow types explicitly
- Component props interfaces are named `<ComponentName>Props`

---

## Environment Variables

### Backend (`appsettings.Development.json` or env)

```
ConnectionStrings__Postgres     PostgreSQL connection string
ConnectionStrings__Redis        Redis connection string
Jwt__Secret                     JWT signing secret (min 32 chars)
Jwt__Issuer                     Token issuer
Jwt__Audience                   Token audience
OAuth__Google__ClientId         Google OAuth client ID
OAuth__Google__ClientSecret     Google OAuth client secret
AiProvider__Name                anthropic | openai | azure-openai
AiProvider__Model               Model name
AiProvider__ApiKey              AI provider API key
Vapid__PublicKey                VAPID public key (push notifications)
Vapid__PrivateKey               VAPID private key
Vapid__Subject                  mailto: address
```

### Frontend (`.env.local`)

```
VITE_API_BASE_URL               e.g. http://localhost:5000/v1
VITE_VAPID_PUBLIC_KEY           VAPID public key (same as backend)
```

**Never commit secrets. All sensitive values are loaded from the environment or Secrets Manager.**

---

## Testing Expectations

- Every new service method → at least one unit test in `MyTimeGarden.Tests`
- Every new API endpoint → at least one integration test using `WebApplicationFactory`
- Frontend: new hooks get Vitest unit tests; new pages get at minimum a render smoke test
- Do not skip tests to make a feature "done faster" — flag blockers instead

---

## Git Workflow

- Never commit directly to `main`
- Branch naming: `feature/<short-description>`, `fix/<short-description>`, `chore/<short-description>`
- Commit messages follow Conventional Commits: `feat:`, `fix:`, `chore:`, `docs:`, `test:`
- Each logical change is its own commit — do not batch unrelated changes
- Run `dotnet build` and `npm run typecheck` before committing

---

## Security Rules

- All API endpoints (except `/auth/*`) require a valid JWT — never skip `[Authorize]`
- All data queries must be scoped to the authenticated `userId` from the JWT claim
- Admin endpoints must additionally check `role == "admin"`
- Never log tokens, passwords, or PII
- Never expose AI provider keys to the frontend

---

## Deep-Dive References

Read these files when working on the relevant area — do not load all of them upfront:

| Topic | File |
|---|---|
| Full app specification | `agent_docs/specification.md` |
| Database schema details | `agent_docs/database_schema.md` |
| API contracts / shapes | `agent_docs/api_contracts.md` |
| Frontend component design | `agent_docs/frontend_components.md` |
| Cloud deployment & CI/CD | `agent_docs/deployment.md` |
