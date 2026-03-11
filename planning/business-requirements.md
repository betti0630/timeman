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

---

## 1. Executive Summary

**My Time Garden** is a personal time-allocation and life-management web application designed to help individuals plan, track, and reflect on how they invest their discretionary time across key life domains.

The application targets adults who juggle multiple competing responsibilities — household duties, self-improvement, professional growth, hobbies, and family obligations — and who struggle to give adequate attention to all of them within a limited daily window of 2–4 hours of free time.

The product will be delivered as a cloud-hosted SaaS web application, accessible from any modern browser on desktop or mobile. It will be built on a .NET 10 backend, a React 18 frontend, and a PostgreSQL database, deployed to a cloud provider (AWS, Azure, or GCP).

In its first version, the application is a personal tool with no monetisation and no team or family sharing features. It is designed to be extensible toward a commercial SaaS offering in future versions.

---

## 2. Business Context

### 2.1 Problem Statement

Modern adults with families and career ambitions face a persistent challenge: time is finite and demands are numerous. Common pain points include:

- Losing track of how much time is actually spent on different life areas
- Neglecting personal learning goals in favour of more urgent household or family tasks
- No simple, integrated tool that covers all five life domains together
- Existing tools are either too rigid (scheduled calendars) or too vague (generic habit trackers)
- No personalised guidance on how to spend remaining time in a day

### 2.2 Opportunity

There is a clear gap in the market for a warm, encouraging, AI-assisted time planner that:

- Tracks actual time spent (not just planned time)
- Surfaces progress visually in real time
- Adapts suggestions to the user's own priorities and goals
- Feels more like a personal coach than a corporate productivity tool

### 2.3 Proposed Solution

My Time Garden provides:

- A daily and weekly planner with five pre-defined life categories
- Real-time time logging with goal tracking and visual progress indicators
- A drag-and-drop priority system to reflect what matters most
- An AI assistant that gives personalised scheduling suggestions and motivation, aware of the user's current data
- Browser push notifications as lightweight reminders
- An admin dashboard for platform oversight

### 2.4 Strategic Fit

This product serves as both a personal utility and a proof-of-concept platform. Building a clean, scalable SaaS foundation now creates the option to expand into a commercial multi-user product, add premium features (custom categories, advanced analytics, coaching integrations), or pivot to a B2C subscription model in a future version.

---

## 3. Business Objectives

| ID | Objective | Measure of Success |
|---|---|---|
| BO-01 | Provide a single tool that covers all five life domains | All five categories are live and usable at launch |
| BO-02 | Make daily time logging frictionless | Users can log time in under 5 seconds per entry |
| BO-03 | Help users understand their time patterns | Progress charts available for daily and weekly views |
| BO-04 | Deliver personalised AI guidance within the app | AI assistant responds with context-aware suggestions |
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
- Frustration: Reaches the end of the day and has no idea where the time went

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

### 5.2 Time Planning

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-06 | The system must present a daily view showing all five life categories and the user's progress against their daily time goals | Must Have | BO-01, BO-03 |
| BR-07 | The system must present a weekly view showing time logged and goals across all five categories for the current and past weeks | Must Have | BO-03 |
| BR-08 | The system must allow users to navigate to any past day or week to view their historical data | Must Have | BO-03 |
| BR-09 | The system must display progress visually (charts, progress bars, percentage completion) in real time as time is logged | Must Have | BO-03 |

### 5.3 Time Logging

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-10 | The system must allow users to log time against any category on any date with minimal steps | Must Have | BO-02 |
| BR-11 | The system must provide quick-log shortcuts for common durations (15, 30, 60 minutes) | Must Have | BO-02 |
| BR-12 | The system must allow users to enter a custom duration in minutes | Must Have | BO-02 |
| BR-13 | The system must allow users to attach an optional text note to any time entry | Should Have | BO-01 |
| BR-14 | The system must allow users to edit or delete any of their time entries | Must Have | BO-01 |

### 5.4 Goals & Priorities

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-15 | The system must allow each user to set individual daily and weekly time goals for each of the five categories | Must Have | BO-01, BO-03 |
| BR-16 | The system must allow users to change their goals at any time | Must Have | BO-01 |
| BR-17 | The system must allow users to rank the five categories in order of personal priority | Must Have | BO-01 |
| BR-18 | The priority order must be reflected throughout the app, particularly in the order categories are displayed and in AI suggestions | Must Have | BO-01, BO-04 |

### 5.5 Progress & Insights

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-19 | The system must show users how much time they have logged versus their goal for each category, for both daily and weekly periods | Must Have | BO-03 |
| BR-20 | The system must track and display a consecutive-day streak for each category | Should Have | BO-03 |
| BR-21 | The system must allow users to compare the current week's total logged time to the previous week | Should Have | BO-03 |

### 5.6 AI Assistant

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-22 | The system must provide an AI-powered chat assistant accessible within the app | Must Have | BO-04 |
| BR-23 | The AI assistant must be aware of the user's goals, priorities, and time logged on the current day and current week | Must Have | BO-04 |
| BR-24 | The AI assistant must be capable of suggesting a schedule for the remainder of the day, reviewing the user's week, and providing motivational messages | Must Have | BO-04 |
| BR-25 | The AI provider must be configurable and swappable without a code change (e.g. between OpenAI, Anthropic Claude, and Azure OpenAI) | Must Have | BO-06 |
| BR-26 | The system must enforce a daily limit on AI messages per user to manage API costs | Must Have | BO-05 |
| BR-27 | AI API keys must never be exposed to the client — all AI calls must be proxied through the backend | Must Have | BO-07 |

### 5.7 Notifications

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-28 | The system must be capable of sending browser push notifications to users who have granted permission | Should Have | BO-01 |
| BR-29 | Users must be able to configure the time(s) and days on which reminder notifications are sent | Should Have | BO-01 |
| BR-30 | Users must be able to disable all notifications at any time | Must Have | BO-07 |
| BR-31 | The system must send a weekly summary notification to opted-in users | Could Have | BO-03 |

### 5.8 Administration

| ID | Requirement | Priority | Linked Objective |
|---|---|---|---|
| BR-32 | The system must provide an admin dashboard showing key platform metrics: total users, active users, entries today, AI usage | Must Have | BO-05 |
| BR-33 | Admins must be able to search, view, and manage all user accounts | Must Have | BO-05 |
| BR-34 | Admins must be able to deactivate a user account, preventing future logins | Must Have | BO-05, BO-07 |
| BR-35 | Admins must be able to view and edit the global category definitions (labels, emojis, colours) | Should Have | BO-05 |
| BR-36 | The system must capture and display analytics events for admin review (user registrations, time logged, AI usage, notification delivery) | Should Have | BO-05 |

### 5.9 Non-Functional Business Requirements

| ID | Requirement | Priority |
|---|---|---|
| BR-37 | The application must be accessible from any modern desktop or mobile web browser without requiring installation | Must Have |
| BR-38 | The application must be hosted on a public cloud provider to ensure availability and scalability | Must Have |
| BR-39 | User data must be isolated — a user must never be able to access another user's data | Must Have |
| BR-40 | The system must remain responsive and usable with many concurrent users | Must Have |
| BR-41 | The application must present all system messages, notifications, and AI responses in an encouraging, warm tone | Should Have |

---

## 6. Assumptions & Dependencies

### 6.1 Assumptions

| ID | Assumption |
|---|---|
| A-01 | Users have a modern web browser (Chrome, Firefox, Safari, Edge — current or previous major version) |
| A-02 | Users have a Google account for sign-in in V1 |
| A-03 | The five life categories (Housekeeping, English Learning, Professional Learning, Gardening, Child-Rearing) are fixed in V1 and do not need to be user-configurable |
| A-04 | The daily time budget of 2–4 hours is a design constraint for the AI assistant's suggestions, not a hard system limit |
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
| C-02 | No custom category creation in V1 — categories are fixed | Scope | Users cannot add, rename, or remove categories |
| C-03 | No monetisation or payment processing in V1 | Scope | No subscription or billing features |
| C-04 | No team or family sharing in V1 — single-user accounts only | Scope | Each account is entirely private |
| C-05 | No offline-first / PWA support in V1 | Technical | App requires an internet connection |
| C-06 | AI messages are limited per user per day | Cost | Users must plan their AI usage within the daily quota |
| C-07 | Admin role must be assigned manually — no self-service elevation | Security | Admin accounts require direct database or tooling intervention |

---

## 8. Risks

| ID | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R-01 | AI provider API costs exceed budget as user numbers grow | Medium | High | Per-user daily message limits; configurable provider to switch to cheaper option |
| R-02 | Google deprecates or changes the OAuth API used for sign-in | Low | High | Abstract auth provider; add alternative OAuth providers (GitHub, Microsoft) |
| R-03 | Browser push notification permissions are denied by most users, reducing notification value | High | Medium | All core functionality works without notifications; push is supplementary |
| R-04 | Users find five fixed categories insufficient for their needs | Medium | Medium | Design V2 to support custom categories; gather user feedback post-launch |
| R-05 | AI assistant responses are off-topic or inconsistent without sufficient user context | Medium | Medium | Careful system prompt engineering; context injected server-side per request |
| R-06 | Data breach due to insufficient access controls | Low | Very High | All queries scoped to authenticated user ID; regular security audits; secrets in vault |
| R-07 | Scope creep delays V1 delivery | High | Medium | Strict V1 scope definition; all new ideas moved to a V2 backlog |
| R-08 | Low user engagement — users sign up but do not return daily | Medium | High | Push notifications, streaks, and AI encouragement designed to drive re-engagement |

---

## 9. Success Criteria

The V1 release will be considered successful when the following criteria are met:

### 9.1 Functional Completeness

- All five categories are visible and fully functional in the daily and weekly planner
- Time entries can be created, edited, and deleted without errors
- Goals and priority order can be set and are correctly reflected across the app
- AI assistant responds with contextually appropriate messages
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
- First-time onboarding flow (goals setup, notification opt-in)
- Daily planner with per-category progress, quick-log, and custom-log
- Weekly planner with goal vs. actual chart and summary table
- Time entry management (create, edit, delete, note)
- Category goals (daily and weekly) per user
- Priority management (drag-to-reorder)
- Progress page with streaks and charts
- AI assistant chat with full user context, conversation history, and daily message limit
- Browser push notifications (reminders, goal achieved, weekly summary)
- User settings (profile, goals, notifications, account deletion)
- Admin dashboard (KPIs, user table, category management)
- Admin analytics (charts, event log)
- Cloud deployment on AWS / Azure / GCP
- CI/CD pipeline (GitHub Actions)

### 10.2 Out of Scope — V1 (Candidates for V2)

| Item | Reason deferred |
|---|---|
| Native iOS / Android app | Significant additional effort; responsive web covers mobile use |
| Custom user-defined categories | Increases complexity; V1 validates the core concept with fixed categories |
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
| Category | One of the five life domains tracked by the application: Housekeeping, English Learning, Professional Learning, Gardening, Child-Rearing |
| Daily Goal | The number of minutes a user wants to log for a given category in a single day |
| Weekly Goal | The number of minutes a user wants to log for a given category across a full week (Mon–Sun) |
| Time Entry | A single logged record of time spent on a category on a specific date |
| Streak | The number of consecutive calendar days on which at least one time entry was made for a given category |
| Priority Rank | The user-defined importance order of the five categories; rank 1 is highest priority |
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
