# My Time Garden — Business Requirements Document

**Document type:** Business Requirements Document (BRD)
**Version:** 1.0
**Date:** March 2026
**Author:** Product Owner
**Status:** Draft — Awaiting Approval

---

## Document Control

| Version | Date | Author | Changes |
|---|---|---|---|
| 0.1 | March 2026 | Product Owner | Initial draft |
| 1.0 | March 2026 | Product Owner | First complete version for review |

## Approvals

| Role | Name | Signature | Date |
|---|---|---|---|
| Product Owner | | | |
| Lead Developer | | | |
| Stakeholder | | | |

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Business Context](#2-business-context)
3. [Business Objectives](#3-business-objectives)
4. [Stakeholders](#4-stakeholders)
5. [Business Requirements](#5-business-requirements)
6. [Assumptions & Dependencies](#6-assumptions--dependencies)
7. [Constraints](#7-constraints)
8. [Risks](#8-risks)
9. [Success Criteria](#9-success-criteria)
10. [Scope](#10-scope)
11. [Glossary](#11-glossary)
12. [Decision Log](#12-decision-log)

---

## 1. Executive Summary

**My Time Garden** is a personal time-planning and life-management web application that helps individuals plan their discretionary time across key life domains, track execution of those plans, and reflect on how well they follow through.

The application targets adults who juggle multiple competing responsibilities — household duties, self-improvement, professional growth, hobbies, and family obligations — and who struggle to give adequate attention to all of them within their personal daily free-time window.

The product will be delivered as a cloud-hosted SaaS web application, accessible from any modern browser on desktop or mobile. It will be built on a .NET 10 backend, a React 18 frontend, and a PostgreSQL database, deployed to a cloud provider (AWS, Azure, or GCP).

In its first version, the application is a personal tool with no monetisation and no team or family sharing features. It is designed to be extensible toward a commercial SaaS offering in future versions.

---

## 2. Business Context

### 2.1 Problem Statement

Modern adults with families and career ambitions face a persistent challenge: time is finite and demands are numerous. Common pain points include:

- No clear daily or weekly plan for how to spend discretionary time across life domains
- Losing track of how much time is actually spent versus what was intended
- Neglecting personal learning goals in favour of more urgent household or family tasks
- No simple, integrated tool that covers all key life domains together
- Existing tools are either too rigid (scheduled calendars) or too vague (generic habit trackers)
- No personalised guidance on how to plan and spend remaining time in a day

### 2.2 Opportunity

There is a clear gap in the market for a warm, encouraging, AI-assisted time planner that:

- Makes daily planning fast and frictionless — the plan comes first
- Tracks execution of the plan and surfaces the gap between intended and actual time
- Adapts planning suggestions to the user's own priorities, goals, and available time
- Feels more like a personal coach than a corporate productivity tool

### 2.3 Proposed Solution

My Time Garden provides:

- A daily, weekly, and monthly planner built around planned time entries — plan first, log completion second
- A drag-and-drop priority system to reflect what matters most
- Visual plan vs. actual tracking with completion rates and progress indicators
- An AI assistant that helps build and refine the user's plan, aware of their goals, priorities, and available time
- Browser push notifications as lightweight reminders
- An admin dashboard for platform oversight

### 2.4 Strategic Fit

This product serves as both a personal utility and a proof-of-concept platform. Building a clean, scalable SaaS foundation now creates the option to expand into a commercial multi-user product, add premium features (custom categories, advanced analytics, coaching integrations), or pivot to a B2C subscription model in a future version.

---

## 3. Business Objectives

| ID | Objective | Measure of Success |
|---|---|---|
| BO-01 | Provide a single planning tool covering all key life domains | All seeded categories are live and fully plannable at launch |
| BO-04 | Deliver an AI planning assistant as the primary source of value | AI assistant helps users plan at day, week, month, and longer horizons with one-click apply (for users with a non-zero AI quota tier) |
| BO-02 | Make daily planning frictionless | Users can create a planned time entry in under 5 seconds |
| BO-03 | Help users understand their time patterns and plan vs. actual gaps | Completion rate and progress charts available for daily, weekly, and monthly views |
| BO-05 | Build a scalable SaaS foundation for future growth | Architecture supports many concurrent users |
| BO-06 | Establish a stable, maintainable codebase | Full test coverage on core services; CI/CD pipeline live |
| BO-07 | Ensure user data security and privacy | OAuth-only login; all data scoped to the authenticated user |

---

## 4. Stakeholders

### 4.1 Internal Stakeholders

| Stakeholder | Role | Interest |
|---|---|---|
| Product Owner | Defines requirements, prioritises backlog | Ensuring the product solves the stated problem |
| Lead Developer | Architecture and implementation | Technical feasibility, code quality, maintainability |
| UI/UX Designer | Interface design | Usability, visual consistency, accessibility |

### 4.2 External Stakeholders

| Stakeholder | Role | Interest |
|---|---|---|
| End Users (Regular) | Primary consumers of the product | Ease of use, reliable tracking, motivating experience |
| End Users (Admin) | Platform administrators | System health, user management, analytics visibility |
| OAuth Provider (Google) | Provides identity verification | Correct implementation of OAuth 2.0 standards |
| AI Provider | Provides language model API | API usage within rate limits and cost budgets |
| Cloud Provider | Hosts the infrastructure | Correct resource provisioning and billing |

### 4.3 User Personas

**Persona 1 — The Busy Parent**
- Age: 30–45
- Situation: Works part-time or full-time, has one or more young children, wants to carve out time for self-improvement
- Goal: Spend at least 30 minutes per day on English learning and not let housekeeping take over the whole day
- Frustration: Has no clear plan for the day and reacts to whatever demands arise

**Persona 2 — The Self-Improver**
- Age: 25–40
- Situation: Has clear learning goals (language, professional skills) but struggles with consistency
- Goal: Build daily habits and see streaks grow over time
- Frustration: Habit apps feel cold and corporate; wants something that feels personal and encouraging

**Persona 3 — The Platform Admin**
- Age: Any
- Situation: Responsible for the health of the deployment
- Goal: Monitor active users, resolve account issues, track system usage
- Frustration: Lack of visibility into what is happening on the platform

---

## 5. Business Requirements

Business requirements describe **what** the system must do to satisfy the business objectives. They are technology-neutral — implementation details belong in the technical or functional specification.

### 5.1 User Access & Identity

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-01 | The system must allow users to register and sign in using a third-party OAuth provider (Google as minimum) without creating a separate username and password | Must Have | BO-07 |
| BR-02 | The system must maintain a secure, persistent session so users do not need to re-authenticate on every visit | Must Have | BO-02 |
| BR-03 | The system must allow users to sign out and must invalidate their session immediately | Must Have | BO-07 |
| BR-04 | The system must allow users to permanently delete their account and all associated data | Must Have | BO-07 |
| BR-05 | The system must support an admin role with elevated capabilities, distinct from regular users | Must Have | BO-05 |

### 5.2 Onboarding

New users must complete a first-run setup flow immediately after their first sign-in. All steps are required before accessing the main application. The same settings remain editable in Settings after onboarding.

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-06 | Upon first sign-in the system must present a mandatory onboarding flow that collects the user's initial configuration before granting access to the main application | Must Have | BO-01, BO-02 |
| BR-07 | Onboarding must require the user to select which categories from the admin catalogue they wish to track; if the catalogue contains no visible categories, the system must show a fallback message and prevent onboarding from proceeding until an admin makes at least one category available | Must Have | BO-01 |
| BR-08 | Onboarding must require the user to set a daily free-time budget in minutes | Must Have | BO-04 |
| BR-09 | Onboarding must require the user to add at least one Backlog Item to each selected category; the system must guide the user through creating their initial backlog before accessing the main application | Must Have | BO-01, BO-02 |
| BR-10 | Onboarding must prompt the user to configure browser push notification preferences (with the ability to opt out); if the user skips this step, notifications must default to off | Must Have | BO-01 |
| BR-11 | New users must be assigned the zero AI message quota tier by default, giving no access to the AI assistant until an admin upgrades their tier | Must Have | BO-05 |

### 5.3 Time Planning

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-12 | The system must present a daily view showing the user's active categories and their progress against daily time goals | Must Have | BO-01, BO-03 |
| BR-13 | The system must present a weekly view showing, for each day: planned entries for future days and logged entries for past days, alongside goals and completion rates across the user's active categories | Must Have | BO-03 |
| BR-14 | The system must allow users to navigate to any past day or week to view their historical data; all history is retained indefinitely | Must Have | BO-03 |
| BR-15 | The system must display progress visually (charts, progress bars, percentage completion) in real time as time is logged | Must Have | BO-03 |
| BR-16 | The daily planner must display a visual remaining budget indicator showing how much of the user's daily free-time budget has been consumed across all active categories | Should Have | BO-03, BO-04 |
| BR-17 | The system must detect and store the user's local timezone on every sign-in and apply it consistently for all date calculations, day boundaries, and AI scheduling suggestions | Must Have | BO-02, BO-04 |
| BR-84 | The system must present a monthly view showing all scheduled Tasks and generic time blocks across the calendar month, allowing users to navigate forward and backward by month | Must Have | BO-03 |

### 5.4 Time Logging

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-18 | The system must allow users to log ad-hoc time against any category on today or any past date; logging against future dates is not permitted; ad-hoc logging is optional — the primary workflow is creating and completing planned entries | Should Have | BO-02 |
| BR-19 | The system must provide quick-log shortcuts for common durations (15, 30, 60 minutes) | Must Have | BO-02 |
| BR-20 | The system must allow users to enter a custom duration in minutes | Must Have | BO-02 |
| BR-21 | The system must allow users to attach an optional text note to any time entry | Should Have | BO-01 |
| BR-22 | The system must allow users to edit or delete any of their time entries | Must Have | BO-01 |

### 5.4b Task Management

Each category has a **Backlog** — a catalogue of user-defined activities (Backlog Items) that the user intends to do. Scheduling a Backlog Item for a specific date and duration creates a **Task**. Tasks are the primary planning unit. Completing a Task automatically logs the time. Backlog Items are persistent catalogue entries; they are not consumed or removed when Tasks are created from them.

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-67 | The system must allow users to create, edit, and delete Backlog Items within any of their active categories; Backlog Items are entirely user-managed with no admin involvement | Must Have | BO-01 |
| BR-68 | Each Backlog Item must have a name; it may optionally have an estimated duration in minutes, a due date, and a recurrence schedule | Must Have | BO-01 |
| BR-69 | The system must allow users to order Backlog Items within a category; the order is user-defined and reflected in the backlog panel and AI suggestions | Must Have | BO-01 |
| BR-70 | The system must allow users to create a Task by scheduling a Backlog Item for a specific date and assigning it a duration; the Backlog Item remains in the backlog and the same Backlog Item may have multiple Tasks scheduled across different dates simultaneously; a Task may also be created as a generic time block without a linked Backlog Item | Must Have | BO-02 |
| BR-71 | Marking a Task done must prompt the user to confirm or adjust the actual time spent (pre-filled with the planned duration); on confirmation, a logged time entry is automatically created for the Task's category using the actual duration; the original planned duration is preserved separately for plan vs. actual comparison | Must Have | BO-01, BO-02 |
| BR-72 | Backlog Items configured with a recurrence schedule must automatically generate the next Task instance in the backlog when the current Task is marked done | Should Have | BO-01 |
| BR-73 | The AI assistant must be aware of each category's full backlog — including Backlog Item names, estimated durations, due dates, recurrence, and order — when suggesting plans at any horizon | Must Have | BO-04 |
| BR-74 | Each active category must display a backlog panel showing all Backlog Items ordered by user-defined sequence, accessible as a sidebar during planning | Must Have | BO-01 |
| BR-75 | If a Task's scheduled date passes without being marked done, it must be displayed as overdue; the user may complete, reschedule, or delete the overdue Task | Must Have | BO-01 |
| BR-76 | The backlog panel must be visible within the planning view so the user can pick Backlog Items when building or adjusting their plan | Must Have | BO-02 |
| BR-77 | A Backlog Item is archived (removed from the active backlog) when the user explicitly marks it as permanently done from the backlog view | Must Have | BO-01 |
| BR-78 | Planned entries (Tasks and generic time blocks) must be visually distinct from logged time entries in the daily planner, colour-coded by type | Must Have | BO-02 |
| BR-79 | Tasks must not count toward completion rates or the remaining budget indicator until marked done; only logged (completed) entries contribute to those measures | Must Have | BO-03, BO-04 |
| BR-80 | The user must be able to edit a Task's duration or linked Backlog Item before marking it done; the original planned duration is preserved separately | Must Have | BO-02 |
| BR-81 | At end of day, the system must prompt the user to review any incomplete Tasks remaining for today and surface tomorrow's scheduled Tasks so the user can prepare or adjust; for each incomplete Task the user may complete, reschedule, or delete | Should Have | BO-03 |
| BR-82 | Tasks may be created for any date from today up to 1 year ahead; this applies to both AI-applied and manually created Tasks | Must Have | BO-02 |
| BR-83 | The system must allow users to manually create Tasks for any date directly from the planner UI, independently of the AI assistant; generic time blocks without a linked Backlog Item are also supported | Must Have | BO-02 |

### 5.5 Goals & Priorities

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-23 | The system must derive each user's time commitment per category from their scheduled Tasks; the sum of planned Task durations for a given day or week is the effective goal for that period — there are no separately stored time goals | Must Have | BO-01, BO-03 |
| BR-25 | The system must allow users to rank their active categories in order of personal priority | Must Have | BO-01 |
| BR-26 | The priority order must be reflected throughout the app, particularly in the order categories are displayed and in AI suggestions | Must Have | BO-01, BO-04 |

### 5.6 User Settings

Personal settings that apply across the entire app and are editable at any time after onboarding.

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-27 | The system must allow users to select which categories from the admin-managed catalogue are active in their personal planner view; when a user deactivates a category, its existing time entries, Backlog Items, and Tasks are hidden from the planner but fully preserved and restored if the category is reactivated | Must Have | BO-01 |
| BR-28 | The system must allow users to set a personal daily free-time budget (a single value in minutes) | Must Have | BO-04 |

### 5.7 Progress & Insights

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-29 | The system must show users how much time they have logged versus their goal for each category, for both daily and weekly periods | Must Have | BO-03 |
| BR-30 | The system must track and display a consecutive-day streak for each category, based on days where at least one planned entry for that category was completed | Should Have | BO-03 |
| BR-66 | The system must calculate and display a daily completion rate per category: the percentage of planned minutes that were actually logged on a given day | Should Have | BO-03 |
| BR-31 | The system must allow users to compare the current week's total logged time to the previous week | Should Have | BO-03 |

### 5.8 AI Assistant

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-32 | The system must provide an AI-powered chat assistant accessible within the app; users on the zero quota tier must see a clear in-app message explaining that AI access is not yet enabled and that they should contact an admin | Must Have | BO-04 |
| BR-33 | The AI assistant must be aware of the user's priorities, daily free-time budget, full backlog (Backlog Item names, durations, due dates, recurrence, order), scheduled Tasks, completion rates, and time logged across all relevant horizons; a per-day budget stated by the user in the chat must take precedence over the stored setting for that session | Must Have | BO-04 |
| BR-34 | The AI assistant must be capable of suggesting a plan at any time horizon — remainder of the day, a full day, a week, a month, or longer; plans longer than one week are broken down into weekly segments; the AI must also review plan vs. actual completion and provide motivational messages | Must Have | BO-04 |
| BR-35 | The AI provider must be configurable and swappable without a code change (e.g. between OpenAI, Anthropic Claude, and Azure OpenAI) | Must Have | BO-06 |
| BR-36 | The system must enforce a daily AI message limit per user based on their assigned quota tier; three tiers exist: zero (no AI access), low, and high — the low and high limits are configurable by admins; tier changes take effect immediately — a demotion to zero locks the user out at once, and an upgrade grants the remaining allowance of the new tier (new tier limit minus messages already sent today) | Must Have | BO-05 |
| BR-37 | The system must retain AI conversation history indefinitely for user reference; only the most recent messages (up to a configurable limit) are injected into each API request to manage context window size and cost | Must Have | BO-05, BO-07 |
| BR-38 | AI API keys must never be exposed to the client — all AI calls must be proxied through the backend | Must Have | BO-07 |
| BR-56 | When the AI presents a plan at any horizon, it must offer a one-click apply action that creates the corresponding Tasks (from Backlog Items) or generic time blocks across the relevant dates; multi-week plans are applied week by week | Must Have | BO-04 |
| BR-58 | One-click apply is all-or-nothing — the user cannot partially apply a plan; if the suggestion is not suitable, the user must ask the AI to revise it before applying | Must Have | BO-04 |
| BR-59 | The user must be able to undo an applied plan in a single action, removing all Tasks created by that application | Must Have | BO-04 |

### 5.9 Notifications

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-39 | The system must be capable of sending browser push notifications to users who have granted permission | Should Have | BO-01 |
| BR-40 | Users must be able to configure the schedule (time and days) for each notification type independently | Should Have | BO-01 |
| BR-41 | Users must be able to disable all notifications at any time | Must Have | BO-07 |
| BR-42 | The system must send a weekly summary notification to opted-in users; the summary is delivered on Monday early morning in the user's local timezone | Could Have | BO-03 |

### 5.10 Administration

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-43 | The system must provide an admin dashboard showing key platform metrics: total users, active users, entries today, AI usage, and current quota tier limits | Must Have | BO-05 |
| BR-44 | Admins must be able to search and view all user accounts | Must Have | BO-05 |
| BR-45 | Admins must be able to deactivate and reactivate user accounts; a deactivated account cannot log in but all data is preserved and fully restored upon reactivation | Must Have | BO-05, BO-07 |
| BR-46 | Admins must be able to assign a user to any of the three AI message quota tiers: zero, low, or high | Must Have | BO-05 |
| BR-47 | Admins must be able to configure the daily message limits for the low and high tiers via the admin dashboard; changes apply immediately to all users on that tier | Must Have | BO-05 |
| BR-48 | Admins must be able to view, add, hide, and edit global category definitions (labels, emojis, colours) | Should Have | BO-05 |
| BR-49 | A hidden category must no longer appear in any user's selectable catalogue, but all existing time entries, Backlog Items, and Tasks associated with it must be preserved and remain linked to that category | Must Have | BO-01, BO-07 |
| BR-50 | The system must capture and display analytics events for admin review (user registrations, time logged, AI usage, notification delivery) | Should Have | BO-05 |

### 5.11 Non-Functional Business Requirements

| ID | Requirement | Priority |
|---|---|---|
| BR-51 | The application must be accessible from any modern desktop or mobile web browser without requiring installation | Must Have |
| BR-52 | The application must be hosted on a public cloud provider to ensure availability and scalability | Must Have |
| BR-53 | User data must be isolated — a user must never be able to access another user's data | Must Have |
| BR-54 | The system must remain responsive and usable with many concurrent users | Must Have |
| BR-55 | The application must present all system messages, notifications, and AI responses in an encouraging, warm tone | Should Have |

---

## 6. Assumptions & Dependencies

### 6.1 Assumptions

| ID | Assumption |
|---|---|
| A-01 | Users have a modern web browser (Chrome, Firefox, Safari, Edge — current or previous major version) |
| A-02 | Users have a Google account for sign-in in V1 |
| A-03 | Five life categories are seeded at launch; admins may add or hide categories; users select which active categories appear in their personal view; time entries always remain linked to their original category (see Section 12 for full detail) |
| A-04 | Each user configures a personal daily free-time budget (single value in minutes) used as the primary AI scheduling input; per-day chat overrides take precedence; it is not a hard system limit (see Section 13 for full detail) |
| A-05 | The AI provider will be available with sufficient rate limits to support expected usage |
| A-06 | Browser push notification support is available in the user's browser (graceful degradation where not supported) |
| A-07 | The application will be used primarily in a single language (English) in V1 |
| A-08 | No payment processing or subscription management is required in V1 |

### 6.2 Dependencies

| ID | Dependency | Owner | Risk if unavailable |
|---|---|---|---|
| D-01 | Google OAuth 2.0 API | Google | Users cannot sign in |
| D-02 | Chosen AI provider API (Anthropic / OpenAI / Azure) | AI Provider | AI assistant unavailable |
| D-03 | Cloud hosting provider (AWS / Azure / GCP) | Cloud Provider | Full application unavailable |
| D-04 | Browser Push Notification API (Web Push / VAPID) | Browser vendors | Notifications unavailable |
| D-05 | PostgreSQL managed database service | Cloud Provider | All data read/write unavailable |

---

## 7. Constraints

| ID | Constraint | Type | Impact |
|---|---|---|---|
| C-01 | No native mobile app in V1 — web only | Scope | Users on mobile use the responsive web version |
| C-02 | Users cannot create their own category definitions — category management is an admin function | Scope | Users select from the admin-managed catalogue; they cannot define new categories from scratch |
| C-03 | No monetisation or payment processing in V1 | Scope | No subscription or billing features |
| C-04 | No team or family sharing in V1 — single-user accounts only | Scope | Each account is entirely private |
| C-05 | No offline-first / PWA support in V1 | Technical | App requires an internet connection |
| C-06 | AI message access is gated by a three-tier quota system (zero, low, high); new users default to zero (no access) until an admin assigns a non-zero tier | Cost | Users have no AI access until explicitly enabled by an admin; message counts reset daily |
| C-07 | Admin role must be assigned manually — no self-service elevation | Security | The first admin account is created via a seed script at deployment; subsequent admins require direct database or tooling intervention |
| C-08 | The week is fixed as Monday–Sunday — the week start day is not user-configurable in V1 | Scope | All weekly goals, charts, and summaries are calculated on a Mon–Sun basis |
| C-09 | Planned entries may be created up to 1 year ahead — planning beyond 12 months is not supported in V1 | Scope | Keeps the planner focused; longer horizons are decomposed into weekly segments by the AI |

---

## 8. Risks

| ID | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R-01 | AI provider API costs exceed budget as user numbers grow | Low | High | Zero-tier default means no AI costs until admin explicitly enables access; per-user daily limits and configurable provider provide additional protection |
| R-02 | Google deprecates or changes the OAuth API used for sign-in | Low | High | Abstract auth provider; add alternative OAuth providers (GitHub, Microsoft) |
| R-03 | Browser push notification permissions are denied by most users, reducing notification value | High | Medium | All core functionality works without notifications; push is supplementary |
| R-04 | Users find the seeded categories insufficient for their needs | Low | Medium | Admins can add categories in V1; users can hide irrelevant ones; full user-created categories deferred to V2; gather feedback post-launch |
| R-05 | AI assistant responses are off-topic or inconsistent without sufficient user context | Medium | Medium | Careful system prompt engineering; context injected server-side per request |
| R-06 | Data breach due to insufficient access controls | Low | Very High | All queries scoped to authenticated user ID; regular security audits; secrets in vault |
| R-07 | Scope creep delays V1 delivery | Very High | High | Scope has grown significantly (task pools, Kanban model, monthly planner, AI multi-horizon planning); strict feature freeze needed; all further additions must go to V2 backlog |
| R-08 | Low user engagement — users sign up but do not return daily | Medium | High | Push notifications and streaks drive re-engagement for all users; AI encouragement available once admin enables a non-zero quota tier |
| R-09 | Admin never upgrades users from zero AI tier, making BO-04 unreachable for all users | Medium | High | Admin dashboard highlights users stuck on zero tier; admin onboarding guidance should include activating AI access as a first step |

---

## 9. Success Criteria

The V1 release will be considered successful when the following criteria are met:

### 9.1 Functional Completeness

- All seeded categories are visible in the admin catalogue and fully functional in the daily, weekly, and monthly planner
- Backlog Items can be created, ordered, and managed within each category
- Tasks can be created, edited, marked done, and overdue Tasks are flagged correctly
- Completing a Task creates a logged time entry with the correct duration
- Completion rates and streaks are calculated correctly from completed Tasks
- AI assistant suggests and applies plans at day, week, and month horizons (requires at least one test user with a non-zero AI quota tier)
- Push notifications are sent and received correctly for users who opt in
- Admin dashboard shows accurate live metrics and supports user management

### 9.2 Quality

- All API endpoints covered by integration tests
- Core service logic covered by unit tests with at least 80% line coverage
- No Severity 1 (data loss or security) bugs open at release
- Application loads in under 2 seconds on a 4G mobile connection

### 9.3 Operational

- CI/CD pipeline is live: pushes to main deploy to staging automatically; production requires a manual approval gate
- Application has been running on the staging environment for at least 7 days without unplanned downtime
- Monitoring and alerting are configured for API errors and latency

### 9.4 User Acceptance

- The product owner has completed end-to-end testing of all user journeys defined in the Functional Specification
- At least one external user (not involved in development) has tested the app and provided feedback

---

## 10. Scope

### 10.1 In Scope — V1

- OAuth 2.0 sign-in (Google)
- First-time onboarding flow (category selection, daily free-time budget, goals setup, notification opt-in)
- Daily planner (Task timeline, backlog panel, remaining budget indicator)
- Weekly planner (plan vs. actual, completion rates, overdue Tasks)
- Monthly planner (calendar view of all scheduled Tasks)
- Task management (create, mark done, overdue handling, recurrence)
- Time entry management (ad-hoc logging; edit, delete, note)
- Priority management (drag-to-reorder categories)
- Progress page with streaks and charts
- AI assistant chat with full user context, conversation history, three-tier quota system, and per-type notification scheduling
- AI one-click schedule apply (creates planned time entries; all-or-nothing; undoable)
- Task pool per category (Kanban-style: Pool → Planned → Done states; pool visible during planning)
- Task management (create, order, duration, due date, recurrence; overdue entry handling)
- Planned task entry management (manual and AI apply; visually distinct; mark done auto-logs time)
- Weekly and monthly planning (future-dated planned entries; higher-level goal templates)
- Browser push notifications (reminders, goal achieved, weekly summary)
- User settings (profile, goals, daily free-time budget, category selection, notifications, account deletion)
- Admin dashboard (KPIs, user table, category management, AI quota tier management, zero-tier user alert)
- User deactivation and reactivation
- Timezone detection on every sign-in
- AI conversation history (stored indefinitely; context-window injection capped at configurable limit)
- Admin analytics (charts, event log)
- Cloud deployment on AWS / Azure / GCP
- CI/CD pipeline (GitHub Actions)

### 10.2 Out of Scope — V1 (Candidates for V2)

| Item | Reason deferred |
|---|---|
| Native iOS / Android app | Significant additional effort; responsive web covers mobile use |
| User-created category definitions | Users can select and hide from the admin catalogue but cannot create entirely new categories; deferred to V2 |
| Team / family shared accounts | Multi-tenancy at account level requires significant additional design |
| Calendar sync (Google Calendar, Outlook) | OAuth scope complexity; not core to V1 value proposition |
| Paid plans / subscription billing | No monetisation required in V1 |
| Offline-first / PWA | Not critical for V1; requires Service Worker caching strategy |
| Email notifications | Push notifications are sufficient for V1 |
| Data export (CSV / PDF) | Useful but not core; deferred to V2 |
| Advanced AI features (voice, image input) | Out of scope for a text-based planning assistant |

---

## 11. Glossary

| Term | Definition |
|---|---|
| BRD | Business Requirements Document — this document |
| Category | A life domain tracked by the application. Five are seeded at launch (Housekeeping, English Learning, Professional Learning, Gardening, Child-Rearing); admins may add or hide categories; users select which active categories appear in their personal view |
| Backlog Item | A named activity created by the user in a category's backlog; a persistent catalogue entry; optionally has an estimated duration, a due date, and a recurrence schedule; user-ordered within the backlog; not consumed when Tasks are created from it |
| Backlog | The per-category catalogue of all Backlog Items; visible as a panel during planning; the source from which Tasks are created |
| Task | A Backlog Item scheduled for a specific date and assigned a duration; the primary planning unit; has a state of Planned or Done; the same Backlog Item may have multiple Tasks across different dates simultaneously |
| Generic Time Block | A planned time block without a linked Backlog Item; used for unspecified category time; also has Planned or Done states |
| Time Entry | A logged record of time actually spent on a category; created automatically when a Task or Generic Time Block is marked done (actual duration confirmed by user) or by ad-hoc logging; distinct from a Task |
| Completion Rate | The percentage of planned Task duration for a category on a given day that was actually logged; e.g. planned 2 h of Tasks, logged 1 h = 50% |
| Streak | The number of consecutive calendar days on which at least one Task for a given category was marked done |
| Daily Commitment | The sum of durations of all Tasks scheduled for a category on a given day; derived automatically from the plan — not manually entered |
| Weekly Commitment | The sum of durations of all Tasks scheduled for a category across a full week (Mon–Sun); derived from the plan |
| Priority Rank | The user-defined importance order of their active categories; rank 1 is highest priority |
| Daily Free-Time Budget | The user's personal total discretionary time available per day, stored as a single value in minutes; used as the primary input for AI scheduling suggestions |
| AI Message Quota Tier | One of three platform-defined daily AI message allowances assigned to a user by an admin: zero (no AI access), low, and high; new users default to zero |
| AI Assistant | The in-app chat interface powered by a configurable large language model (LLM) |
| Push Notification | A browser-level notification delivered by the Web Push protocol, requiring prior user permission |
| OAuth | Open Authorisation — the delegated authentication standard used for social login (e.g. Sign in with Google) |
| JWT | JSON Web Token — the token format used for authenticating API requests |
| SaaS | Software as a Service — a cloud-hosted application delivered via a browser |
| Admin | A user with the elevated role allowing access to the Admin Dashboard |
| V1 | Version 1 — the first production release of the application |
| V2 | Version 2 — the next planned release, containing deferred features |

---

*End of Business Requirements Document.*
*This document should be reviewed and re-baselined whenever a significant scope change is agreed. All changes must be recorded in the Document Control table.*

---

## 12. Decision Log

All key decisions made during document review. Replaces the detailed review notes (Sections 12–23).

### 12.1 Categories

| Decision | Detail |
|---|---|
| Five seeded categories at launch | Admins may add or hide categories; users select which active ones appear in their view |
| Time entries always linked to original category | Hiding a category preserves all its data; entries remain accessible in history |
| User-created categories | Deferred to V2; users select from admin-managed catalogue only |

### 12.2 Daily Free-Time Budget

| Decision | Detail |
|---|---|
| Per-user configurable | Single value in minutes; set in onboarding and editable in Settings |
| Per-day override | User can state their available time in the AI chat; overrides stored value for that session |
| Visual indicator | Remaining budget shown on the daily planner; not a hard limit |

### 12.3 User Account & Access

| Decision | Detail |
|---|---|
| First admin | Created via seed script at deployment; subsequent admins via direct DB intervention |
| Deactivation | Reversible; all data preserved and restored on reactivation |
| Timezone | Detected and stored on every sign-in; applied to all date calculations and AI suggestions |
| Week start | Fixed Monday–Sunday (C-08); not user-configurable in V1 |

### 12.4 AI Quota System

| Decision | Detail |
|---|---|
| Three tiers | Zero (no access, default for new users), Low, High |
| Admin-configurable | Low and High limits set by admin via dashboard |
| Tier change timing | Immediate; demotion locks out at once; upgrade grants remaining allowance of new tier (new limit − messages sent today) |
| Zero-tier UX | In-app message explaining AI is not enabled; user told to contact admin |
| AI conversation history | Stored indefinitely; only most recent messages injected per API call |

### 12.5 Onboarding

| Decision | Detail |
|---|---|
| Mandatory steps | Category selection, daily free-time budget, at least one Backlog Item per selected category, notification preferences |
| All steps Must Have | Including notification prompt (defaults to off if skipped) |
| Empty catalogue | Fallback message shown; onboarding blocked until admin makes at least one category available |
| New users | Assigned zero AI tier by default |

### 12.6 Planning Model (Planning-First)

| Decision | Detail |
|---|---|
| Primary workflow | Create Backlog Items → schedule as Tasks → mark done (auto-logs time) |
| Ad-hoc logging | Optional (Should Have); for unplanned time not associated with a Task |
| Future date restriction | Tasks may be created up to 1 year ahead; ad-hoc logs restricted to today and past |
| Mark done | Prompts user to confirm or adjust actual time (pre-filled with planned duration); preserves original planned duration for plan vs. actual comparison |
| Weekly summary notification | Monday early morning in user's local timezone |
| Notification schedule | Configurable per notification type independently |

### 12.7 Task & Backlog Model

| Decision | Detail |
|---|---|
| Two entities | Backlog Item (persistent catalogue entry per category) → Task (scheduled instance with date + duration) |
| Backlog Items | Name, optional estimated duration, optional due date, optional recurrence; user-ordered; never consumed when Tasks are created |
| Tasks | Planned or Done state; same Backlog Item may have multiple Tasks across different dates simultaneously |
| Completion | Marking a Task done auto-creates a logged time entry; user confirms/adjusts actual duration |
| Recurrence | Auto-generates next Task in backlog when current is marked done |
| Overdue | Unfinished Task past its date stays as overdue; user may complete, reschedule, or delete |
| Data on category hide | Backlog Items and Tasks preserved alongside time entries when category is hidden (admin or user) |
| Onboarding | At least one Backlog Item per selected category required before proceeding |

### 12.8 Goals & Progress

| Decision | Detail |
|---|---|
| Time-based goals removed | Daily/Weekly Commitment is derived automatically from sum of scheduled Task durations |
| Completion rate | Percentage of planned Task duration actually logged per category per day |
| Streak | Consecutive days with at least one completed Task per category |
| Plan vs. actual | Always available; planned duration preserved separately when actual differs |

### 12.9 AI Planning

| Decision | Detail |
|---|---|
| AI is second business objective | After planning tool itself (BO-01 primary, BO-04 second) |
| Planning horizons | Day, week, month, longer; periods beyond one week decomposed into weekly segments |
| One-click apply | Available at all horizons; creates Tasks from Backlog Items; all-or-nothing; undoable |
| AI context | Full backlog (names, durations, due dates, recurrence, order), scheduled Tasks, completion rates, priorities, daily free-time budget |
| Zero-tier | App fully usable without AI; AI is the primary value-add, not a requirement |

### 12.10 Planner Views

| Decision | Detail |
|---|---|
| Daily planner | Single colour-coded timeline of Tasks and logged entries; backlog panel sidebar; remaining budget indicator |
| Weekly planner | Future days show planned Tasks; past days show logged entries; completion rates per category |
| Monthly planner | Calendar view of all scheduled Tasks and generic time blocks (BR-84) |
| End-of-day review | Prompts for today's incomplete Tasks + surfaces tomorrow's plan |

### 12.11 Deferred to V2

| Item |
|---|
| User-created category definitions |
| Weekly and monthly plan templates (reusable named templates) |
| Native iOS / Android app |
| Team / family shared accounts |
| Calendar sync |
| Paid plans / subscription billing |
| Offline-first / PWA |
| Email notifications |
| Data export (CSV / PDF) |
| Advanced AI features (voice, image input) |

---

*End of Business Requirements Document.*
*This document should be reviewed and re-baselined whenever a significant scope change is agreed.*
