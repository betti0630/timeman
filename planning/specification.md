# My Time Garden — Application Specification

**Version:** 1.0
**Date:** March 2026
**Architecture:** .NET 8 · React 18 · PostgreSQL
**Deployment:** Cloud (AWS / Azure / GCP)

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [System Architecture](#2-system-architecture)
3. [Technology Stack](#3-technology-stack)
4. [Database Design](#4-database-design)
5. [Backend API Specification](#5-backend-api-specification)
6. [Frontend Specification](#6-frontend-specification)
7. [Authentication & Authorisation](#7-authentication--authorisation)
8. [AI Assistant Integration](#8-ai-assistant-integration)
9. [Notifications](#9-notifications)
10. [Admin Dashboard](#10-admin-dashboard)
11. [Analytics & Usage Stats](#11-analytics--usage-stats)
12. [Non-Functional Requirements](#12-non-functional-requirements)
13. [Cloud Deployment](#13-cloud-deployment)
14. [Security](#14-security)
15. [Testing Strategy](#15-testing-strategy)
16. [Project Milestones](#16-project-milestones)

---

## 1. Project Overview

### 1.1 Purpose

My Time Garden is a SaaS personal time-allocation and life-management web application. It helps users plan, track, and reflect on time spent across five life domains: **Housekeeping**, **English Learning**, **Professional Learning**, **Gardening**, and **Child-Rearing**. The name is extensible — administrators can configure categories per-user in future iterations.

### 1.2 Goals

- Allow users to set weekly/daily time goals per category
- Log time actually spent on each category
- Visualise progress via charts and summaries
- Provide an AI-powered assistant for scheduling suggestions, progress review, and motivation
- Deliver browser push notifications as reminders
- Support many concurrent users via a scalable cloud-hosted SaaS architecture

### 1.3 Scope (V1)

| In Scope | Out of Scope |
|---|---|
| OAuth social login | Native mobile apps |
| Daily & weekly planner views | Payments / subscriptions |
| Drag-and-drop priority management | Team / shared accounts |
| Time tracking & goal setting | Calendar sync (Google Cal etc.) |
| Progress charts | Offline-first PWA |
| AI assistant chat | Custom category creation (V2) |
| Browser push notifications | File uploads |
| Admin dashboard | |
| Analytics & usage stats | |

---

## 2. System Architecture

### 2.1 High-Level Overview

```
┌─────────────────────────────────────────────────────┐
│                    Browser Client                    │
│              React 18 SPA (Vite + TS)               │
└───────────────────────┬─────────────────────────────┘
                        │ HTTPS / REST
┌───────────────────────▼─────────────────────────────┐
│               .NET 8 Web API (C#)                   │
│   Controllers · Services · Repositories · EF Core  │
│                                                     │
│  ┌──────────┐  ┌───────────┐  ┌──────────────────┐ │
│  │ Auth     │  │ AI Proxy  │  │ Push Notif.      │ │
│  │ (OAuth)  │  │ Service   │  │ Service          │ │
│  └──────────┘  └───────────┘  └──────────────────┘ │
└───────────────────────┬─────────────────────────────┘
                        │
          ┌─────────────┴──────────────┐
          │                            │
┌─────────▼──────────┐    ┌───────────▼────────────┐
│   PostgreSQL DB    │    │   External AI API      │
│  (AWS RDS / etc.)  │    │  (OpenAI / Claude /    │
└────────────────────┘    │   Azure OpenAI)        │
                          └────────────────────────┘
```

### 2.2 Deployment Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Cloud Provider                    │
│                                                     │
│  CDN (CloudFront / Azure CDN)                       │
│    └── Static React build (S3 / Blob Storage)       │
│                                                     │
│  Load Balancer                                      │
│    └── App Service / ECS / Container Apps           │
│          └── .NET 8 API (Docker container)          │
│                                                     │
│  Managed PostgreSQL (RDS / Azure DB / Cloud SQL)    │
│  Secrets Manager (API keys, OAuth secrets)          │
│  Redis (session cache, rate limiting)               │
└─────────────────────────────────────────────────────┘
```

---

## 3. Technology Stack

### 3.1 Backend

| Component | Technology | Notes |
|---|---|---|
| Runtime | .NET 8 | LTS release |
| Language | C# 12 | |
| Web framework | ASP.NET Core Web API | Minimal API or controller-based |
| ORM | Entity Framework Core 8 | Code-first migrations |
| Database | PostgreSQL 15+ | Via Npgsql EF provider |
| Auth | ASP.NET Core Identity + OAuth | Google, GitHub, Microsoft providers |
| JWT | System.IdentityModel.Tokens.Jwt | Access + refresh token pattern |
| Caching | Redis (StackExchange.Redis) | Session, rate limiting |
| Validation | FluentValidation | Request DTO validation |
| Logging | Serilog | Structured logging to cloud sink |
| API docs | Swagger / Scalar (NSwag) | Auto-generated from attributes |
| Mapping | AutoMapper or Mapster | Entity ↔ DTO |
| Background jobs | Hangfire or Quartz.NET | Scheduled notification dispatch |
| Push notifications | Web Push (WebPush-NetCore) | VAPID protocol |
| Testing | xUnit + Moq + TestContainers | Unit + integration tests |

### 3.2 Frontend

| Component | Technology | Notes |
|---|---|---|
| Framework | React 18 | Functional components + hooks |
| Language | TypeScript 5 | Strict mode |
| Build tool | Vite | Fast dev + optimised builds |
| State management | Zustand | Lightweight, no boilerplate |
| Server state | TanStack Query (React Query) | API caching, background refetch |
| Routing | React Router v6 | |
| UI components | shadcn/ui + Tailwind CSS | Accessible, customisable |
| Charts | Recharts | Progress visualisation |
| Drag and drop | @dnd-kit/core | Priority reordering |
| Forms | React Hook Form + Zod | Validation mirrors backend |
| HTTP client | Axios | Interceptors for auth |
| Notifications | web-push + Service Worker | Browser push |
| Testing | Vitest + React Testing Library | |

### 3.3 Infrastructure

| Component | Technology |
|---|---|
| Container | Docker + Docker Compose (dev) |
| CI/CD | GitHub Actions |
| Cloud | AWS / Azure / GCP (TBD) |
| IaC | Terraform or Bicep |
| Monitoring | Application Insights / CloudWatch |
| Secrets | AWS Secrets Manager / Azure Key Vault |

---

## 4. Database Design

### 4.1 Entity Relationship Overview

```
Users ──< UserCategories ──< TimeEntries
  │               │
  │         CategoryGoals
  │
  ├──< AiConversations ──< AiMessages
  │
  ├──< PushSubscriptions
  │
  └──< UserAnalyticsEvents
```

### 4.2 Table Definitions

#### `users`

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK, default gen_random_uuid() |
| email | VARCHAR(255) | UNIQUE, NOT NULL |
| display_name | VARCHAR(100) | NOT NULL |
| avatar_url | TEXT | NULLABLE |
| oauth_provider | VARCHAR(50) | e.g. 'google', 'github' |
| oauth_subject | VARCHAR(255) | Provider's user ID |
| role | VARCHAR(20) | 'user' or 'admin', default 'user' |
| timezone | VARCHAR(50) | e.g. 'Europe/London', default 'UTC' |
| created_at | TIMESTAMPTZ | default now() |
| updated_at | TIMESTAMPTZ | default now() |
| last_login_at | TIMESTAMPTZ | NULLABLE |
| is_active | BOOLEAN | default true |

#### `categories`

Seed data — the five life domains (extensible).

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| slug | VARCHAR(50) | UNIQUE e.g. 'housekeeping' |
| label | VARCHAR(100) | e.g. 'Housekeeping' |
| emoji | VARCHAR(10) | e.g. '🏠' |
| color_hex | CHAR(6) | e.g. 'e07b5a' |
| sort_order | INTEGER | default display order |
| is_active | BOOLEAN | default true |

#### `user_category_settings`

Per-user priority and goal configuration per category.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| user_id | UUID | FK → users.id |
| category_id | UUID | FK → categories.id |
| priority_rank | INTEGER | 1 = highest priority |
| daily_goal_minutes | INTEGER | default 30 |
| weekly_goal_minutes | INTEGER | default 180 |
| updated_at | TIMESTAMPTZ | default now() |

Unique constraint on `(user_id, category_id)`.

#### `time_entries`

Individual time-logging events.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| user_id | UUID | FK → users.id |
| category_id | UUID | FK → categories.id |
| logged_date | DATE | The day this entry belongs to |
| duration_minutes | INTEGER | NOT NULL, > 0 |
| note | TEXT | NULLABLE, user memo |
| created_at | TIMESTAMPTZ | default now() |

Index on `(user_id, logged_date)`.

#### `ai_conversations`

Chat sessions with the AI assistant.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| user_id | UUID | FK → users.id |
| title | VARCHAR(200) | Auto-generated summary |
| created_at | TIMESTAMPTZ | default now() |
| updated_at | TIMESTAMPTZ | default now() |

#### `ai_messages`

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| conversation_id | UUID | FK → ai_conversations.id |
| role | VARCHAR(20) | 'user' or 'assistant' |
| content | TEXT | NOT NULL |
| created_at | TIMESTAMPTZ | default now() |

#### `push_subscriptions`

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| user_id | UUID | FK → users.id |
| endpoint | TEXT | UNIQUE, NOT NULL |
| p256dh_key | TEXT | NOT NULL |
| auth_key | TEXT | NOT NULL |
| user_agent | TEXT | NULLABLE |
| created_at | TIMESTAMPTZ | default now() |

#### `analytics_events`

Lightweight event stream for usage analytics.

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PK |
| user_id | UUID | FK → users.id, NULLABLE |
| event_type | VARCHAR(100) | e.g. 'time_logged', 'ai_message_sent' |
| payload | JSONB | Event-specific data |
| created_at | TIMESTAMPTZ | default now() |

Index on `(event_type, created_at)`.

---

## 5. Backend API Specification

Base URL: `https://api.mytimegarden.com/v1`

All endpoints (except auth) require `Authorization: Bearer <access_token>`.

### 5.1 Authentication — `/auth`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/auth/login/{provider}` | Redirect to OAuth provider (Google etc.) |
| GET | `/auth/callback/{provider}` | OAuth callback, returns JWT tokens |
| POST | `/auth/refresh` | Refresh access token |
| POST | `/auth/logout` | Revoke refresh token |
| GET | `/auth/me` | Current user profile |
| PUT | `/auth/me` | Update display name, timezone |

**Token Response:**
```json
{
  "accessToken": "eyJ...",
  "refreshToken": "abc123...",
  "expiresIn": 3600,
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "displayName": "Jane",
    "avatarUrl": "https://...",
    "role": "user",
    "timezone": "Europe/Budapest"
  }
}
```

### 5.2 Categories — `/categories`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/categories` | List all active categories |

### 5.3 User Settings — `/settings`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/settings/categories` | Get all user category settings (goals + priority) |
| PUT | `/settings/categories` | Bulk update goals and priority ranks |
| PUT | `/settings/categories/{categoryId}` | Update single category setting |

**PUT `/settings/categories` Request Body:**
```json
[
  { "categoryId": "uuid", "priorityRank": 1, "dailyGoalMinutes": 40, "weeklyGoalMinutes": 240 },
  { "categoryId": "uuid", "priorityRank": 2, "dailyGoalMinutes": 50, "weeklyGoalMinutes": 300 }
]
```

### 5.4 Time Entries — `/time-entries`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/time-entries` | Query entries (filter by date range) |
| POST | `/time-entries` | Log a new time entry |
| PUT | `/time-entries/{id}` | Edit an entry |
| DELETE | `/time-entries/{id}` | Delete an entry |

**GET `/time-entries` Query Parameters:**

| Param | Type | Example |
|---|---|---|
| from | date | 2026-03-01 |
| to | date | 2026-03-07 |
| categoryId | UUID | optional filter |

**POST `/time-entries` Request Body:**
```json
{
  "categoryId": "uuid",
  "loggedDate": "2026-03-10",
  "durationMinutes": 45,
  "note": "Cleaned kitchen and bathroom"
}
```

### 5.5 Analytics — `/analytics`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/analytics/daily?date=2026-03-10` | Time totals per category for one day |
| GET | `/analytics/weekly?weekStart=2026-03-09` | Time totals per category for a week |
| GET | `/analytics/summary?from=...&to=...` | Aggregated summary across date range |
| GET | `/analytics/streak/{categoryId}` | Current and best streak (days with any time logged) |

**Weekly Response Example:**
```json
{
  "weekStart": "2026-03-09",
  "weekEnd": "2026-03-15",
  "days": ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
  "categories": [
    {
      "categoryId": "uuid",
      "slug": "housekeeping",
      "label": "Housekeeping",
      "weeklyGoalMinutes": 240,
      "totalMinutes": 185,
      "progressPercent": 77.1,
      "dailyBreakdown": [40, 30, 0, 45, 30, 20, 20]
    }
  ]
}
```

### 5.6 AI Assistant — `/ai`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/ai/conversations` | List user's past conversations |
| POST | `/ai/conversations` | Start a new conversation |
| GET | `/ai/conversations/{id}` | Get conversation with all messages |
| POST | `/ai/conversations/{id}/messages` | Send a message, get AI reply |
| DELETE | `/ai/conversations/{id}` | Delete a conversation |

**POST `/ai/conversations/{id}/messages` Request Body:**
```json
{
  "content": "Can you suggest a schedule for today?"
}
```

The backend builds the system prompt server-side, injecting the user's current goals, priorities, and recent time log data before calling the configured AI provider. The AI provider key and model are read from server configuration — never exposed to the frontend.

**Response:**
```json
{
  "id": "uuid",
  "role": "assistant",
  "content": "Based on your priorities...",
  "createdAt": "2026-03-10T09:00:00Z"
}
```

### 5.7 Push Notifications — `/notifications`

| Method | Endpoint | Description |
|---|---|---|
| POST | `/notifications/subscribe` | Register a push subscription |
| DELETE | `/notifications/subscribe` | Unsubscribe current device |
| GET | `/notifications/settings` | Get user notification preferences |
| PUT | `/notifications/settings` | Update notification preferences |

**Notification Settings Object:**
```json
{
  "enabled": true,
  "reminderTimes": ["08:00", "20:00"],
  "reminderDays": ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
  "motivationalMessages": true
}
```

### 5.8 Admin — `/admin`

All admin endpoints require `role = 'admin'`.

| Method | Endpoint | Description |
|---|---|---|
| GET | `/admin/users` | List all users (paginated) |
| GET | `/admin/users/{id}` | Get user detail |
| PUT | `/admin/users/{id}/status` | Activate or deactivate a user |
| GET | `/admin/analytics/overview` | System-wide usage stats |
| GET | `/admin/analytics/events` | Raw analytics event stream |
| GET | `/admin/categories` | Manage global categories |
| POST | `/admin/categories` | Add a new category |
| PUT | `/admin/categories/{id}` | Edit a category |

---

## 6. Frontend Specification

### 6.1 Application Structure

```
src/
├── app/
│   ├── App.tsx               # Router + auth guard
│   ├── routes.tsx
│   └── store.ts              # Zustand global store
├── features/
│   ├── auth/                 # Login, OAuth callback, profile
│   ├── planner/              # Daily & weekly planner views
│   ├── tracker/              # Time logging UI
│   ├── progress/             # Charts and summaries
│   ├── priorities/           # Drag-and-drop priority manager
│   ├── ai/                   # AI assistant chat panel
│   ├── notifications/        # Push notification management
│   └── admin/                # Admin dashboard (role-gated)
├── components/
│   ├── ui/                   # shadcn/ui wrappers
│   ├── charts/               # Recharts wrappers
│   └── layout/               # Shell, nav, sidebar
├── hooks/                    # Shared custom hooks
├── services/
│   ├── api.ts                # Axios instance + interceptors
│   └── push.ts               # Service Worker registration
└── types/                    # Shared TypeScript interfaces
```

### 6.2 Pages & Routes

| Route | Component | Auth Required | Role |
|---|---|---|---|
| `/` | Redirect → `/planner` | Yes | user |
| `/login` | LoginPage | No | — |
| `/auth/callback` | OAuthCallbackPage | No | — |
| `/planner` | PlannerPage | Yes | user |
| `/progress` | ProgressPage | Yes | user |
| `/priorities` | PrioritiesPage | Yes | user |
| `/ai` | AiAssistantPage | Yes | user |
| `/settings` | SettingsPage | Yes | user |
| `/admin` | AdminDashboardPage | Yes | admin |
| `/admin/users` | AdminUsersPage | Yes | admin |
| `/admin/analytics` | AdminAnalyticsPage | Yes | admin |

### 6.3 Key Components

#### PlannerPage

- Tab toggle: **Day View** / **Week View**
- Day selector strip showing Mon–Sun with a dot indicator on days with logged time
- Per-category cards sorted by priority rank, each showing:
  - Emoji, label, priority badge
  - Circular radial progress ring (goal vs spent)
  - Linear progress bar
  - Quick-log buttons: +15m, +30m, +60m
  - Custom minute input field
- Week view shows a 7-column grid with per-day totals per category

#### ProgressPage

- Grouped bar chart (goal vs actual) per category — switchable daily/weekly
- Per-category summary rows with percentage completion
- Streak indicators per category
- Date range picker for custom reporting

#### PrioritiesPage

- Vertical drag-and-drop list of categories (`@dnd-kit/core`)
- Priority numbers update live on reorder
- Weekly time summary beneath the list
- Save button sends bulk update to API

#### AiAssistantPage

- Chat bubble interface
- System context (goals, priorities, recent logs) is assembled by the backend
- Conversation history persisted; list of past conversations in a sidebar
- Suggested prompts: "Plan today", "Review my week", "Motivate me"

#### AdminDashboardPage

- KPI cards: total users, active users (last 7d), time entries today, AI messages today
- User table with search, pagination, active/inactive toggle
- Charts: daily active users, time entries per category (last 30 days)
- Raw analytics event log with filters

### 6.4 State Management

| State | Location | Notes |
|---|---|---|
| Auth user / tokens | Zustand + localStorage | Tokens stored securely |
| Server data (API) | TanStack Query | Auto-invalidated on mutation |
| UI state (tabs, modals) | Local React state | Not persisted |
| Drag-and-drop order | Local state → API on save | Optimistic update |

---

## 7. Authentication & Authorisation

### 7.1 OAuth Flow

1. User clicks "Sign in with Google" on the login page
2. Frontend redirects to `GET /auth/login/google`
3. Backend redirects to Google's OAuth consent screen
4. Google redirects to `GET /auth/callback/google?code=...`
5. Backend exchanges code for Google tokens, upserts `users` row
6. Backend issues a short-lived **access token** (15 min) and long-lived **refresh token** (30 days) as JWTs
7. Frontend stores tokens; all API calls include `Authorization: Bearer <accessToken>`
8. On 401, frontend silently calls `/auth/refresh` and retries

### 7.2 Supported OAuth Providers (V1)

- Google
- GitHub (optional, easy to add with same pattern)
- Microsoft (optional)

### 7.3 Role-Based Access Control

| Role | Capabilities |
|---|---|
| `user` | Full access to own data only |
| `admin` | All user capabilities + admin endpoints |

Admin role is assigned manually in the database; no self-service promotion.

---

## 8. AI Assistant Integration

### 8.1 Provider Abstraction

The backend uses an `IAiProvider` interface, allowing the concrete provider to be swapped via configuration.

```csharp
public interface IAiProvider
{
    Task<string> CompleteAsync(string systemPrompt, IEnumerable<AiMessage> history, string userMessage);
}

// Implementations:
// - AnthropicClaudeProvider
// - OpenAiProvider
// - AzureOpenAiProvider
```

Provider selection is driven by `appsettings.json`:

```json
{
  "AiProvider": {
    "Name": "anthropic",
    "Model": "claude-sonnet-4-20250514",
    "ApiKey": "<from secrets manager>"
  }
}
```

### 8.2 System Prompt Construction

Before each API call, the backend assembles a context-rich system prompt:

```
You are a warm, encouraging personal time management assistant for My Time Garden.

User profile:
- Timezone: {timezone}
- Today: {localDate}

Category priorities (1 = most important):
{1. Housekeeping 🏠 — daily goal: 40m, weekly goal: 240m}
{2. English Learning 📚 — daily goal: 50m, weekly goal: 300m}
...

Today's logged time:
- Housekeeping: 25m / 40m goal (63%)
- English Learning: 0m / 50m goal (0%)
...

This week's totals:
...

Be concise, warm, and practical. Use emojis naturally.
```

### 8.3 Rate Limiting

- 50 AI messages per user per day (configurable)
- Enforced server-side via Redis counter
- 429 response when exceeded, with reset timestamp

---

## 9. Notifications

### 9.1 Browser Push (Web Push / VAPID)

1. On first login the frontend asks for notification permission
2. If granted, the Service Worker subscribes to the browser's push service and sends the subscription object to `POST /notifications/subscribe`
3. The backend stores `endpoint`, `p256dh`, and `auth` keys in `push_subscriptions`
4. A background job (Hangfire / Quartz) runs every hour and dispatches push notifications to users whose reminder time matches the current hour in their timezone

### 9.2 Notification Types

| Type | Trigger | Content Example |
|---|---|---|
| Daily reminder | Scheduled (user-configured time) | "Time to tend your garden! You haven't logged anything yet today 🌱" |
| Goal achieved | When 100% of daily goal reached | "You hit your Housekeeping goal today! 🏠✓" |
| Motivational | Once daily if configured | "Small steps every day add up to big changes 💪" |
| Weekly summary | Sunday evening | "This week you logged 3h 20m — your best yet! 📊" |

---

## 10. Admin Dashboard

### 10.1 Access

Accessible at `/admin`. Visible only to users with `role = 'admin'`. The nav item is hidden for regular users; the route is also server-enforced.

### 10.2 Features

**Overview KPIs:**
- Total registered users
- Active users (logged time in last 7 days)
- Time entries created today / this week
- AI messages sent today
- Push subscriptions active

**User Management:**
- Searchable, sortable, paginated user table
- Columns: name, email, joined date, last active, entry count, status
- Actions: view profile detail, deactivate / reactivate account

**Category Management:**
- View and edit global category labels, emojis, colours
- Reorder default sort order
- Activate / deactivate categories system-wide

---

## 11. Analytics & Usage Stats

### 11.1 Event Tracking

Key events written to `analytics_events`:

| Event Type | Payload |
|---|---|
| `user_registered` | `{ provider }` |
| `user_login` | `{ provider }` |
| `time_logged` | `{ categoryId, durationMinutes, loggedDate }` |
| `goal_updated` | `{ categoryId, oldGoal, newGoal }` |
| `priority_updated` | `{ newOrder[] }` |
| `ai_message_sent` | `{ conversationId, messageLength }` |
| `push_subscribed` | `{ userAgent }` |
| `notification_sent` | `{ type, success }` |

### 11.2 User-Facing Analytics (Progress Page)

- Daily and weekly bar charts (goal vs. spent per category)
- Streak counters (current streak, best streak)
- Personal all-time stats: total hours logged, favourite category

### 11.3 Admin Analytics

- Daily active users chart (last 30 days)
- Time logged per category (last 30 days, system-wide)
- AI usage per day
- Notification delivery success rate

---

## 12. Non-Functional Requirements

### 12.1 Performance

| Metric | Target |
|---|---|
| API response time (p95) | < 200ms for non-AI endpoints |
| AI response time (p95) | < 8 seconds (provider-dependent) |
| Frontend first contentful paint | < 1.5s on 4G |
| Database queries | All user-facing queries indexed, < 50ms |

### 12.2 Scalability

- Stateless API — horizontally scalable behind a load balancer
- Redis for session/cache — shared across instances
- Background jobs on a separate worker process
- Database connection pooling via PgBouncer or EF Core connection pool

### 12.3 Availability

- Target uptime: 99.5% monthly
- Health-check endpoint: `GET /health`
- Graceful shutdown with in-flight request draining

### 12.4 Internationalisation

- All timestamps stored as UTC; converted to user's timezone in the frontend
- Date formatting respects browser locale
- English only in V1; i18n architecture (`react-i18next`) added as a foundation

---

## 13. Cloud Deployment

### 13.1 Recommended Stack (AWS Example)

| Component | AWS Service |
|---|---|
| React SPA | S3 + CloudFront |
| .NET API | ECS Fargate (Docker) or App Service |
| PostgreSQL | Amazon RDS for PostgreSQL |
| Redis | Amazon ElastiCache |
| Secrets | AWS Secrets Manager |
| Logs | CloudWatch Logs |
| CI/CD | GitHub Actions → ECR → ECS |
| Domain / TLS | Route 53 + ACM |

### 13.2 Environments

| Environment | Purpose |
|---|---|
| `development` | Local Docker Compose |
| `staging` | Cloud deployment, mirrors prod, used for QA |
| `production` | Live environment |

### 13.3 Docker Compose (Development)

```yaml
services:
  api:
    build: ./backend
    ports: ["5000:8080"]
    environment:
      - ConnectionStrings__Postgres=Host=db;Database=timegarden;Username=pg;Password=pg
      - ConnectionStrings__Redis=redis:6379
    depends_on: [db, redis]

  frontend:
    build: ./frontend
    ports: ["3000:3000"]
    environment:
      - VITE_API_BASE_URL=http://localhost:5000/v1

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: timegarden
      POSTGRES_USER: pg
      POSTGRES_PASSWORD: pg
    volumes: [pgdata:/var/lib/postgresql/data]

  redis:
    image: redis:7-alpine

volumes:
  pgdata:
```

### 13.4 CI/CD Pipeline (GitHub Actions)

```
On push to main:
  1. Run backend tests (dotnet test)
  2. Run frontend tests (vitest)
  3. Build Docker image → push to container registry
  4. Run DB migrations (EF Core migrate)
  5. Deploy to staging → smoke tests
  6. Manual approval gate
  7. Deploy to production
```

---

## 14. Security

| Area | Measure |
|---|---|
| Transport | HTTPS everywhere; HSTS header |
| Auth tokens | Short-lived JWTs (15 min); refresh token rotation |
| CORS | Whitelist frontend origin only |
| SQL injection | EF Core parameterised queries; no raw SQL |
| XSS | React escapes output; CSP header |
| CSRF | Stateless JWT bearer (no cookies); SameSite on refresh token cookie |
| Rate limiting | Per-IP and per-user (Redis); AI endpoint separately limited |
| Secrets | Never in source code; loaded from Secrets Manager at runtime |
| OAuth | PKCE flow enforced; state parameter validated |
| Data isolation | All queries scoped to `userId` from JWT claim |
| Dependency scanning | Dependabot / `dotnet audit` / `npm audit` in CI |

---

## 15. Testing Strategy

### 15.1 Backend

| Layer | Framework | Coverage Target |
|---|---|---|
| Unit tests (services) | xUnit + Moq | 80% |
| Integration tests (API) | xUnit + WebApplicationFactory + TestContainers | All endpoints |
| DB tests | EF Core InMemory / TestContainers PostgreSQL | Repository layer |

### 15.2 Frontend

| Layer | Framework | Coverage Target |
|---|---|---|
| Unit tests (hooks, utils) | Vitest | 70% |
| Component tests | React Testing Library | Key components |
| E2E (smoke) | Playwright | Critical user journeys |

### 15.3 Critical User Journeys (E2E)

1. Sign in with Google → land on planner
2. Log 30 minutes to Housekeeping → see progress ring update
3. Reorder priorities → reload page → order persists
4. Send AI message → receive reply
5. Admin: log in as admin → view user list → deactivate a user

---

## 16. Project Milestones

| Milestone | Deliverables |
|---|---|
| **M1 — Foundation** | Repo setup, Docker Compose, DB schema, EF migrations, Auth (OAuth + JWT), basic user CRUD |
| **M2 — Core Features** | Categories API, time entries CRUD, user settings (goals + priorities), analytics endpoints |
| **M3 — Frontend Shell** | React app scaffold, routing, auth flow, planner page, time logging UI |
| **M4 — Progress & Priorities** | Charts (Recharts), weekly view, drag-and-drop priority page |
| **M5 — AI Assistant** | Provider abstraction, conversation API, chat UI, system prompt with user context |
| **M6 — Notifications** | Service Worker, VAPID push, notification settings, background job dispatcher |
| **M7 — Admin & Analytics** | Admin dashboard, user management, analytics event tracking, admin charts |
| **M8 — Production Hardening** | Cloud deployment (IaC), CI/CD pipeline, monitoring, security audit, E2E tests |

---

*End of specification. This document should be treated as a living reference — update version and date on each significant revision.*
