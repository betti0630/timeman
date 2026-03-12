# My Time Garden — Functional Specification

**Document type:** Functional Specification
**Version:** 2.1
**Date:** March 2026
**Audience:** Developers, product owner, stakeholders
**Status:** Draft
**Based on:** Business Requirements Document v1.0

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [User Roles](#2-user-roles)
3. [User Journeys](#3-user-journeys)
4. [Feature Specifications](#4-feature-specifications)
   - 4.1 [Authentication & Onboarding](#41-authentication--onboarding)
   - 4.2 [Daily Planner](#42-daily-planner)
   - 4.3 [Weekly Planner](#43-weekly-planner)
   - 4.4 [Monthly Planner](#44-monthly-planner)
   - 4.5 [Backlog Management](#45-backlog-management)
   - 4.6 [Task Management](#46-task-management)
   - 4.7 [Prerequisites & Dependencies](#47-prerequisites--dependencies)
   - 4.8 [Ad-hoc Time Logging](#48-ad-hoc-time-logging)
   - 4.9 [Priority Management](#49-priority-management)
   - 4.10 [Progress & Insights](#410-progress--insights)
   - 4.11 [AI Assistant](#411-ai-assistant)
   - 4.12 [Push Notifications](#412-push-notifications)
   - 4.13 [User Settings](#413-user-settings)
   - 4.14 [Admin Dashboard](#414-admin-dashboard)
   - 4.15 [Admin Analytics](#415-admin-analytics)
5. [Screen Descriptions](#5-screen-descriptions)
6. [User Stories](#6-user-stories)

---

## Appendices

- [Appendix A — Data Model](#appendix-a--data-model)
- [Appendix B — API Contracts](#appendix-b--api-contracts)
- [Appendix C — Decision Log](#appendix-c--decision-log)

---

## 1. Introduction

### 1.1 Purpose

This document describes the functional behaviour of **My Time Garden**, a SaaS web application that helps individuals plan and track progress across personal life domains. It bridges the Business Requirements Document (BRD v1.0) and technical implementation, describing **what** the system does and how users interact with it.

For full business context, terminology, and decision rationale see the BRD. Infrastructure and technical implementation details are covered in `planning/specification.md`.

### 1.2 Design Principles

- **Planning first.** The primary value is helping the user decide *what* to do; execution tracking is the consequence.
- **Task-driven progress.** Progress means marking Tasks done, not accumulating minutes. Time is an attribute of a Task, not the planning unit.
- **AI-assisted planning.** The AI assistant is the highest-value feature after the planning tool itself.
- **Warm and encouraging.** All system messages, AI responses, and notifications use a positive, motivating tone.
- **Flexible, not prescriptive.** The system suggests and tracks — it never locks the user in.

### 1.3 Out of Scope (V1)

- Native iOS / Android apps
- Calendar sync (Google Calendar, Outlook)
- Team or family shared accounts
- Paid plans / subscription billing
- Offline-first / PWA support
- Email notifications
- Data export (CSV / PDF)
- Advanced AI features (voice, image input)

---

## 2. User Roles

### 2.1 Regular User

A registered individual managing their own time. All data is private and scoped to their account. A regular user can:

- Sign in via OAuth (Google)
- Complete onboarding to create personal categories and initial backlog
- Plan, schedule, and complete Tasks
- Use the AI assistant (subject to their assigned quota tier)
- View progress, streaks, and completion rates
- Configure categories, daily free-time budget, and notification preferences
- Delete their account and all associated data

### 2.2 Admin

Has all regular user capabilities plus access to the Admin Dashboard. Admin status is assigned manually — no self-service elevation. An admin can additionally:

- View and search all registered user accounts
- Deactivate and reactivate user accounts
- Assign AI message quota tiers (zero, low, high) to users
- Configure the daily message limits for the low and high quota tiers
- View platform-wide KPIs and analytics

---

## 3. User Journeys

### 3.1 First-Time User — Onboarding

```
Landing page
  └── "Sign in with Google"
        └── Google OAuth consent screen
              └── App detects first sign-in
                    └── Step 1: Welcome screen
                          └── Step 2: Category setup
                                (pre-filled defaults with descriptions;
                                 keep / rename / delete / add own)
                                └── Step 3: Daily free-time budget
                                      └── Step 4: Initial backlog
                                            (≥1 Backlog Item per category)
                                            └── Step 5: Notification preferences
                                                  └── Daily Planner
```

### 3.2 Returning User — Daily Planning

```
Opens app → Daily Planner (today)
  ├── Reviews scheduled Tasks per category
  │     └── Opens Backlog sidebar
  │           └── Schedules a new Task from backlog (date + duration)
  ├── Marks a Task done
  │     └── Confirms actual duration (pre-filled with planned)
  │           └── Time Entry auto-created; completion rate updates
  └── Opens AI Assistant
        └── "Plan the rest of my day"
              └── AI returns day plan
                    └── User clicks "Apply" → Tasks created for today
```

### 3.3 Returning User — Weekly Planning

```
Weekly Planner tab
  └── AI Assistant: "Plan my week"
        └── AI returns week-by-week plan
              └── User adjusts in chat
                    └── "Apply" → Tasks created Mon–Sun
                          └── Weekly view shows planned Tasks per day
```

### 3.4 Task Postponement with Cascade

```
Daily Planner → user moves a Task to a later date
  └── System detects dependent Tasks
        └── All dependents shift by the same delta automatically
              └── Optionally: "Ask AI to revise schedule"
                    └── AI suggests adjusted dependency chain
```

### 3.5 End-of-Day Review

```
Evening (user-configured time):
  ├── Push notification: "How did today go? Review your Tasks."
  └── In-app: banner appears when app is opened after a set hour
        └── End-of-day review panel
              ├── Today's incomplete Tasks → complete / reschedule / delete
              └── Tomorrow's plan → adjust if needed
```

### 3.6 Admin — AI Tier Management

```
Admin Dashboard → KPI: X users on zero AI tier
  └── Users table → filter by AI tier = zero
        └── Select user → Assign Low or High tier
              └── Change takes effect immediately
```

---

## 4. Feature Specifications

### 4.1 Authentication & Onboarding

**BR refs:** BR-AUTH-01–05, BR-ONB-01–06

#### Authentication

- Sign-in exclusively via OAuth 2.0 (Google in V1). No username/password accounts.
- On successful OAuth return, the backend issues a JWT access token (15-minute lifetime) and a refresh token (30-day lifetime).
- The user's local timezone is detected and stored on every sign-in. Existing Task dates are not adjusted — only "today" determination and notifications are affected (see Appendix C, D-13).
- If the account is deactivated, sign-in is rejected: *"Your account has been deactivated. Please contact support."*
- On subsequent visits the access token is refreshed silently. If the refresh token has also expired the user is redirected to login.
- Sign-out clears local tokens and invalidates the refresh token server-side.

#### First-Time Onboarding

A mandatory multi-step flow shown once, immediately after the first successful sign-in. All steps must be completed before accessing the main application. The same settings are editable in Settings after completion.

**Step 1 — Welcome**
App name, tagline, and a "Get started" button.

**Step 2 — Category Setup**
Five suggested defaults are pre-filled with names, emojis, colours, and descriptions. The user may keep, rename, delete, or replace any default, and add their own categories. At least one category must be created before proceeding.

Default categories and suggested descriptions:

| Emoji | Name | Suggested description |
|---|---|---|
| 🏠 | Housekeeping | Keeping the home clean, organised, and running smoothly. Includes daily cleaning, laundry, tidying, and household admin. Small regular efforts prevent tasks from piling up. |
| 📚 | English Learning | Building English language skills through daily practice: reading, vocabulary, listening, writing, and speaking exercises. Consistent short sessions build habits more effectively than occasional long study periods. |
| 💼 | Professional Learning | Investing in career and skill development. Online courses, technical reading, certifications, and practising skills relevant to current or future work. Even 30 minutes a day compounds significantly over a year. |
| 🌿 | Gardening | Caring for the garden: watering, planting, pruning, weeding, and seasonal maintenance. A mix of short daily tasks and longer project-based sessions. |
| 👶 | Child-Rearing | Intentional, engaged parenting time: reading together, educational play, homework help, outdoor activities, and one-on-one conversations that support the child's development and wellbeing. |

**Step 3 — Daily Free-Time Budget**
A single value in minutes representing available discretionary time per day. Used as the primary AI scheduling input. Not a hard limit.

**Step 4 — Initial Backlog**
For each category the user must add at least one Backlog Item (name required). The system blocks proceeding with an empty category.

**Step 5 — Notification Preferences**
Browser push notifications opt-in with reminder time configuration. Defaults to off if skipped.

New users are assigned the **zero AI quota tier** by default.

---

### 4.2 Daily Planner

**BR refs:** BR-PLAN-01, BR-PLAN-03–06, BR-TASK-04, BR-TASK-09, BR-TASK-12–17

#### Overview

The primary screen. Shows active categories, scheduled Tasks for the selected day, and real-time completion progress.

#### Date Navigation

- Opens on today's date.
- Horizontal day-selector strip shows Mon–Sun for the current week. Days with any planned or logged entries are marked with an indicator dot.
- Tapping a day loads that day's data without a page reload.
- Navigation to any past day is supported; all history is retained indefinitely.

#### Task Timeline

- Categories are displayed in the user's current priority order (highest first).
- Planned Tasks and logged Time Entries are visually distinct and colour-coded by type (BR-TASK-12).
- Within a category, Tasks are displayed in creation order. The user can drag-to-reorder Tasks within the day.
- Only logged (completed) entries contribute to the completion rate and the remaining budget indicator — planned Tasks do not reduce the budget until marked done.

#### Backlog Panel Sidebar

- A collapsible sidebar lists all Backlog Items for each active category in user-defined order.
- Each Backlog Item shows a badge indicating how many active (Planned) Tasks it currently has (e.g., "×2"), so the user can avoid over-scheduling the same item.
- The user drags a Backlog Item from the sidebar to the timeline (or uses a Schedule form) to create a Task: assigns date = selected day and a duration.

#### Remaining Budget Indicator

A visual bar showing how much of the daily free-time budget remains after accounting for completed (logged) Time Entries. It does not count planned-but-not-done Tasks. It is a planning aid, not a hard limit.

#### Creating a Task Manually

- Tasks can be created for any date from today up to 1 year ahead.
- A Task can be linked to a Backlog Item or created as a **Generic Time Block** (no Backlog Item link, free-text label).
- Scheduling a Backlog Item with an unmet prerequisite is blocked; the system lists the outstanding prerequisites. The filter is enforced in the UI — only completable Backlog Items are offered when scheduling.

#### Marking a Task or Generic Time Block Done

- Tapping "Done" opens a confirmation dialog pre-filled with the planned duration.
- The user confirms or adjusts the actual time spent.
- On confirmation: a Time Entry is created for the Task's category using the actual duration; the original planned duration is preserved separately; the Task state changes to Done.
- Generic Time Blocks follow the identical mark-done flow.

#### Editing a Task

Duration and linked Backlog Item are editable before completion. Original planned duration is preserved separately when the actual differs.

#### Overdue Tasks

A Task whose scheduled date has passed without being marked done is flagged as overdue. The user may complete it, reschedule it, or delete it. Overdue Tasks are not automatically rescheduled.

#### End-of-Day Review

Triggered two ways (see §4.12 for notification trigger):
1. A push notification at the user-configured evening time.
2. An in-app banner that appears when the app is opened after the same hour.

The review panel shows today's incomplete Tasks (complete / reschedule / delete each) and tomorrow's scheduled Tasks (adjust if needed).

---

### 4.3 Weekly Planner

**BR refs:** BR-PLAN-02, BR-PLAN-03

- Accessed via a "Week" tab toggle alongside the Daily Planner.
- Week is fixed Monday–Sunday (not user-configurable in V1).
- A week selector allows navigation to past and future weeks (Tasks can be created up to 1 year ahead).
- **Future days** show planned Tasks (Planned state).
- **Past days** show logged Time Entries alongside any remaining unfinished Tasks.
- **Today** shows both planned Tasks and logged entries.
- Completion rates are shown per category per day.
- Daily Commitments (sum of planned Task durations) are shown per category per day.

---

### 4.4 Monthly Planner

**BR refs:** BR-PLAN-07

- Accessed via a "Month" tab.
- Standard calendar grid (Mon–Sun columns).
- Each day cell shows category emoji dots or short Task labels for scheduled Tasks.
- Tapping a day cell opens the Daily Planner for that day.
- Navigate forward and backward by month.
- Tasks can be created or edited by tapping a day cell.

---

### 4.5 Backlog Management

**BR refs:** BR-TASK-01–03, BR-TASK-06, BR-TASK-08, BR-TASK-10, BR-TASK-11

#### Overview

Each category has a **Backlog** — a user-managed catalogue of activities. Backlog Items are persistent; they are not consumed when Tasks are created from them. The same Backlog Item can be scheduled into multiple Tasks across different dates simultaneously.

#### Creating and Editing Backlog Items

- **Required:** name.
- **Optional:** description (used by the AI as scheduling context), estimated duration (minutes), due date, recurrence schedule, prerequisite Backlog Items (same category only).
- Backlog Items can be created, edited, and deleted at any time.

#### Recurrence

Supported recurrence types in V1: **Daily**, **Weekly**, **Monthly**, **Every N days**.

When a recurring Backlog Item's Task is marked done, the system automatically creates a new Backlog Item entry for the next occurrence (ready to be scheduled by the user or AI — not automatically scheduled as a Task).

#### Ordering

Backlog Items within a category are user-ordered via drag-to-reorder. The order is reflected in the backlog panel sidebar and in AI planning suggestions.

#### Archiving

A Backlog Item is archived when the user explicitly marks it as permanently done. Archived items no longer appear in the active backlog panel, but their associated Tasks and Time Entries are preserved in history.

#### Backlog Panel and Dedicated Backlog View

- The backlog panel sidebar is accessible during daily planning (§4.2).
- A dedicated full-screen **Backlog** page shows all categories' backlogs with full detail (estimated durations, due dates, recurrence, active Task count, prerequisites). This is also where the user manages (creates, edits, reorders, archives) Backlog Items outside the planning flow.

---

### 4.6 Task Management

**BR refs:** BR-TASK-04–05, BR-TASK-09, BR-TASK-12–17

#### Scheduling a Task

- A Task is created by assigning a Backlog Item a date and duration.
- A **Generic Time Block** (free-text label, no Backlog Item link) can also be scheduled. It behaves identically to a Task for all purposes (mark done, time logging, completion rate).
- Tasks may be created for any date from today up to 1 year ahead.
- AI-applied plans create Tasks automatically (see §4.11).

#### Task States

`Planned` → `Done`

#### Editing a Task

Duration and linked Backlog Item are editable before completion. The original planned duration is always preserved separately.

#### Marking Done

See §4.2 — identical flow for Tasks and Generic Time Blocks.

#### Overdue Tasks

A Task that passes its scheduled date without being marked done is displayed as overdue. The user may: complete it (confirm actual time), reschedule it to a new date, or delete it.

#### Visual Distinction

Planned Tasks and logged Time Entries (completed or ad-hoc) are visually distinct and colour-coded by type throughout all planner views.

---

### 4.7 Prerequisites & Dependencies

**BR refs:** BR-TASK-18–22

#### Backlog Item Prerequisites (Backlog level)

- A Backlog Item may depend on one or more other Backlog Items **in the same category**.
- The system prevents scheduling a Task for a Backlog Item if any of its prerequisites have not been marked done (the prerequisite Backlog Item must have at least one Task in Done state).
- The UI enforces this silently: when scheduling, only eligible Backlog Items are offered; ineligible ones are shown greyed out with a tooltip listing outstanding prerequisites.

#### Task Dependencies (Task level)

- Scheduled Tasks **in the same category** can be linked to define dependency relationships.
- Dependencies are created in the planner UI by linking Tasks.
- The UI only offers Tasks from the same category when creating a dependency.
- When the user manually moves a Task to a later date, all Tasks that depend on it (directly or transitively) automatically shift by the same time delta.
- The user may additionally ask the AI to suggest a revised schedule for the entire dependency chain after a postponement.

#### Circular Dependency Prevention

Circular dependencies are detected and rejected at both the Backlog Item and Task levels. If an attempted link would create a cycle the system rejects it and informs the user which link caused the conflict.

---

### 4.8 Ad-hoc Time Logging

**BR refs:** BR-LOG-01–05

Ad-hoc logging is optional and secondary. The primary workflow is creating and completing planned Tasks.

- Log time against any active category on today or any past date. Future dates are not permitted.
- Quick-log shortcuts: **+15m**, **+30m**, **+60m**; or a custom minute input.
- An optional text note can be attached to any Time Entry.
- Any Time Entry can be edited (duration, note) or deleted.
- Deleting a Time Entry immediately updates progress indicators.

---

### 4.9 Priority Management

**BR refs:** BR-PRI-01–03

- The user ranks their active categories in order of personal priority (rank 1 = highest) via drag-to-reorder on the Priorities page.
- A **Save** button commits the new order. Unsaved changes trigger a confirmation dialog on navigation away.
- Priority order is reflected throughout the app: display order in the daily planner, and as a primary input to AI planning suggestions.

---

### 4.10 Progress & Insights

**BR refs:** BR-INS-01–04, BR-PLAN-04

#### Task Completion

For each category: Tasks completed vs. Tasks planned (task count), alongside Daily and Weekly Commitments (sum of scheduled Task durations). Available for both daily and weekly periods.

#### Completion Rate

Percentage of planned Tasks marked done per category per day (count-based, not time-based). Example: 3 planned, 2 done = 67%. If a category has zero Tasks planned on a given day, no completion rate is shown (not 0%) and the day does not count against the streak.

#### Plan vs. Actual

For each completed Task, both the planned duration and actual duration are stored and displayed for comparison.

#### Streaks

Consecutive calendar days on which at least one Task was marked done for the category. Best-ever streak is also shown. A streak resets if no Task is completed for that category on a given day. Days with zero Tasks planned do not break a streak.

#### Week-over-Week Comparison

Total logged time (current week vs. previous week) shown on the Progress page.

#### Visual Updates

Charts, progress bars, and percentages update in real time as Tasks are marked done or ad-hoc time is logged.

---

### 4.11 AI Assistant

**BR refs:** BR-AI-01–10

#### Overview

An AI-powered chat assistant for planning at any time horizon. It is the second most important feature after the planning tool itself.

#### Quota System

| Tier | Access | Default for new users |
|---|---|---|
| **Zero** | No AI access. In-app message: *"AI planning is not yet enabled for your account. Contact your admin to get access."* The rest of the app is fully usable. | Yes |
| **Low** | Up to the daily low-tier message limit (admin-configurable). | No |
| **High** | Up to the daily high-tier message limit (admin-configurable). | No |

- Message counts reset daily.
- Tier changes take effect immediately. A downgrade to zero locks the user out at once. An upgrade grants the remaining allowance of the new tier (new limit − messages already sent today).
- A counter showing remaining messages today is displayed near the chat input.

#### Context (see BR-AI-02 for full field list)

For every AI request the backend assembles a system prompt containing: user timezone, current date, daily free-time budget (per-session chat override takes precedence), all active categories with descriptions and priority ranks, full backlog (Backlog Item names, descriptions, estimated durations, due dates, recurrence, sort order, prerequisite relationships), scheduled Tasks and their dependency relationships, and task completion rates. All context is injected server-side — never exposed to the frontend.

#### Planning Horizons

The AI can suggest plans at any horizon: remainder of today, a full day, a week, a month, or longer (plans beyond one week are decomposed into weekly segments). The AI respects all prerequisite and dependency constraints and never suggests scheduling a Task before its prerequisites are met.

#### One-Click Apply

- When the AI presents a plan it offers an **Apply** button.
- Apply is **all-or-nothing**: existing Tasks on the affected dates are **replaced** by the new plan. The user should review carefully before applying.
- Multi-week plans are applied week by week.
- The user can **Undo** an applied plan in a single action, removing all Tasks created by that application.

#### Chat Interface

- Chat bubble layout: user messages right, assistant messages left.
- Streaming or brief thinking indicator for AI responses.
- Suggested prompt chips on an empty conversation: "Plan my day", "Plan my week", "Review this week", "Motivate me".

#### Conversation History

- Stored indefinitely for user reference.
- Only the most recent messages (configurable limit) are injected into each API request to manage context window size and cost.
- Previous conversations listed in a sidebar (desktop) or back-navigable list (mobile). Users can delete any past conversation.

---

### 4.12 Push Notifications

**BR refs:** BR-NOT-01–04

#### Permission

Invited during onboarding and in Settings. If the browser does not support push, the option is hidden. Notifications default to off if the onboarding step is skipped.

#### Notification Types

| Type | Trigger | Example |
|---|---|---|
| Daily reminder | User-configured time(s) and days | "Time to tend your garden! You haven't started yet today 🌱" |
| End-of-day review | User-configured evening time | "How did today go? Review your Tasks and plan tomorrow." |
| Weekly summary | Monday, user-configured time | "New week, fresh plan! Here's how last week went 📊" |

#### Configuration

- Master enable/disable toggle.
- Each notification type has its own independently configurable time and day-of-week schedule.
- The weekly summary is always sent on **Monday** (aligns with the fixed Mon–Sun week); only the delivery time is user-configurable.
- Clicking a notification opens the app and navigates to the relevant screen.

---

### 4.13 User Settings

**BR refs:** BR-SET-01–04, BR-AUTH-03–04

#### Category Management

- Create, rename, reorder (drag), and delete personal categories at any time.
- Each category: name (required), colour, emoji, description (optional; used by AI).
- Deleting a category archives it — all associated Backlog Items, Tasks, and Time Entries are preserved. The user **cannot browse** historical data of an archived category; the data is only restored if the category is reactivated.
- **Archived categories** are listed in a collapsible "Archived" section in Settings with a **Reactivate** button per category.

#### Daily Free-Time Budget

Single value in minutes, editable at any time.

#### Notification Settings

Master enable/disable toggle, per-type schedule configuration, weekly summary time.

#### Profile

Display name (editable), email (read-only, from OAuth), avatar (read-only, from OAuth), timezone (auto-detected on sign-in; read-only; contact support to override if incorrect).

#### Account

- **Sign out** — clears local tokens; invalidates refresh token server-side.
- **Delete my account** — confirmation dialog ("This will permanently delete all your data. This cannot be undone."). On confirmation, account and all data are deleted and the user is redirected to login.

---

### 4.14 Admin Dashboard

**BR refs:** BR-ADM-01–06, BR-AUTH-05

#### Access

The `/admin` route redirects regular users to the home page. The Admin nav item is rendered only for admin users.

#### Overview Page (`/admin`)

KPI cards: Total users, Active users (last 7 days), Tasks completed today, AI messages sent today, **Users on zero AI tier** (shown as an alert if non-zero count — mitigates R-09).

Charts: Daily active users (last 30 days), Tasks completed per category (last 30 days, system-wide).

#### Users Page (`/admin/users`)

Searchable, sortable, paginated table. Columns: avatar, display name, email, joined date, last active, AI tier, status. Clicking a row opens a detail panel.

**Deactivate / Reactivate** — deactivating immediately invalidates the user's sessions; all data is preserved and fully restored on reactivation.

**Assign AI Tier** — set to zero, low, or high. Change takes effect immediately.

#### AI Tier Settings (`/admin/settings`)

View and edit the daily message limits for the low and high tiers. Changes apply immediately to all users on that tier.

---

### 4.15 Admin Analytics

**BR refs:** BR-ADM-06

Time range selector: Last 7 / 30 / 90 days / Custom range.

**Charts:** Daily active users, new registrations per day, Tasks completed per category per day, AI messages per day, notification delivery success rate.

**Summary table:** One row per category — Tasks completed, unique users with completions, average completion rate.

**Event log:** Paginated, filterable table. Columns: timestamp, masked user, event type, summary. Filterable by event type and date range.

---

## 5. Screen Descriptions

Text wireframes to guide UI design. Not pixel-perfect.

---

### 5.1 Login Screen

```
┌─────────────────────────────────────┐
│                                     │
│         🌱 My Time Garden           │
│    Plan your days, grow your life.  │
│                                     │
│    ┌─────────────────────────────┐  │
│    │  G  Sign in with Google     │  │
│    └─────────────────────────────┘  │
│                                     │
│    By signing in you agree to our   │
│    Terms of Service & Privacy Policy│
└─────────────────────────────────────┘
```

---

### 5.2 Onboarding — Category Setup (Step 2)

```
┌─────────────────────────────────────┐
│  ← Back               Step 2 of 5  │
│                                     │
│  Set up your categories             │
│  These are the life areas you'll    │
│  plan and track. Customise them!    │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ 🏠  Housekeeping      [✎][✕] │   │
│  │ Description: [pre-filled... ]│   │
│  └──────────────────────────────┘   │
│  ┌──────────────────────────────┐   │
│  │ 📚  English Learning  [✎][✕] │   │
│  │ Description: [pre-filled... ]│   │
│  └──────────────────────────────┘   │
│  [ ... other defaults ... ]         │
│                                     │
│  [+ Add your own category]          │
│                                     │
│       [ Continue → ]                │
└─────────────────────────────────────┘
```

---

### 5.3 Onboarding — Daily Budget (Step 3)

```
┌─────────────────────────────────────┐
│  ← Back               Step 3 of 5  │
│                                     │
│  How much free time do you have?    │
│  Your typical discretionary time    │
│  available each day.                │
│                                     │
│  ┌────────────┐                     │
│  │  [ 120 ]   │  minutes per day    │
│  └────────────┘                     │
│                                     │
│  The AI uses this to build plans    │
│  that fit your day. You can override│
│  it anytime in the AI chat.         │
│                                     │
│       [ Continue → ]                │
└─────────────────────────────────────┘
```

---

### 5.4 Onboarding — Initial Backlog (Step 4)

```
┌─────────────────────────────────────┐
│  ← Back               Step 4 of 5  │
│                                     │
│  Add things you want to do          │
│  At least one per category.         │
│                                     │
│  🏠 Housekeeping                    │
│  ┌──────────────────────────────┐   │
│  │ Vacuum living room     [✕]   │   │
│  └──────────────────────────────┘   │
│  [+ Add item]                       │
│                                     │
│  📚 English Learning                │
│  ┌──────────────────────────────┐   │
│  │ Duolingo 10 minutes    [✕]   │   │
│  └──────────────────────────────┘   │
│  [+ Add item]                       │
│                                     │
│  [ ... other categories ... ]       │
│                                     │
│       [ Continue → ]                │
└─────────────────────────────────────┘
```

---

### 5.5 Daily Planner

```
┌──────────────────────────────────────────────┐
│  🌱 My Time Garden               [avatar ▾]  │
│──────────────────────────────────────────────│
│  [ Day ]  [ Week ]  [ Month ]                │
│                                              │
│  Mon  Tue  Wed  Thu  Fri  Sat  Sun           │
│   •         •   [12]                         │
│──────────────────────────────────────────────│
│  Budget: ████████░░░░  90m remaining         │
│──────────────────────────────────────────────│
│                        ┌─────────────────────┐│
│  ┌──────────────────┐  │ BACKLOG             ││
│  │ 🏠 Housekeep. #1 │  │─────────────────────││
│  │  ✓ Vacuum   30m  │  │ 🏠 Housekeeping     ││
│  │  ○ Bathroom 20m  │  │  Vacuum living rm   ││
│  │  + Add task      │  │  ×1 active task     ││
│  │  2/3 done   67%  │  │  Bathroom cleaning  ││
│  └──────────────────┘  │  no active tasks    ││
│                        │─────────────────────││
│  ┌──────────────────┐  │ 📚 English          ││
│  │ 📚 English   #2  │  │  Duolingo 10 min    ││
│  │  ○ Duolingo  10m │  │  no active tasks    ││
│  │  ○ Reading   30m │  │  Read chapter 3     ││
│  │  + Add task      │  │  no active tasks    ││
│  │  0/2 done    0%  │  │─────────────────────││
│  └──────────────────┘  │ 💼 Professional     ││
│                        │  ...                ││
│  [ ... categories ... ]└─────────────────────┘│
└──────────────────────────────────────────────┘
```

---

### 5.6 Mark Task Done — Dialog

```
┌─────────────────────────────────────┐
│  ✓ Mark task done                   │
│─────────────────────────────────────│
│  Vacuum living room                 │
│  Category: 🏠 Housekeeping          │
│                                     │
│  How long did it actually take?     │
│  ┌────────────────────────────┐     │
│  │  [ 30 ]  minutes           │     │
│  └────────────────────────────┘     │
│  Planned: 30m                       │
│                                     │
│  [ Cancel ]        [ ✓ Confirm ]    │
└─────────────────────────────────────┘
```

---

### 5.7 Weekly Planner

```
┌────────────────────────────────────────────────────┐
│  [ Day ]  [ Week ]  [ Month ]                      │
│  ← Week of Mar 9 – Mar 15, 2026 →                 │
│────────────────────────────────────────────────────│
│              Mon   Tue   Wed   Thu   Fri   Sat  Sun │
│ 🏠 Housekeep                                       │
│   Commitment  50m   20m   20m    0m    0m    -    - │
│   Done        50m    0m    —     —     —     -    - │
│   Rate       100%    0%   —     —     —     -    - │
│                                                    │
│ 📚 English                                         │
│   Commitment  40m   40m   40m   40m    0m    -    - │
│   Done        40m   40m    0m    —     —     -    - │
│   Rate       100%  100%    0%    —     —     -    - │
│                                                    │
│ [ ... other categories ... ]                       │
│────────────────────────────────────────────────────│
│  [+ Schedule tasks for any future day]             │
└────────────────────────────────────────────────────┘
```

---

### 5.8 Monthly Planner

```
┌──────────────────────────────────────────────────────┐
│  [ Day ]  [ Week ]  [ Month ]                        │
│  ← March 2026 →                                      │
│──────────────────────────────────────────────────────│
│  Mon     Tue     Wed     Thu     Fri     Sat     Sun  │
│                                                      │
│    2       3       4       5       6       7       8  │
│  🏠📚    💼    🏠📚    📚💼    🌿      —      👶    │
│                                                      │
│    9      10      11      12      13      14      15  │
│  🏠📚   ✓🏠    📚💼    🏠💼    💼      —      👶    │
│ [today]                                              │
│                                                      │
│   16      17      18  ...                            │
│  🏠📚    💼      —   ...                            │
│                                                      │
│  Tap any day to open the Daily Planner.              │
└──────────────────────────────────────────────────────┘
```

---

### 5.9 Backlog Page

```
┌──────────────────────────────────────────────────┐
│  📋 Backlog                      [+ Add category] │
│  🏠  📚  💼  🌿  👶                              │
│──────────────────────────────────────────────────│
│  🏠 Housekeeping                  [+ Add item]   │
│                                                  │
│  ⠿  Vacuum living room                          │
│     est. 30m  ×1 active task  Due: —   [✎][✕]   │
│                                                  │
│  ⠿  Clean bathroom                              │
│     est. 20m  no active tasks  Due: Mar 15 [✎][✕]│
│                                                  │
│  ⠿  Laundry                      🔁 Weekly      │
│     est. 45m  no active tasks  Due: —   [✎][✕]   │
│     Prereq: none                                 │
│                                                  │
│  ⠿  Iron shirts                                 │
│     est. 20m  Prereq: ⚠ Laundry  [✎][✕]         │
│──────────────────────────────────────────────────│
│  📚 English Learning              [+ Add item]   │
│  [ ... ]                                         │
└──────────────────────────────────────────────────┘
```

---

### 5.10 AI Assistant

```
┌─────────────────────────────────────┐
│  🌱 Garden Assistant   ● Online     │
│  8 messages remaining today         │
│─────────────────────────────────────│
│    Hi! I know your categories,      │
│    backlog, and today's plan.       │
│    What would you like to plan? 🌱  │
│                                     │
│  [Plan my day]  [Plan my week]      │
│  [Review week]  [Motivate me]       │
│                                     │
│                  ┌───────────────┐  │
│                  │ Plan my week  │  │
│                  └───────────────┘  │
│                                     │
│  ┌────────────────────────────────┐ │
│  │ Here's a plan for Mon–Sun:     │ │
│  │                                │ │
│  │ Monday (120m available)        │ │
│  │ • 🏠 Vacuum living rm    30m  │ │
│  │ • 📚 Duolingo + reading  40m  │ │
│  │ • 💼 Course module 1     50m  │ │
│  │                                │ │
│  │ Tuesday ...                    │ │
│  │                                │ │
│  │ ⚠ This will replace any       │ │
│  │   existing Tasks on these days │ │
│  │                                │ │
│  │ [✓ Apply plan]  [✕ Dismiss]   │ │
│  └────────────────────────────────┘ │
│─────────────────────────────────────│
│  ┌──────────────────────────┐[Send] │
│  │ Type a message…          │       │
│  └──────────────────────────┘       │
└─────────────────────────────────────┘
```

---

### 5.11 Progress Page

```
┌─────────────────────────────────────┐
│  📊 Progress                        │
│  [ Daily ]  [ Weekly* ]             │
│  ← Week of Mar 9 – Mar 15 →        │
│─────────────────────────────────────│
│  🏠 Housekeeping                    │
│     Tasks: 5 done / 7 planned  71%  │
│     ██████████░░░░                  │
│     Commitment: 110m planned        │
│     Actual logged: 95m              │
│     🔥 Streak: 5 days  Best: 12     │
│                                     │
│  📚 English Learning                │
│     Tasks: 7 done / 7 planned 100%  │
│     ████████████████                │
│     🔥 Streak: 7 days  Best: 21     │
│                                     │
│  [ ... remaining categories ... ]   │
│─────────────────────────────────────│
│  This week: 1,055m  Last week: 890m │
│             ↑ +19% vs last week     │
└─────────────────────────────────────┘
```

---

### 5.12 Priorities Page

```
┌─────────────────────────────────────┐
│  🎯 Priorities                      │
│  Drag to reorder your categories.   │
│  The AI plans in this order.        │
│─────────────────────────────────────│
│                                     │
│  ⠿  1  🏠 Housekeeping             │
│  ⠿  2  📚 English Learning         │
│  ⠿  3  💼 Professional Learning    │
│  ⠿  4  🌿 Gardening               │
│  ⠿  5  👶 Child-Rearing           │
│                                     │
│  [ Save Priority Order ]            │
│                                     │
│─────────────────────────────────────│
│  This week's Tasks completed        │
│  🏠 Housekeeping       5 / 7        │
│  📚 English Learning   7 / 7  ✓    │
│  💼 Professional       2 / 5        │
│  🌿 Gardening          1 / 2        │
│  👶 Child-Rearing      4 / 4  ✓    │
└─────────────────────────────────────┘
```

---

### 5.13 User Settings Page

```
┌─────────────────────────────────────┐
│  ⚙ Settings                        │
│─────────────────────────────────────│
│  Profile                            │
│  Name:     [ Jane Smith          ]  │
│  Email:    jane@example.com         │
│  Timezone: Europe/Budapest (auto)   │
│─────────────────────────────────────│
│  Categories                         │
│  ⠿  🏠 Housekeeping   [✎] [archive]│
│  ⠿  📚 English        [✎] [archive]│
│  ⠿  💼 Professional   [✎] [archive]│
│  ⠿  🌿 Gardening      [✎] [archive]│
│  ⠿  👶 Child-Rearing  [✎] [archive]│
│  [+ Add category]                   │
│                                     │
│  ▼ Archived (2)                     │
│    Cooking     [Reactivate]         │
│    Fitness     [Reactivate]         │
│─────────────────────────────────────│
│  Daily free-time budget             │
│  [ 120 ] minutes per day   [Save]   │
│─────────────────────────────────────│
│  Notifications                      │
│  Enable notifications  [●]          │
│  Daily reminder:  08:00  [Mon–Fri]  │
│  End-of-day:      21:00  [Mon–Sun]  │
│  Weekly summary:  08:00  [Mon]      │
│─────────────────────────────────────│
│  Account                            │
│  [ Sign out ]   [ Delete account ]  │
└─────────────────────────────────────┘
```

---

### 5.14 Admin Dashboard

```
┌──────────────────────────────────────────────────────┐
│  🌱 My Time Garden  [Admin]             [avatar ▾]   │
│──────────────────────────────────────────────────────│
│  Overview | Users | AI Settings | Analytics          │
│──────────────────────────────────────────────────────│
│                                                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐  │
│  │  1,240   │ │   318    │ │   892    │ │ ⚠  47  │  │
│  │  Total   │ │  Active  │ │  Tasks   │ │  Zero  │  │
│  │  Users   │ │ (7 days) │ │  Today   │ │  Tier  │  │
│  └──────────┘ └──────────┘ └──────────┘ └────────┘  │
│                                                      │
│  Daily Active Users — last 30 days                   │
│  [Line chart]                                        │
│                                                      │
│  Tasks Completed per Category — last 30 days         │
│  [Grouped bar chart]                                 │
└──────────────────────────────────────────────────────┘
```

---

### 5.15 Admin Users Table

```
┌─────────────────────────────────────────────────────────────┐
│  Users                             [ Search by name/email ] │
│─────────────────────────────────────────────────────────────│
│  Avatar  Name       Email          AI Tier  Status   Action │
│  ──────  ─────────  ─────────────  ───────  ──────── ──────│
│  [img]   Jane Smith jane@ex.com    High     ● Active  [⋮]  │
│  [img]   Bob Lee    bob@ex.com     Low      ● Active  [⋮]  │
│  [img]   Anna K.    anna@ex.com    Zero     ● Active  [⋮]  │
│  [img]   Mark R.    mark@ex.com    Zero     ○ Inactiv [⋮]  │
│                                                             │
│  ← 1 2 3 … 24 →                         Showing 1–20/472  │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. User Stories

### 6.1 Authentication & Onboarding

| ID | As a… | I want to… | So that… |
|---|---|---|---|
| US-01 | visitor | sign in with my Google account | I don't have to create a separate password |
| US-02 | returning user | be automatically kept signed in | I don't have to log in on every visit |
| US-03 | user | sign out | I can keep my account secure |
| US-04 | new user | create my own personal categories with descriptions | the AI understands my goals in each life area |
| US-05 | new user | set my daily free-time budget | the AI plans within my actual available time |
| US-06 | new user | add initial backlog items during onboarding | I have something to plan from the very first day |

### 6.2 Planning

| ID | As a… | I want to… | So that… |
|---|---|---|---|
| US-07 | user | see today's scheduled Tasks across all categories | I know what I've planned and what's still to do |
| US-08 | user | schedule a Backlog Item for a specific date and duration | I have a concrete plan, not just a wishlist |
| US-09 | user | create a generic time block without a Backlog Item | I can plan less structured time |
| US-10 | user | see a remaining budget indicator | I don't over-schedule my day |
| US-11 | user | browse my backlog from the planner sidebar | I can pick what to schedule without leaving the planner |
| US-12 | user | plan Tasks up to a year ahead | I can do long-term as well as daily planning |
| US-13 | user | see a weekly view of all my planned and logged entries | I can understand my balance across the whole week |
| US-14 | user | see a monthly calendar of all scheduled Tasks | I can plan and review at a higher level |

### 6.3 Tasks & Backlog

| ID | As a… | I want to… | So that… |
|---|---|---|---|
| US-15 | user | mark a Task done and confirm the actual time spent | the app reflects what I really did vs. what I planned |
| US-16 | user | see overdue Tasks flagged clearly | I don't lose track of things I didn't finish |
| US-17 | user | reschedule or delete an overdue Task | I can keep my plan realistic |
| US-18 | user | define ordering constraints between planned activities | the system prevents me scheduling things in the wrong order, whether at the backlog level (prerequisites) or the scheduled Task level (dependencies) |
| US-19 | user | move a Task and have dependent Tasks shift automatically | I don't need to manually adjust my entire plan |
| US-20 | user | add a recurring Backlog Item | the next occurrence appears in my backlog automatically when I complete one |

### 6.4 Progress & Insights

| ID | As a… | I want to… | So that… |
|---|---|---|---|
| US-21 | user | see how many Tasks I completed vs. planned per category | I can track my execution rate |
| US-22 | user | see plan vs. actual time per completed Task | I can see if my time estimates are accurate |
| US-23 | user | see a streak counter per category | I feel motivated to maintain consecutive days |
| US-24 | user | compare this week's logged time to last week | I can see whether I'm improving |

### 6.5 AI Assistant

| ID | As a… | I want to… | So that… |
|---|---|---|---|
| US-25 | user | ask the AI to plan my day or week | I get a personalised, context-aware plan |
| US-26 | user | apply an AI plan with one click | I don't have to schedule each Task manually |
| US-27 | user | undo an applied AI plan | I can try a different approach without manual cleanup |
| US-28 | user | ask the AI to review my week | I get a summary and encouragement |
| US-29 | user | see my past AI conversations | I can refer back to previous suggestions |
| US-30 | user | know how many AI messages I have left today | I can use them thoughtfully |
| US-31 | user | use the full app even if I have no AI access | planning works independently of the AI tier |

### 6.6 Notifications

| ID | As a… | I want to… | So that… |
|---|---|---|---|
| US-32 | user | receive a push reminder at my chosen time | I don't forget to work on my plan |
| US-33 | user | receive an end-of-day review prompt | I reflect on the day and prepare for tomorrow |
| US-34 | user | receive a weekly summary on Monday | I start the week reflecting on the last one |
| US-35 | user | disable all notifications at any time | I stay in control of how the app communicates with me |

### 6.7 Admin

| ID | As an… | I want to… | So that… |
|---|---|---|---|
| US-36 | admin | see how many users are on the zero AI tier | I can proactively enable AI access for users |
| US-37 | admin | assign an AI quota tier to any user | I control who has AI access and at what level |
| US-38 | admin | configure the daily limits for low and high tiers | I can adjust AI costs as usage grows |
| US-39 | admin | deactivate a user account | I can handle abuse without deleting data |
| US-40 | admin | reactivate a deactivated account | I can reverse a deactivation |
| US-41 | admin | view system-wide analytics | I can track platform health and engagement |

---

## Appendix A — Data Model

### A.1 Core Entities

| Entity | Key Fields | Notes |
|---|---|---|
| `users` | id, email, display_name, avatar_url, oauth_provider, oauth_subject, role, timezone, daily_free_time_budget_minutes, ai_quota_tier, ai_messages_sent_today, ai_messages_date, is_active, created_at | One row per registered user |
| `categories` | id, user_id, name, emoji, color_hex, description, priority_rank, is_archived, created_at | Personal per user; no global catalogue |
| `backlog_items` | id, category_id, user_id, name, description, estimated_duration_minutes, due_date, recurrence_type, recurrence_interval, sort_order, is_archived, created_at | `recurrence_type`: daily \| weekly \| monthly \| every_n_days |
| `backlog_item_prerequisites` | backlog_item_id, prerequisite_id | Self-referential; both items must belong to the same category |
| `tasks` | id, backlog_item_id (nullable), category_id, user_id, label (nullable, for generic blocks), scheduled_date, planned_duration_minutes, actual_duration_minutes (nullable), state, completed_at, created_at | `state`: planned \| done; `backlog_item_id` null = Generic Time Block |
| `task_dependencies` | task_id, depends_on_task_id | Directed acyclic graph; both tasks must be in the same category |
| `time_entries` | id, user_id, category_id, task_id (nullable), logged_date, duration_minutes, note, is_adhoc, created_at | `task_id` set when created by Task completion; null for ad-hoc |
| `ai_conversations` | id, user_id, title, created_at | One per chat session |
| `ai_messages` | id, conversation_id, role, content, created_at | `role`: user \| assistant |
| `ai_plan_applications` | id, user_id, conversation_id, applied_at | Used for undo; tracks which Tasks were created |
| `applied_plan_tasks` | plan_application_id, task_id | Join table for plan undo |
| `push_subscriptions` | id, user_id, endpoint, p256dh_key, auth_key, created_at | One per browser per user |
| `notification_settings` | id, user_id, enabled, daily_reminder_times, daily_reminder_days, end_of_day_time, weekly_summary_time | Per user; weekly summary day is always Monday |
| `analytics_events` | id, user_id, event_type, payload, created_at | Append-only event log |

### A.2 Key Relationships

- A **user** has many **categories** (personal; no shared catalogue).
- A **category** has many **backlog_items** and many **tasks**.
- A **backlog_item** may have many **prerequisites** (self-referential; same category enforced).
- A **task** is optionally linked to a **backlog_item**; the same backlog_item may spawn multiple tasks across dates.
- A **task** may have many **task_dependencies** (predecessor tasks; same category enforced).
- A **time_entry** is optionally linked to the **task** that produced it, or is ad-hoc.
- An **ai_plan_application** links via **applied_plan_tasks** to all tasks it created (for undo).
- On account deletion, all user data is cascade-deleted.

### A.3 AI Quota Tier Values

| Value | Meaning |
|---|---|
| `zero` | No AI access (default for new users) |
| `low` | Up to admin-configured low daily message limit |
| `high` | Up to admin-configured high daily message limit |

---

## Appendix B — API Contracts

### B.1 Base URL & Versioning

```
https://api.mytimegarden.com/v1/
```

All endpoints except `/auth/*` require:
```
Authorization: Bearer <access_token>
```

### B.2 Auth Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/auth/login/google` | Initiate Google OAuth flow |
| GET | `/auth/callback/google` | OAuth callback; returns tokens |
| POST | `/auth/refresh` | Exchange refresh token for new access token |
| POST | `/auth/logout` | Revoke refresh token |
| GET | `/auth/me` | Return current user profile |
| PUT | `/auth/me` | Update display name |
| DELETE | `/auth/me` | Permanently delete account and all data |

### B.3 Category Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/categories` | List user's active categories (with priority ranks) |
| GET | `/categories/archived` | List user's archived categories |
| POST | `/categories` | Create a new category |
| PUT | `/categories/{id}` | Update name, emoji, colour, description |
| DELETE | `/categories/{id}` | Archive category (data preserved) |
| POST | `/categories/{id}/reactivate` | Restore an archived category |
| PUT | `/categories/reorder` | Bulk update priority ranks |

### B.4 Backlog Item Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/categories/{id}/backlog-items` | List backlog items for a category |
| POST | `/categories/{id}/backlog-items` | Create a backlog item |
| PUT | `/backlog-items/{id}` | Update name, description, duration, due date, recurrence |
| DELETE | `/backlog-items/{id}` | Delete a backlog item |
| POST | `/backlog-items/{id}/archive` | Mark permanently done (archive) |
| PUT | `/categories/{id}/backlog-items/reorder` | Bulk update sort order |
| GET | `/backlog-items/{id}/prerequisites` | List prerequisites for a backlog item |
| POST | `/backlog-items/{id}/prerequisites` | Add a prerequisite |
| DELETE | `/backlog-items/{id}/prerequisites/{prereqId}` | Remove a prerequisite |

### B.5 Task Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/tasks?from=&to=&categoryId=&overdue=` | Query tasks; `overdue=true` returns only overdue tasks |
| POST | `/tasks` | Schedule a task (from backlog item or generic block) |
| PUT | `/tasks/{id}` | Update planned duration, label, or linked backlog item |
| DELETE | `/tasks/{id}` | Delete a task |
| POST | `/tasks/{id}/complete` | Mark done; provide actual duration; auto-creates time entry |
| PUT | `/tasks/{id}/reschedule` | Move to a new date; cascade dependents automatically |
| POST | `/tasks/{id}/dependencies` | Add a dependency (predecessor task; same category only) |
| DELETE | `/tasks/{id}/dependencies/{depId}` | Remove a dependency |

### B.6 Time Entry Endpoints (Ad-hoc)

| Method | Path | Description |
|---|---|---|
| GET | `/time-entries?from=&to=` | Query entries by date range |
| POST | `/time-entries` | Create an ad-hoc time entry (today or past only) |
| PUT | `/time-entries/{id}` | Update duration or note |
| DELETE | `/time-entries/{id}` | Delete an entry |

### B.7 Analytics Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/analytics/daily?date=` | Task completion and time totals per category for one day |
| GET | `/analytics/weekly?weekStart=` | Task completion and time totals per category for a week |
| GET | `/analytics/monthly?year=&month=` | Task and time summary per category for a month |
| GET | `/analytics/streak/{categoryId}` | Current and best streak for a category |
| GET | `/analytics/progress?from=&to=` | Plan vs. actual across a date range |

### B.8 Planner Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/planner/end-of-day` | Returns: incomplete Tasks for today + Tasks scheduled for tomorrow |

### B.9 AI Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/ai/conversations` | List user's conversations |
| POST | `/ai/conversations` | Start a new conversation |
| GET | `/ai/conversations/{id}` | Get conversation with messages |
| POST | `/ai/conversations/{id}/messages` | Send a message; receive AI reply |
| DELETE | `/ai/conversations/{id}` | Delete a conversation |
| POST | `/ai/plans/{applicationId}/undo` | Undo an applied plan (removes all created tasks) |

### B.10 Notification Endpoints

| Method | Path | Description |
|---|---|---|
| POST | `/notifications/subscribe` | Register a push subscription |
| DELETE | `/notifications/subscribe` | Unsubscribe current device |
| GET | `/notifications/settings` | Get notification preferences |
| PUT | `/notifications/settings` | Update notification preferences |

### B.11 Admin Endpoints

All require `role = admin`.

| Method | Path | Description |
|---|---|---|
| GET | `/admin/users` | Paginated user list |
| GET | `/admin/users/{id}` | User detail |
| PUT | `/admin/users/{id}/status` | Activate or deactivate user |
| PUT | `/admin/users/{id}/ai-tier` | Assign AI quota tier (zero/low/high) |
| GET | `/admin/ai-tiers` | Get current low/high daily message limits |
| PUT | `/admin/ai-tiers` | Update low/high daily message limits |
| GET | `/admin/analytics/overview` | System KPIs |
| GET | `/admin/analytics/events` | Paginated analytics event log |

### B.12 Standard Error Response

```json
{
  "status": 400,
  "code": "VALIDATION_ERROR",
  "message": "scheduled_date must not be in the past",
  "errors": [
    { "field": "scheduledDate", "message": "Must be today or a future date" }
  ]
}
```

Common error codes: `VALIDATION_ERROR`, `UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`, `PREREQUISITE_NOT_MET`, `DEPENDENCY_CYCLE`, `RATE_LIMITED`, `INTERNAL_ERROR`.

---

## Appendix C — Decision Log

Key decisions made during functional specification review.

| ID | Decision |
|---|---|
| D-01 | Tasks within a category on the daily planner are ordered by creation time. The user can drag-to-reorder within the day. |
| D-02 | The remaining budget indicator reflects only **completed** (logged) time — planned Tasks do not reduce the budget until marked done. |
| D-03 | AI Apply **replaces** existing Tasks on the affected dates. A warning is shown in the Apply confirmation. The action is undoable. |
| D-04 | Recurrence creates a new **Backlog Item** (ready to be scheduled). It does not automatically schedule a Task. |
| D-05 | End-of-day review is triggered by both (a) a push notification at the user's configured evening time and (b) an in-app banner when the app is opened after that hour. |
| D-06 | When a category has zero Tasks planned for a day, no completion rate is shown (not 0%). The day does not count against or break the streak. |
| D-07 | Generic Time Blocks follow the same mark-done flow as Tasks (confirm actual duration, auto-create Time Entry). |
| D-08 | Archived categories: user cannot browse historical data; they can only reactivate the category in Settings to restore access. |
| D-09 | Default onboarding categories include pre-filled descriptions. See §4.1 for the five default texts. |
| D-10 | Supported recurrence types in V1: Daily, Weekly, Monthly, Every N days. |
| D-11 | The backlog panel sidebar shows an active Task count badge per Backlog Item (e.g., "×2"). |
| D-12 | Same-category constraint for prerequisites and dependencies is enforced silently in the UI (only same-category items are offered in selectors; no runtime validation error needed). |
| D-13 | Timezone change on sign-in: Task `scheduled_date` stores a calendar DATE and is never adjusted. Timezone only affects "today" boundary determination, AI scheduling suggestions, and notification dispatch timing. |
| D-14 | Weekly summary notification day is fixed as Monday (aligned with the Mon–Sun week). Only the delivery time is user-configurable. |
| D-15 | `planning/specification.md` is out of date and should be rewritten to match the current technical architecture before implementation begins. |

---

*End of Functional Specification.*
*Raise change requests against this document for any scope changes before implementation begins.*
*All changes must be reflected back in the Business Requirements Document (BRD v1.0).*
