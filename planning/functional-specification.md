# My Time Garden — Functional Specification

**Document type:** Functional Specification
**Version:** 1.0
**Date:** March 2026
**Audience:** Developers, stakeholders, product owner
**Status:** Draft

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [User Roles](#2-user-roles)
3. [User Journeys](#3-user-journeys)
4. [Feature Specifications](#4-feature-specifications)
   - 4.1 [Authentication & Onboarding](#41-authentication--onboarding)
   - 4.2 [Daily Planner](#42-daily-planner)
   - 4.3 [Weekly Planner](#43-weekly-planner)
   - 4.4 [Time Tracking & Logging](#44-time-tracking--logging)
   - 4.5 [Priority Management](#45-priority-management)
   - 4.6 [Progress & Charts](#46-progress--charts)
   - 4.7 [AI Assistant](#47-ai-assistant)
   - 4.8 [Push Notifications](#48-push-notifications)
   - 4.9 [User Settings](#49-user-settings)
   - 4.10 [Admin Dashboard](#410-admin-dashboard)
   - 4.11 [Admin Analytics](#411-admin-analytics)
5. [Screen Descriptions](#5-screen-descriptions)
6. [User Stories](#6-user-stories)

---

## Appendices

- [Appendix A — Data Model](#appendix-a--data-model)
- [Appendix B — API Contracts](#appendix-b--api-contracts)

---

## 1. Introduction

### 1.1 Purpose

This document describes the functional behaviour of **My Time Garden**, a SaaS web application that helps individuals plan, log, and reflect on time spent across five life domains. It is written to align developers, the product owner, and any external stakeholders on what the system does and how users interact with it.

This document does not cover infrastructure, deployment pipelines, or low-level technical implementation — those are covered in the Application Specification (`agent_docs/specification.md`).

### 1.2 Background

The application emerged from a personal need: balancing household responsibilities, learning goals, and family commitments within a limited daily window of 2–4 hours of discretionary time. The five categories are:

| Emoji | Category | Description |
|---|---|---|
| 🏠 | Housekeeping | Cleaning, laundry, organising, household admin |
| 📚 | English Learning | Reading, vocabulary, listening, writing practice |
| 💼 | Professional Learning | Courses, articles, skills development, career growth |
| 🌿 | Gardening | Planting, pruning, watering, outdoor maintenance |
| 👶 | Child-Rearing | Active parenting time, reading with children, school prep |

### 1.3 Design Principles

- **Simple over complex.** Every screen should be understood at a glance.
- **Progress over perfection.** Partial completion is shown positively, not as failure.
- **Flexible, not prescriptive.** The system suggests and tracks — it never locks the user in.
- **Encouraging tone.** All system messages, AI responses, and notifications are warm and motivating.

### 1.4 Out of Scope (V1)

- Native mobile apps (iOS / Android)
- Calendar sync (Google Calendar, Outlook)
- Team or family shared accounts
- Custom category creation
- Paid plans or billing

---

## 2. User Roles

### 2.1 Regular User

A registered individual who uses the app to manage their own time. All user data is private and scoped to their account. A regular user can:

- Log in via OAuth (Google)
- View and interact with all planner, tracking, progress, priorities, and AI features
- Configure their own goals, notification preferences, and priorities
- Delete their own time entries and conversations

### 2.2 Admin

An administrator has all the capabilities of a regular user plus access to the Admin Dashboard. Admin status is assigned manually — there is no self-service upgrade path. An admin can:

- View a list of all registered users
- Activate or deactivate any user account
- View system-wide usage analytics
- Manage global category definitions (labels, emojis, colours)

---

## 3. User Journeys

### 3.1 First-Time User

```
Landing page
  └── Clicks "Sign in with Google"
        └── Google OAuth consent screen
              └── Redirected back → app loads for first time
                    └── Brief welcome screen (one-time)
                          └── Prompted to set weekly time goals per category
                                └── Prompted to set notification preferences
                                      └── Land on Daily Planner
```

### 3.2 Returning User — Daily Check-in

```
Opens app / clicks notification link
  └── Lands on Daily Planner (today's date selected)
        └── Reviews today's progress rings
              ├── Logs time against one or more categories
              │     └── Progress rings and bars update in real time
              └── Opens AI Assistant
                    └── Asks "What should I focus on this afternoon?"
                          └── Receives personalised suggestion based on goals + logged time
```

### 3.3 Returning User — Weekly Review

```
End of week (or any time)
  └── Switches to Weekly Planner tab
        └── Reviews bar chart — goal vs. actual per category
              └── Navigates to Progress page
                    └── Reviews streaks and category summaries
                          └── Adjusts goals for next week via Settings
```

### 3.4 Returning User — Reprioritising

```
Navigates to Priorities page
  └── Drags categories into new order
        └── Saves new order
              └── Daily Planner immediately reflects new order
```

### 3.5 Admin — User Management

```
Logs in with admin account
  └── Navigates to /admin
        └── Views KPI summary cards
              └── Opens Users table
                    ├── Searches for a specific user
                    └── Deactivates an account
                          └── User loses ability to log in
```

---

## 4. Feature Specifications

### 4.1 Authentication & Onboarding

#### Description

Users sign in exclusively through OAuth 2.0 social login. No username/password accounts exist. On first sign-in, users are guided through a short onboarding flow before reaching the main app.

#### Behaviour

**Sign-in:**
- The login page displays a "Sign in with Google" button (and optionally GitHub/Microsoft in future).
- Clicking the button initiates the OAuth flow. The user is redirected to the provider's consent screen.
- On successful return, the backend issues a JWT access token (15-minute lifetime) and a refresh token (30-day lifetime).
- If the user's account is deactivated by an admin, sign-in is rejected with a clear message: "Your account has been deactivated. Please contact support."
- If the OAuth provider returns an error, the user is shown a friendly error page with a retry option.

**First-time onboarding (shown once, after first successful login):**
1. Welcome screen: the app name, a one-sentence description, and a "Let's get started" button.
2. Goals screen: a form showing all five categories with numeric inputs for daily and weekly time goals (in minutes). Sensible defaults are pre-filled.
3. Notifications screen: toggle to enable browser push notifications; time pickers for up to two daily reminder times.
4. On completing onboarding, the user lands on the Daily Planner.

**Subsequent logins:**
- The user lands directly on the Daily Planner.
- If the access token has expired, a silent refresh is attempted using the refresh token. If the refresh token has also expired, the user is redirected to the login page.

**Sign-out:**
- Available from the user avatar menu (top-right).
- Clears local tokens and redirects to the login page.

---

### 4.2 Daily Planner

#### Description

The Daily Planner is the primary screen of the application. It shows the user's five categories for a selected day, their progress against daily goals, and controls to log time.

#### Behaviour

- The page opens with today's date selected.
- A horizontal day-selector strip at the top shows Mon–Sun for the current week. Days on which at least one time entry exists are marked with a small indicator dot.
- Tapping a day loads that day's data without a page reload.
- The five category cards are displayed in the user's current priority order (highest priority first).
- Each category card contains:
  - The category emoji and label
  - A priority rank badge (e.g. "#1")
  - A circular radial progress ring showing percentage of daily goal achieved
  - The exact time logged vs. goal in minutes (e.g. "25m / 40m")
  - A linear progress bar beneath the ring
  - Three quick-log buttons: **+15m**, **+30m**, **+60m**
  - A custom input field accepting a whole number of minutes, with an **Add** button
- Tapping a quick-log button or submitting the custom input:
  - Immediately updates the progress ring and bar (optimistic update)
  - Saves the entry to the backend
  - If the save fails, the optimistic update is rolled back and an error toast is shown
- If a daily goal is reached or exceeded, the progress ring turns fully green and a brief celebration animation plays (confetti or pulse). This animation plays only once per goal achievement per session.
- Past days are fully editable — there is no lock on historical data.

---

### 4.3 Weekly Planner

#### Description

The Weekly Planner provides a seven-day overview of time logged and goals across all categories for the current (or any selected) week.

#### Behaviour

- Accessed via a "Week View" tab toggle at the top of the Planner page (alongside "Day View").
- Displays the week Monday–Sunday. A week selector allows navigation to previous weeks (future weeks are not selectable).
- The main content is a grouped bar chart: for each category, two bars are shown side by side — the weekly goal (dashed/outlined) and the actual time logged (solid). The chart is interactive: hovering or tapping a bar shows the exact values.
- Beneath the chart, a summary table lists each category with:
  - Total minutes logged this week
  - Weekly goal in minutes
  - Percentage completion
  - A sparkline or mini bar showing daily breakdown (Mon–Sun)
- A "This week vs. last week" comparison row is shown at the bottom of the table, showing total time logged across all categories for both weeks.

---

### 4.4 Time Tracking & Logging

#### Description

Time entries are the core data unit of the application. Each entry records a duration against a category on a specific date. Entries can be created from the Daily Planner, edited, and deleted.

#### Behaviour

**Creating an entry:**
- Via the Daily Planner quick-log buttons or custom input (see 4.2).
- Optionally, users can add a short text note to any entry (e.g. "Cleaned kitchen"). The note field is accessible by expanding the category card or via an edit flow.

**Viewing entries:**
- Individual entries for a day are accessible by tapping a category card's "detail" affordance (e.g. a chevron or expand button).
- A list of that day's entries for that category is shown, each with its duration, time of logging, and note (if any).

**Editing an entry:**
- Any entry can be edited to change its duration or note.
- The date of an entry cannot be changed after creation (to move time to another day, delete and re-create).

**Deleting an entry:**
- Any entry can be deleted from the detail view. A confirmation prompt is shown before deletion.
- Deleting an entry immediately updates the progress ring and totals on the planner.

---

### 4.5 Priority Management

#### Description

The user can define the relative importance of each of the five categories. The priority order is reflected in the Daily Planner (highest priority shown first) and informs the AI assistant's scheduling suggestions.

#### Behaviour

- The Priorities page displays all five categories as a vertical drag-and-drop list.
- Each item shows the category emoji, label, and its current rank number (1 = highest).
- The user drags an item to reorder it. Rank numbers update live as items are dragged.
- A **Save** button commits the new order. Until saved, changes are local only.
- On save, the Daily Planner immediately reflects the new order on next load or refresh.
- Below the priority list, a weekly time summary is shown: total minutes logged this week per category, giving context for prioritisation decisions.
- If the user navigates away with unsaved changes, a confirmation dialog warns them that changes will be lost.

---

### 4.6 Progress & Charts

#### Description

The Progress page gives the user a detailed view of their time investment over time, with charts, streaks, and goal completion summaries.

#### Behaviour

**View toggle:**
- The page supports two views: **Daily** (for a selected date) and **Weekly** (for a selected week).
- Default view is Weekly, showing the current week.

**Bar chart:**
- A grouped bar chart shows goal (outlined) vs. actual (filled) minutes per category.
- The chart is colour-coded per category (matching the category's brand colour).
- Hovering or tapping a bar shows an exact tooltip.

**Category summary rows:**
- Below the chart, each category has a summary row showing:
  - Emoji and label
  - Minutes logged vs. goal
  - A thin progress bar
  - A checkmark (✓) if the goal was met, or a percentage if not

**Streaks:**
- Each category shows a streak counter: the number of consecutive days on which any time was logged for that category.
- The best-ever streak is also shown.
- A streak resets if no time is logged on a given day for that category.

**Goal editing:**
- A collapsible "Edit Goals" section at the bottom of the page allows the user to update daily and weekly goal targets per category without navigating to Settings.

---

### 4.7 AI Assistant

#### Description

An AI-powered chat assistant that helps the user plan their day, review their progress, and stay motivated. The assistant has full context of the user's goals, priorities, and recent time log.

#### Behaviour

**Chat interface:**
- The AI page displays a chat bubble interface: user messages on the right (green), assistant messages on the left (light grey).
- A text input at the bottom with a **Send** button. Pressing Enter also sends.
- The assistant's response streams in or appears after a short "thinking" indicator.

**Context:**
- The backend assembles a system prompt for every request that includes:
  - The user's five categories with their priority ranks and daily/weekly goals
  - Today's logged time per category
  - This week's total logged time per category
  - The user's timezone and today's date
- The user does not see this context — it is injected server-side.

**Suggested prompts:**
- Three tappable suggestion chips are shown above the input on an empty conversation:
  - "Plan my day"
  - "Review this week"
  - "Motivate me"
- Tapping a chip sends that message automatically.

**Conversation history:**
- Previous conversations are listed in a sidebar (desktop) or a back-navigable list (mobile).
- Each conversation is titled automatically from its first exchange.
- Users can delete any past conversation.

**Rate limiting:**
- Users are limited to 50 AI messages per day. A counter is shown near the input: "38 messages remaining today."
- When the limit is reached, the input is disabled and a message explains when it resets (midnight UTC).

---

### 4.8 Push Notifications

#### Description

The app can send browser push notifications to remind the user to log time and celebrate achievements.

#### Behaviour

**Permission request:**
- On completing onboarding (or from Settings), the user is invited to enable push notifications.
- If the browser does not support push notifications, this option is hidden.
- If the user declines, the option remains available in Settings to enable later.

**Notification types:**

| Type | When sent | Example message |
|---|---|---|
| Daily reminder | At the user's configured reminder time(s) | "Time to tend your garden! You haven't logged anything yet today 🌱" |
| Goal achieved | When 100% of a daily category goal is reached | "You hit your Housekeeping goal today! 🏠 ✓" |
| Motivational | Once daily (if enabled) at morning reminder time | "Small steps every day add up to big changes 💪" |
| Weekly summary | Sunday at 8pm (user's timezone) | "This week you logged 3h 20m — your best yet! 📊" |

**Settings:**
- Users can configure:
  - Whether notifications are enabled (master toggle)
  - Up to two daily reminder times
  - Which days of the week reminders are sent
  - Whether motivational messages are included

**Clicking a notification:**
- Opens (or focuses) the app and navigates to the relevant screen (e.g. Daily Planner for reminders, Progress for the weekly summary).

---

### 4.9 User Settings

#### Description

A settings page where the user can manage their profile, goals, notification preferences, and account.

#### Behaviour

**Profile section:**
- Display name (editable text field)
- Email address (read-only, sourced from OAuth)
- Avatar (read-only, sourced from OAuth provider)
- Timezone selector (dropdown, defaults to browser timezone on first sign-in)

**Goals section:**
- Daily and weekly goal (in minutes) for each of the five categories
- Changes saved immediately on blur or via an explicit Save button

**Notification section:**
- Master enable/disable toggle
- Reminder time pickers (up to two per day)
- Day-of-week checkboxes
- Motivational messages toggle

**Account section:**
- "Sign out" button
- "Delete my account" button — triggers a confirmation dialog ("This will permanently delete all your data. This cannot be undone."). On confirmation, the account and all associated data are deleted and the user is redirected to the login page.

---

### 4.10 Admin Dashboard

#### Description

A restricted area accessible only to users with the admin role. Provides an overview of system health and user activity, plus tools for user and category management.

#### Behaviour

**Access control:**
- The `/admin` route and all sub-routes are inaccessible to regular users. Attempting to navigate there redirects to the home page with no error (security by obscurity — do not reveal that an admin section exists).
- The Admin link in the navigation is only rendered for admin users.

**Overview page (`/admin`):**

Five KPI cards at the top:
- Total registered users
- Active users in the last 7 days (any login or time entry)
- Time entries created today
- AI messages sent today
- Active push subscriptions

Two charts below the KPIs:
- Daily active users — line chart, last 30 days
- Time entries per category — grouped bar chart, last 30 days (system-wide)

**Users page (`/admin/users`):**
- A searchable, sortable, paginated table of all users
- Columns: avatar, display name, email, joined date, last active, total time entries, status (Active / Inactive)
- Search filters by name or email
- Clicking a row opens a user detail panel showing full profile and activity summary
- Each row has an action button to Activate or Deactivate the account
- Deactivating a user immediately invalidates their sessions

**Categories page (`/admin/categories`):**
- A list of the five global categories with their current label, emoji, and colour
- Each category is editable inline: label text, emoji picker, hex colour input
- A toggle to activate or deactivate a category system-wide (deactivated categories are hidden from all users)
- Drag-to-reorder to change the default priority order for new users

---

### 4.11 Admin Analytics

#### Description

A detailed analytics view available to admins, showing usage trends across the entire user base.

#### Behaviour

**Page (`/admin/analytics`):**

**Time range selector:** Last 7 days / Last 30 days / Last 90 days / Custom range.

**Charts:**
- Daily active users (line chart)
- New user registrations per day (bar chart)
- Time logged per category per day (stacked area chart)
- AI assistant messages sent per day (line chart)
- Push notification delivery success rate (line chart)

**Summary table:**
- One row per category showing: total minutes logged (in selected range), unique users who logged at least once, average minutes per active user per day

**Event log:**
- A paginated, filterable table of raw analytics events
- Columns: timestamp, user (masked to first name + last initial for privacy), event type, summary
- Filterable by event type and date range

---

## 5. Screen Descriptions

This section describes the layout and key elements of each screen. These are text wireframes intended to guide UI design, not prescribe pixel-perfect layouts.

---

### 5.1 Login Screen

```
┌─────────────────────────────────────┐
│                                     │
│         🌱 My Time Garden           │
│    Grow your days, one task at a    │
│              time.                  │
│                                     │
│    ┌─────────────────────────────┐  │
│    │  G  Sign in with Google     │  │
│    └─────────────────────────────┘  │
│                                     │
│    ─────────── or ──────────────    │
│                                     │
│    ┌─────────────────────────────┐  │
│    │  🐙  Sign in with GitHub    │  │  (optional V1)
│    └─────────────────────────────┘  │
│                                     │
│    By signing in you agree to our   │
│    Terms of Service & Privacy Policy│
└─────────────────────────────────────┘
```

---

### 5.2 Onboarding — Goals Setup

```
┌─────────────────────────────────────┐
│  ← Back          Step 2 of 3        │
│                                     │
│  Set your time goals                │
│  How much time do you want to spend │
│  on each area each week?            │
│                                     │
│  🏠 Housekeeping                    │
│     Daily: [  40 ] min              │
│     Weekly: [ 240 ] min             │
│                                     │
│  📚 English Learning                │
│     Daily: [  50 ] min              │
│     Weekly: [ 300 ] min             │
│                                     │
│  💼 Professional Learning           │
│     Daily: [  30 ] min              │
│     Weekly: [ 180 ] min             │
│                                     │
│  🌿 Gardening                       │
│     Daily: [  20 ] min              │
│     Weekly: [ 120 ] min             │
│                                     │
│  👶 Child-Rearing                   │
│     Daily: [  60 ] min              │
│     Weekly: [ 360 ] min             │
│                                     │
│         [ Save & Continue → ]       │
└─────────────────────────────────────┘
```

---

### 5.3 Daily Planner

```
┌─────────────────────────────────────┐
│  🌱 My Time Garden      [avatar ▾]  │
│─────────────────────────────────────│
│  [ Day View ]  [ Week View ]        │
│                                     │
│  Mon  Tue  Wed  Thu  Fri  Sat  Sun  │
│   •         •   [10]                │  ← today highlighted, dots = has data
│─────────────────────────────────────│
│  [⏱ Track] [📊 Progress] [🎯 Pri.] [🤖 AI] │
│─────────────────────────────────────│
│                                     │
│  ┌─────────────────────────────────┐│
│  │ 🏠 Housekeeping         #1     ││
│  │ ────────────────────  (○ 63%) ││  ← radial ring
│  │ 25m / 40m goal                 ││
│  │ ████████░░░░░░░░░░░░           ││  ← progress bar
│  │ [+15m]  [+30m]  [+60m]  [_ +] ││
│  └─────────────────────────────────┘│
│                                     │
│  ┌─────────────────────────────────┐│
│  │ 📚 English Learning     #2     ││
│  │ ────────────────────  (○  0%) ││
│  │  0m / 50m goal                 ││
│  │ ░░░░░░░░░░░░░░░░░░░░           ││
│  │ [+15m]  [+30m]  [+60m]  [_ +] ││
│  └─────────────────────────────────┘│
│                                     │
│  [ ... remaining categories ... ]   │
└─────────────────────────────────────┘
```

---

### 5.4 Weekly Planner

```
┌─────────────────────────────────────┐
│  [ Day View ]  [ Week View ]        │
│  ← Week of Mar 9 – Mar 15, 2026 →  │
│─────────────────────────────────────│
│                                     │
│  [Grouped bar chart: goal vs actual]│
│   🏠  📚  💼  🌿  👶               │
│   ▓░  ▓░  ▓░  ▓░  ▓░               │
│   ░ = goal   ▓ = logged            │
│                                     │
│─────────────────────────────────────│
│  Category       Logged  Goal   %    │
│  🏠 Housekeeping 185m   240m  77%  │
│  📚 English      300m   300m  ✓   │
│  💼 Professional  90m   180m  50%  │
│  🌿 Gardening     60m   120m  50%  │
│  👶 Child-Rearing 420m  360m  ✓   │
│─────────────────────────────────────│
│  This week: 1,055m  Last week: 890m │
│             ↑ +19% vs last week     │
└─────────────────────────────────────┘
```

---

### 5.5 Priorities Page

```
┌─────────────────────────────────────┐
│  🎯 Task Priorities                 │
│  Drag to reorder. Top tasks get     │
│  scheduled first by the AI.         │
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
│  This week's time                   │
│  🏠 Housekeeping       3h 05m       │
│  📚 English Learning   5h 00m       │
│  💼 Professional       1h 30m       │
│  🌿 Gardening          1h 00m       │
│  👶 Child-Rearing      7h 00m       │
└─────────────────────────────────────┘
```

---

### 5.6 Progress Page

```
┌─────────────────────────────────────┐
│  📊 Progress                        │
│  [ Daily ]  [ Weekly* ]             │
│  ← Week of Mar 9 – Mar 15 →        │
│─────────────────────────────────────│
│                                     │
│  [Bar chart: goal ░ vs spent ▓]     │
│                                     │
│─────────────────────────────────────│
│  🏠 Housekeeping   185m / 240m  77%│
│     ████████████░░░░                │
│     🔥 Streak: 5 days  Best: 12    │
│                                     │
│  📚 English        300m / 300m  ✓  │
│     ████████████████████            │
│     🔥 Streak: 3 days  Best: 21    │
│                                     │
│  [ ... remaining categories ... ]   │
│                                     │
│  ▼ Edit Goals                       │
│    🏠 Daily: [40] min  Weekly: [240]│
│    📚 Daily: [50] min  Weekly: [300]│
│    ...                              │
└─────────────────────────────────────┘
```

---

### 5.7 AI Assistant

```
┌─────────────────────────────────────┐
│  🌱 Garden Assistant   ● Online     │
│─────────────────────────────────────│
│                                     │
│    Hi! I'm your time planning       │
│    assistant. I know your goals     │
│    and today's progress. Ask me     │
│    anything! 🌱                     │
│                                     │
│  [ Plan my day ] [ Review week ]    │
│  [ Motivate me ]                    │
│                                     │
│                  ┌────────────────┐ │
│                  │ What should I  │ │
│                  │ focus on this  │ │  ← user bubble (right)
│                  │ afternoon?     │ │
│                  └────────────────┘ │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ You've hit your English goal │   │  ← assistant bubble (left)
│  │ today — great work! 📚 ✓    │   │
│  │ With 90 min left, I'd suggest│   │
│  │ 40m on Housekeeping (63% to  │   │
│  │ goal) then 50m with your     │   │
│  │ child. 👶                   │   │
│  └──────────────────────────────┘   │
│─────────────────────────────────────│
│  38 messages remaining today        │
│  ┌──────────────────────────┐ [Send]│
│  │ Type a message…          │       │
│  └──────────────────────────┘       │
└─────────────────────────────────────┘
```

---

### 5.8 Admin Dashboard

```
┌─────────────────────────────────────────────────────┐
│  🌱 My Time Garden  [Admin]              [avatar ▾] │
│─────────────────────────────────────────────────────│
│  Overview | Users | Categories | Analytics          │
│─────────────────────────────────────────────────────│
│                                                     │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐  │
│  │ 1,240   │ │  318    │ │  892    │ │  2,104  │  │
│  │ Total   │ │ Active  │ │Entries  │ │  AI Msg │  │
│  │ Users   │ │(7 days) │ │ Today   │ │  Today  │  │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘  │
│                                                     │
│  Daily Active Users — last 30 days                  │
│  [Line chart]                                       │
│                                                     │
│  Time Logged per Category — last 30 days            │
│  [Grouped bar chart]                                │
└─────────────────────────────────────────────────────┘
```

---

### 5.9 Admin Users Table

```
┌───────────────────────────────────────────────────────────┐
│  Users                          [ Search by name/email ]  │
│───────────────────────────────────────────────────────────│
│  Avatar  Name         Email            Joined   Status    │
│  ──────  ──────────   ─────────────── ──────── ────────── │
│  [img]   Jane Smith   jane@ex.com     Jan 2026  ● Active  [⋮]│
│  [img]   Bob Lee      bob@ex.com      Feb 2026  ● Active  [⋮]│
│  [img]   Anna K.      anna@ex.com     Feb 2026  ○ Inactive[⋮]│
│                                                           │
│  ← 1 2 3 … 24 →                        Showing 1–20/472  │
└───────────────────────────────────────────────────────────┘
```

---

## 6. User Stories

### 6.1 Authentication

| ID | As a… | I want to… | So that… |
|---|---|---|---|
| US-01 | visitor | sign in with my Google account | I don't have to create and remember a separate password |
| US-02 | returning user | be automatically kept signed in | I don't have to log in every time I open the app |
| US-03 | user | sign out from any device | I can keep my account secure |
| US-04 | new user | be guided through setting my initial goals | I can start using the app meaningfully right away |

### 6.2 Planning & Tracking

| ID | As a… | I want to… | So that… |
|---|---|---|---|
| US-05 | user | see today's progress for all five categories at a glance | I know what I've done and what still needs attention |
| US-06 | user | quickly log 15, 30, or 60 minutes to a category | I can record time in seconds without friction |
| US-07 | user | log a custom number of minutes | I can record any duration accurately |
| US-08 | user | add a short note to a time entry | I can remember what I actually did |
| US-09 | user | edit or delete a time entry I logged by mistake | my data stays accurate |
| US-10 | user | view my progress for any past day | I can review what I did earlier in the week |
| US-11 | user | see a weekly view of all categories | I can understand my balance across the whole week |

### 6.3 Goals & Priorities

| ID | As a… | I want to… | So that… |
|---|---|---|---|
| US-12 | user | set daily and weekly time goals per category | I have a clear target to work towards |
| US-13 | user | change my goals at any time | I can adapt as my life circumstances change |
| US-14 | user | drag my categories into priority order | the most important ones appear first and guide my AI suggestions |
| US-15 | user | see the current priority order reflected in the daily planner | I always focus on what matters most |

### 6.4 Progress & Insights

| ID | As a… | I want to… | So that… |
|---|---|---|---|
| US-16 | user | see a chart comparing my goal vs. actual time per category | I can spot where I'm falling short visually |
| US-17 | user | see my streak for each category | I feel motivated to keep consecutive days going |
| US-18 | user | compare this week's total to last week's | I can see whether I'm improving over time |

### 6.5 AI Assistant

| ID | As a… | I want to… | So that… |
|---|---|---|---|
| US-19 | user | ask the AI to plan the rest of my day | I get a personalised, context-aware suggestion without having to think it through myself |
| US-20 | user | ask the AI to review my week | I get a summary and encouragement based on my actual data |
| US-21 | user | ask the AI to motivate me | I get a boost when I'm feeling low-energy |
| US-22 | user | see my past AI conversations | I can refer back to previous suggestions |
| US-23 | user | know how many AI messages I have left today | I can use them thoughtfully |

### 6.6 Notifications

| ID | As a… | I want to… | So that… |
|---|---|---|---|
| US-24 | user | receive a push reminder at a time I choose | I don't forget to log time during the day |
| US-25 | user | be notified when I hit a daily goal | I feel rewarded for my effort |
| US-26 | user | receive a weekly summary notification | I reflect on the week without having to open the app |
| US-27 | user | disable notifications at any time | I stay in control of how the app communicates with me |

### 6.7 Admin

| ID | As an… | I want to… | So that… |
|---|---|---|---|
| US-28 | admin | see how many users are active | I understand the health of the platform |
| US-29 | admin | search for and view any user's profile | I can investigate issues or answer support queries |
| US-30 | admin | deactivate a user account | I can handle abuse or at-risk accounts quickly |
| US-31 | admin | edit category labels and emojis | I can correct mistakes or improve the experience for all users |
| US-32 | admin | view system-wide analytics charts | I can track usage trends over time |

---

## Appendix A — Data Model

### A.1 Core Entities

| Entity | Key Fields | Notes |
|---|---|---|
| `users` | id, email, display_name, oauth_provider, oauth_subject, role, timezone | One row per registered user |
| `categories` | id, slug, label, emoji, color_hex, sort_order | Seed data; 5 rows in V1 |
| `user_category_settings` | user_id, category_id, priority_rank, daily_goal_minutes, weekly_goal_minutes | One row per user per category |
| `time_entries` | id, user_id, category_id, logged_date, duration_minutes, note | Many per user per day |
| `ai_conversations` | id, user_id, title, created_at | One per chat session |
| `ai_messages` | id, conversation_id, role, content | Many per conversation |
| `push_subscriptions` | id, user_id, endpoint, p256dh_key, auth_key | One per browser per user |
| `analytics_events` | id, user_id, event_type, payload, created_at | Append-only event log |

### A.2 Key Relationships

- A **user** has many **time_entries**, many **user_category_settings**, many **ai_conversations**, and many **push_subscriptions**
- A **category** has many **user_category_settings** and many **time_entries**
- An **ai_conversation** has many **ai_messages**
- All user data is deleted when a user deletes their account (cascade delete)

### A.3 Seed Data — Categories

| slug | label | emoji | color_hex | sort_order |
|---|---|---|---|---|
| housekeeping | Housekeeping | 🏠 | e07b5a | 1 |
| english | English Learning | 📚 | 5a8fe0 | 2 |
| professional | Professional Learning | 💼 | 7b5ae0 | 3 |
| gardening | Gardening | 🌿 | 4caf7d | 4 |
| child | Child-Rearing | 👶 | e0b85a | 5 |

---

## Appendix B — API Contracts

### B.1 Base URL & Versioning

All API requests are made to:
```
https://api.mytimegarden.com/v1/
```

All endpoints (except `/auth/*`) require:
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
| PUT | `/auth/me` | Update display name or timezone |
| DELETE | `/auth/me` | Delete account and all data |

### B.3 Settings Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/settings/categories` | Get all user category settings |
| PUT | `/settings/categories` | Bulk update goals and priority ranks |

### B.4 Time Entry Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/time-entries?from=&to=` | Query entries by date range |
| POST | `/time-entries` | Create a new entry |
| PUT | `/time-entries/{id}` | Update duration or note |
| DELETE | `/time-entries/{id}` | Delete an entry |

### B.5 Analytics Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/analytics/daily?date=` | Per-category totals for one day |
| GET | `/analytics/weekly?weekStart=` | Per-category totals for a week |
| GET | `/analytics/streak/{categoryId}` | Current and best streak |

### B.6 AI Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/ai/conversations` | List conversations |
| POST | `/ai/conversations` | Start a new conversation |
| GET | `/ai/conversations/{id}` | Get conversation and messages |
| POST | `/ai/conversations/{id}/messages` | Send a message; receive AI reply |
| DELETE | `/ai/conversations/{id}` | Delete a conversation |

### B.7 Notification Endpoints

| Method | Path | Description |
|---|---|---|
| POST | `/notifications/subscribe` | Register a push subscription |
| DELETE | `/notifications/subscribe` | Unsubscribe current device |
| GET | `/notifications/settings` | Get notification preferences |
| PUT | `/notifications/settings` | Update notification preferences |

### B.8 Admin Endpoints

All require `role = admin`.

| Method | Path | Description |
|---|---|---|
| GET | `/admin/users` | Paginated user list |
| GET | `/admin/users/{id}` | User detail |
| PUT | `/admin/users/{id}/status` | Activate or deactivate user |
| GET | `/admin/analytics/overview` | System KPIs |
| GET | `/admin/analytics/events` | Paginated event log |
| GET | `/admin/categories` | List all categories |
| PUT | `/admin/categories/{id}` | Edit a category |

### B.9 Standard Error Response

All errors return a consistent shape:

```json
{
  "status": 400,
  "code": "VALIDATION_ERROR",
  "message": "duration_minutes must be greater than 0",
  "errors": [
    { "field": "duration_minutes", "message": "Must be greater than 0" }
  ]
}
```

Common error codes: `VALIDATION_ERROR`, `UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`, `RATE_LIMITED`, `INTERNAL_ERROR`.

---

*End of Functional Specification. Raise change requests against this document for any scope changes before implementation begins.*
