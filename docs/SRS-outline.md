# Software Requirements Specification
## Link2Itinerary

**Prepared by:** The Travel Taskforce
**Date:** April 10, 2026
**Course:** CEN4090L — Software Engineering II, Florida State University

### Authors

| Team Member | Role |
|-------------|------|
| Betty Phipps | Backend / Architecture |
| Jakob Midthun | Backend / LLM |
| Kateryna Hrishina | Backend / Database |
| Paul Harrington | Frontend / UX |
| Mashhood Syed | Frontend / UI |
| Myles Stowe | Frontend / UI |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Overall Description](#2-overall-description)
3. [External Interface Requirements](#3-external-interface-requirements)
4. [System Features](#4-system-features)
5. [Other Nonfunctional Requirements](#5-other-nonfunctional-requirements)
6. [Other Requirements](#6-other-requirements)
- [Appendix A: Glossary](#appendix-a-glossary)
- [Appendix B: Analysis Models](#appendix-b-analysis-models)
- [Appendix C: To Be Determined List](#appendix-c-to-be-determined-list)

---

## 1. Introduction

### 1.1 Purpose

This document describes the software requirements for Link2Itinerary. Link2Itinerary is a web application that turns travel inspiration — like an Airbnb listing — into a detailed, day-by-day travel itinerary using artificial intelligence. This document covers the full system, which includes a working backend API, a React frontend, a PostgreSQL database, and an authentication flow.

The goal of this document is to clearly explain what the system does, how it is built, and what requirements it must meet. It is written so that both technical team members and non-technical readers can understand it.

### 1.2 Document Conventions

- **Bold text** is used for important terms and section labels.
- Code names (like endpoint paths) appear in plain text exactly as they are in the code.

### 1.3 Intended Audience and Reading Suggestions

- **Developers (backend and frontend):** Read the full document.
- **Testers and QA:** Focus on Section 4 (System Features) and Section 5 (Non-functional Requirements) to build test cases.
- **Course instructors and project evaluators:** Sections 1 and 2 give a clear high-level overview.
- **New team members:** Start with Section 2 to understand the product before reading the technical details.

### 1.4 Product Scope

Link2Itinerary helps travelers turn travel inspiration into real plans. A user pastes a link to an accommodation listing, writes a short description of their trip, and the app uses OpenAI's GPT-4o model to build a personalized travel itinerary. Support for video-based sources was explored but is out of scope for this version. The app also shows estimated costs broken down by category.

**The main goals of the product are:**
- Save travelers time by turning unstructured inspiration into an organized schedule.
- Give users a smart starting point for trip planning instead of searching through multiple websites.
- Provide realistic cost estimates so users can budget their trips before they commit.

### 1.5 References

- OpenAI API documentation: https://platform.openai.com/docs
- NestJS framework documentation: https://docs.nestjs.com
- React documentation: https://react.dev
- Supabase documentation: https://supabase.com/docs

---

## 2. Overall Description

### 2.1 Product Perspective

Link2Itinerary is a brand new, self-contained web application. It is not a replacement for any existing tool. The system has four main parts that work together:

- A React and TypeScript frontend that users interact with in their browser.
- A NestJS REST API backend that handles all the logic, including AI calls.
- A PostgreSQL database hosted on Supabase that stores user accounts, trips, and cached itineraries.
- An integration with the OpenAI API (GPT-4o) that generates the actual itinerary content.

The frontend talks to the backend through a standard web API. The backend talks to the database and to OpenAI. Users never interact with the database or OpenAI directly.

### 2.2 Product Functions

Here is a summary of what Link2Itinerary can do:

- **Trip Seed Creation:** A user submits a travel link and a short description. The app creates a trip record in the database.
- **Teaser Itinerary Generation:** The app produces a quick 3-day preview of the trip, showing daily themes, highlights, vibe tags, and a rough cost range. This works without an account.
- **User Login State:** The frontend supports a client-side login flow using AuthContext. User authentication state is stored in localStorage and is used to gate access to protected planning flows.
- **Preferences Collection:** Logged-in users can tell the app their travel style — budget level, pace (relaxed, moderate, or packed), interests, dietary needs, and accessibility needs.
- **Full Itinerary Generation:** The app builds a detailed day-by-day itinerary with timed activities, locations, estimated costs, and booking links based on the trip and the user's preferences.
- **Cost Estimation:** A dedicated estimator calculates a full cost breakdown by category — accommodation, dining, activities, transportation, shopping, and miscellaneous — based on the itinerary and the user's budget level.
- **Itinerary and Teaser Caching:** Generated teasers and full itineraries are saved to the database so repeat visits do not require a new OpenAI call.

### 2.3 User Classes and Characteristics

**Guest users (not logged in)**
- Can access the landing page, create a trip seed, and view a teaser itinerary.
- Cannot access the full itinerary or preferences flow.
- No technical knowledge required — the interface is designed for everyday travelers.

**Registered users (logged in)**
- Can do everything a guest can, plus access the full itinerary flow.
- Their trips are linked to their account, and their itineraries are saved if they want to.
- Likely to be travelers who want to plan trips more seriously.

### 2.4 Operating Environment

- **Browser:** Any modern browser.
- **Frontend:** Vite development server (Node.js 18+), or any static file host for a production build.
- **Backend:** Node.js 18+ running NestJS on port 3000.
- **Database:** PostgreSQL 14+, hosted on Supabase.
- **AI Provider:** OpenAI API using the GPT-4o model, called over HTTPS.
- The backend is compatible with Linux, macOS, and Windows.

### 2.5 Design and Implementation Constraints

- The backend must use TypeScript and the NestJS framework.
- The frontend must use TypeScript and React with Vite.
- The database must use TypeORM with a Supabase-hosted PostgreSQL database.
- All AI calls must use the OpenAI API with the GPT-4o model.
- All API request bodies must be validated using NestJS class-validator DTOs.
- Passwords must be hashed with bcrypt before being stored — plaintext passwords must never be saved.
- Authentication must use JSON Web Tokens (JWTs). Tokens expire after 30 days.
- API keys and secrets must be stored in environment variables, never in version control.
- The `synchronize: true` option in TypeORM is used in development only. It must be turned off before any production deployment.

### 2.6 User Documentation

- `README.md` files in the root, `backend/`, and `frontend/` directories with setup instructions.
- `backend/GETTING-STARTED.md`: A step-by-step guide for setting up a local development environment.
- `backend/api-contracts.md`: Full documentation of every API endpoint, including request and response formats.
- Inline comments throughout the code explaining architectural decisions and data flow.

### 2.7 Assumptions and Dependencies

- Users have a reliable internet connection for API calls to OpenAI and Supabase.
- The team has access to a valid OpenAI API key with enough usage quota for development and demos.
- The Supabase project will remain active and accessible throughout the project.
- Travel URLs submitted by users are publicly accessible (no login needed to fetch them).
- OpenAI GPT-4o will return valid JSON when given a strict JSON schema in the prompt.
- This app is for a class project and demo purposes, not for production-scale public use.

---

## 3. External Interface Requirements

### 3.1 User Interfaces

The application has the following pages, all built as React components:

- **Landing Page (`/`):** Introduces the app. Includes a "View sample itinerary" button that opens a popup showing a real 3-day Paris example so users understand what the app produces.
- **Trip Seed Page (`/create/seed`):** A form where users enter a travel link and a required trip summary. An optional collapsible section lets users add location, check-in/check-out dates, and accommodation details. The form validates dates and shows errors inline.
- **Teaser Page (`/trips/:tripId/teaser`):** Shows the 3-day teaser itinerary as a grid of day cards. Displays daily themes, highlights, a cost range, vibe tags, and best time to visit. Has a "Regenerate" button and a "Plan full trip" button.
- **Login Page (`/login`):** A form for existing users to log in with their username and password. Includes a link to registration and redirects back to the original destination after login.
- **Register Page (`/register`):** A form for new users to create an account with a username and password.
- **Preferences Page (`/trips/:tripId/preferences`):** A protected, 4-step wizard for entering travel preferences. Steps cover interests, budget, pace, and dietary/accessibility needs. Only accessible to logged-in users.
- **Generating Page (`/trips/:tripId/generating`):** A dedicated loading screen shown while the full itinerary is being built. Cycles through messages like "Crafting day-by-day activities..." to keep the user informed.
- **Itinerary Page (`/trips/:tripId/itinerary`):** Shows the full day-by-day itinerary in a timeline layout. Each activity includes a time, duration, location, description, cost, optional booking link, and tips. A sidebar shows the total cost breakdown by category.

All pages share a consistent dark-themed design. A header shows the app name and login/logout controls. A footer appears on every page.

### 3.2 Hardware Interfaces

Link2Itinerary is a standard web application. It does not connect to any hardware devices, sensors, or peripherals. Users only need a device with a modern web browser and internet access.

### 3.3 Software Interfaces

| Component | Version | How It Is Used |
|-----------|---------|----------------|
| OpenAI API | GPT-4o | Called from the backend using the `openai` npm SDK. The backend sends a structured prompt and a JSON schema. OpenAI returns a validated JSON itinerary object. |
| Supabase (PostgreSQL) | 14+ | Cloud-hosted database. Connected via a `DATABASE_URL` environment variable using TypeORM. Stores users, trip seeds, and cached itineraries. |
| TypeORM | 0.3.x | ORM layer. Maps TypeScript entity classes to database tables. Uses `autoLoadEntities` and `synchronize` in development. |
| Cheerio | 1.x | HTML parser. Used on the backend to extract readable text from URLs submitted by users, which is then sent to OpenAI as context. |
| Passport / JWT | Latest | Handles authentication. The backend issues a signed JWT token on login. Protected routes check the token using a `JwtAuthGuard`. |
| bcrypt | Latest | Password hashing library. All passwords are hashed with bcrypt before being stored. Plaintext passwords are never saved. |

### 3.4 Communications Interfaces

- **Frontend to Backend:** HTTP/HTTPS REST API. The frontend calls the backend at a configurable base URL (default: `http://localhost:3000/api`). All requests and responses use JSON.
- **Backend to OpenAI:** HTTPS requests to `api.openai.com` using the OpenAI Node.js SDK with the `chat.completions.create` method and a JSON schema response format.
- **Backend to Supabase:** TCP connection using the `pg` PostgreSQL driver through TypeORM. Connection details come from the `DATABASE_URL` environment variable.
- **Backend to user-submitted URLs:** HTTP/HTTPS fetch requests to scrape travel content. Uses Node.js native fetch with a custom User-Agent header.
- **CORS:** The backend uses `app.enableCors()` to allow requests from the frontend dev server (`localhost:5173`) and other allowed origins.

---

## 4. System Features

### 4.1 Trip Seed Creation

#### 4.1.1 Description and Priority

This feature lets users start a trip by submitting a travel link and a short summary. It is the entry point for the whole app. **Priority: HIGH.**

#### 4.1.2 Response Sequences

1. User navigates to `/create/seed`.
2. User optionally enters a travel URL (Hotel, Airbnb, etc.) and optionally opens the details section to enter location, dates, and accommodation info.
3. User enters a required trip summary (e.g., "4-day beach trip to Tulum in May, relaxed pace").
4. User clicks "Generate teaser." The system validates the form and shows an error if the summary is missing or if check-out is before check-in.
5. The system sends a `POST /api/trips/seed` request, creates a `TripSeed` record in the database, and redirects to the teaser page.

#### 4.1.3 Functional Requirements

| Req ID | Description | Priority |
|--------|-------------|----------|
| REQ-SEED1 | The system will provide an optional URL input field on the trip seed form. | High |
| REQ-SEED2 | The system will provide a required summary textarea. Submission will be blocked if the summary is empty. | High |
| REQ-SEED3 | The system will provide a collapsible "optional details" section with fields for location, check-in date, check-out date, accommodation name, and accommodation type. | Medium |
| REQ-SEED4 | The system will validate that check-out is after check-in if both dates are provided. | Medium |
| REQ-SEED5 | The system will create a `TripSeed` record in the database via `POST /api/trips/seed` and redirect the user to the teaser page on success. | High |
| REQ-SEED6 | All TripSeed fields except status will be nullable so the form works with minimal input. | High |

---

### 4.2 Teaser Itinerary Generation

#### 4.2.1 Description and Priority

This feature creates a lightweight 3-day trip preview without requiring the user to log in. It uses OpenAI GPT-4o to generate day themes, highlights, a cost estimate, and vibe tags. Results are cached in the `teaser_cache` table so repeat visits do not re-call OpenAI. **Priority: HIGH.**

#### 4.2.2 Response Sequences

1. User is redirected to `/trips/:tripId/teaser` after trip creation.
2. The frontend calls `POST /api/planner/teaser`. If a cached teaser exists in the database, it is returned immediately. If not, the backend fetches the submitted URL, extracts text, and calls OpenAI.
3. The teaser page renders three day cards, a cost range badge, vibe tags, and a "best time to go" note.
4. User can click "Regenerate teaser" to generate a new version, or "Plan full trip" to continue.

#### 4.2.3 Functional Requirements

| Req ID | Description | Priority |
|--------|-------------|----------|
| REQ-TEAS1 | The system will automatically call `POST /api/planner/teaser` when the teaser page loads. | High |
| REQ-TEAS2 | The teaser will include exactly 3 days, each with a theme string and an array of 3 to 5 highlights. | High |
| REQ-TEAS3 | The teaser will include an estimated cost range with min, max, and USD currency. | High |
| REQ-TEAS4 | The system will cache the teaser in the `teaser_cache` table and return the cached version on repeat requests. | High |
| REQ-TEAS5 | The system will display a loading indicator while the teaser is being generated. | High |
| REQ-TEAS6 | The system will show a user-friendly error with a retry button if teaser generation fails. | High |
| REQ-TEAS7 | The backend will return a deterministic fallback response if the OpenAI API call fails, so the page does not crash. | Medium |
| REQ-TEAS8 | The teaser endpoint (`POST /api/planner/teaser`) will work for both guest users and logged-in users. | Medium |

---

### 4.3 User Registration and Login

#### 4.3.1 Description and Priority

This feature allows users to enter a client-side login flow that gates access to protected planning pages in the frontend. Authentication state is managed through React context and persisted in localStorage. **Priority: HIGH.**

#### 4.3.2 Response Sequences

1. User navigates to the login flow, enters identifying information, and submits. The frontend updates authentication state and allows access to protected planning pages.
2. Returning users can re-enter the frontend login flow to regain access to protected planning pages.
3. The frontend stores user information in localStorage. The app header shows login/logout controls based on authentication state.

#### 4.3.3 Functional Requirements

| Req ID | Description | Priority |
|--------|-------------|----------|
| REQ-AUTH-1 | The system will provide a frontend login flow that updates authentication state and gates access to protected planning pages. | High |
| REQ-AUTH-2 | The system will persist user authentication state in localStorage across page refreshes. | High |
| REQ-AUTH-3 | The preferences page and full itinerary flow will be protected in the frontend. Unauthenticated users will be redirected to the login page. | High |
| REQ-AUTH-4 | After login, the system will redirect the user to the page they were originally trying to access. | Medium |

---

### 4.4 Preferences Collection

#### 4.4.1 Description and Priority

This feature is a 4-step wizard that lets logged-in users customize their trip before the full itinerary is generated. **Priority: HIGH.**

#### 4.4.2 Response Sequences

1. User clicks "Plan full trip" on the teaser page. If not logged in, they are redirected to login first.
2. The 4-step wizard walks the user through: (1) interests, (2) budget level, (3) pace, (4) dietary and accessibility needs.
3. On final submission, preferences are saved to the trip record via `PATCH /api/trips/:id`, then the user is sent to the generating page.

#### 4.4.3 Functional Requirements

| Req ID | Description | Priority |
|--------|-------------|----------|
| REQ-PREF1 | The preferences page will be a protected route. Unauthenticated users will be redirected to login. | High |
| REQ-PREF2 | The wizard will support: interests (multi-select), budget (budget/moderate/luxury), pace (relaxed/moderate/packed), dietary restrictions, and accessibility needs. | High |
| REQ-PREF3 | Step 3 will not auto-submit the form. The "Next" button will advance to step 4 and only the final "Generate itinerary" button will submit. | High |
| REQ-PREF4 | The system will save preferences to the trip record via `PATCH /api/trips/:id` before calling the itinerary endpoint. | High |

---

### 4.5 Full Itinerary Generation

#### 4.5.1 Description and Priority

This is the core feature of the app. It generates a complete, personalized day-by-day travel itinerary using the trip data and user preferences. Results are cached so the same preferences do not trigger a second OpenAI call. **Priority: HIGH.**

#### 4.5.2 Response Sequences

1. After preferences are submitted, the user is sent to the generating page (`/trips/:tripId/generating`).
2. The generating page calls `POST /api/planner/full`. The backend checks the `itinerary_cache` table using a hash of the preferences. If a match exists, it returns the cached result immediately.
3. If no cache hit, the backend fetches the trip from the database, scrapes the URL if available, builds a detailed prompt, and calls OpenAI.
4. OpenAI returns a structured JSON itinerary. The result is saved to `itinerary_cache` and the trip status is updated to `"itinerary_generated"`.
5. The frontend navigates to the itinerary page and displays the results.

#### 4.5.3 Functional Requirements

| Req ID | Description | Priority |
|--------|-------------|----------|
| REQ-ITIN-1 | The full itinerary will cover all days from check-in to check-out (inclusive). If no dates are provided, it will default to a 3-day itinerary starting today. | High |
| REQ-ITIN-2 | Each day will contain activities. The number of activities per day will reflect the user's pace preference: relaxed = 2, moderate = 3, packed = 5. | High |
| REQ-ITIN-3 | Each activity will include: a unique ID, title, time (HH:MM), duration in minutes, description, location name, estimated cost, and optional booking URL and tips. | High |
| REQ-ITIN-4 | The itinerary will include a total estimated cost with min, max, currency, and a breakdown by category. | High |
| REQ-ITIN-5 | The backend will cache the itinerary in the `itinerary_cache` table using a SHA-256 hash of the preferences as the cache key. | High |
| REQ-ITIN-6 | The backend will update the trip status to `"itinerary_generated"` after a successful generation. | Medium |
| REQ-ITIN-7 | The generating page will cycle through status messages while the API call runs, so users know the app is working. | Medium |
| REQ-ITIN-8 | The backend will return a fallback itinerary if the OpenAI API call fails. | Medium |

---

### 4.6 Cost Estimation

#### 4.6.1 Description and Priority

A dedicated Estimator module calculates a detailed cost breakdown for a generated itinerary. It uses real activity costs from the cached itinerary, trip dates, accommodation type, and budget level to produce accurate estimates. **Priority: MEDIUM.**

#### 4.6.2 Response Sequences

1. After the itinerary page loads, the frontend calls `POST /api/estimator/calculate` with the itinerary ID.
2. The backend gets the cached itinerary and trip seed from the database to get real cost data.
3. The estimator sums actual activity costs from the itinerary, calculates accommodation based on nightly rates and accommodation type, adds per-day dining and transportation, plus a shopping estimate and a 5% miscellaneous buffer.
4. The cost breakdown is displayed in the itinerary sidebar.

#### 4.6.3 Functional Requirements

| Req ID | Description | Priority |
|--------|-------------|----------|
| REQ-COST1 | The system will provide `POST /api/estimator/calculate` that accepts an `itineraryId` and returns a full cost breakdown. | High |
| REQ-COST2 | The breakdown will include six categories: accommodation, dining, activities, transportation, shopping, and miscellaneous. | High |
| REQ-COST3 | Activity costs will be summed from the actual cached itinerary data. The range will be +/-10% of the actual total. | High |
| REQ-COST4 | Accommodation costs will be calculated using per-night rates adjusted for budget level (budget/moderate/luxury) and accommodation type (hostel, Airbnb, hotel, resort, etc.). | Medium |
| REQ-COST5 | The response will include a total cost range (min/max), an average, a per-day average, and a `calculatedAt` timestamp. | Medium |

---

## 5. Other Nonfunctional Requirements

### 5.1 Performance Requirements

- The teaser generation endpoint (`POST /api/planner/teaser`) should respond within 15 seconds under normal conditions.
- The full itinerary endpoint (`POST /api/planner/full`) should respond within 30 seconds under normal conditions.
- The trip seed creation endpoint (`POST /api/trips/seed`) should respond within 3 seconds.
- If a cached teaser or itinerary exists, the response should be returned within 2 seconds.
- Any operation expected to take more than 500 milliseconds will display a loading state to the user.

### 5.2 Safety Requirements

This application does not involve physical safety risks. The following data safety measures apply:

- API keys, JWT secrets, and database credentials will never be committed to version control. They must be stored in `.env` files that are listed in `.gitignore`.
- Raw HTML scraped from user-submitted URLs will not be stored in the database.

### 5.3 Security Requirements

- Client-side authentication state stored in localStorage will not contain secrets, API keys, or sensitive backend credentials.
- Protected planning flows in the frontend will require authentication state before granting access to preferences and full itinerary features.
- All API request bodies will be validated using class-validator DTOs. Invalid or missing fields will return a 400 Bad Request response.
- TypeORM parameterized queries will be used for all database interactions to prevent SQL injection.

### 5.4 Software Quality Attributes

**Maintainability:**
- The backend follows NestJS module boundaries — Trips, Planner, Estimator, Auth, Users, Itineraries, and cache modules each have a clear single responsibility.
- Controllers are kept thin. Business logic lives in services.

**Testability:**
- All services use NestJS dependency injection, making them easy to unit test with mocked dependencies.
- The frontend has a complete mock layer (`services/mocks.ts`) controlled by the `VITE_USE_MOCKS` environment variable. This allows full UI testing without a running backend.

**Usability:**
- All error states will show a user-friendly message and a retry option.
- All asynchronous operations will show a loading indicator.
- The landing page sample itinerary popup helps new users understand what the app does before they start.

**Reliability:**
- The backend will cache teaser and itinerary results to avoid repeat OpenAI calls and protect against API downtime.
- All LLM call methods will have fallback responses so the frontend does not crash if OpenAI is unavailable.

### 5.5 Business Rules

- A trip seed can be created without an account. The teaser is public.
- Only logged-in users can access the preferences page and generate a full itinerary.
- If a user was logged in when creating the trip seed, the seed is linked to their account via the `userId` column.
- Itinerary IDs and trip IDs are always generated on the server side, never on the client.
- The full itinerary endpoint (`POST /api/planner/full`) requires the user to be authenticated in the frontend.

---

## 6. Other Requirements

### 6.1 Database Requirements

The system uses a single PostgreSQL database (Supabase) with the following tables:

**users**
- `id`: UUID, primary key
- `username`: VARCHAR(30), unique — used as the login identifier
- `passwordHash`: TEXT — bcrypt hash of the password
- `created_at`, `updated_at`: timestamps

**trip_seeds**
- `id`: UUID, primary key
- `url`: TEXT, nullable — the user's submitted travel link
- `summary`: TEXT, nullable — short trip description
- `location`: VARCHAR(255), nullable
- `check_in`, `check_out`: DATE, nullable
- `accommodation_name`: VARCHAR(255), nullable
- `accommodation_type`: VARCHAR(50), nullable (e.g., "airbnb", "hotel", "resort")
- `status`: VARCHAR(50), default `"seed_created"` (also: `"teaser_generated"`, `"itinerary_generated"`)
- `user_id`: UUID, nullable — links the trip to a registered user
- `created_at`, `updated_at`: timestamps

**teaser_cache**
- `id`: UUID, primary key
- `trip_seed_id`: UUID — foreign key to `trip_seeds` (one-to-one)
- `payload`: JSONB — the full `PlannerTeaserResponse` object
- `generated_at`: timestamp

**itinerary_cache**
- `id`: UUID, primary key (same as the itinerary's logical ID)
- `trip_seed_id`: UUID — foreign key to `trip_seeds`
- `preferences_hash`: VARCHAR(64) — SHA-256 of the serialized preferences
- `preferences`: JSONB — the preferences used to generate this itinerary
- `payload`: JSONB — the full `PlannerFullItineraryResponse` object
- `generated_at`: timestamp

For the complete schema including all 9 tables, see [db/schema-plan.md](../db/schema-plan.md).

### 6.2 Internationalization

Link2Itinerary supports English only. All UI text, error messages, and AI prompts are in English. All cost estimates are in USD. Support for other languages or currencies is out of scope for this release.

---

## Appendix A: Glossary

| Term | Definition |
|------|------------|
| Trip Seed | The starting record created when a user submits a travel link and summary. It is the foundation for all itinerary generation. |
| Teaser Itinerary | A quick 3-day trip preview with daily themes and highlights. Generated without needing a user account. |
| Full Itinerary | A detailed day-by-day plan with timed activities, costs, and booking links. Requires login and preferences. |
| Teaser Cache | A database table that stores generated teasers so the same trip does not need to call OpenAI again on repeat visits. |
| Itinerary Cache | A database table that stores full itineraries, keyed by trip ID and a hash of the user's preferences. |
| LLM | Large Language Model. In this system, refers specifically to OpenAI GPT-4o. |
| DTO | Data Transfer Object. A TypeScript class with validation rules used to check incoming API request bodies in NestJS. |
| JWT | JSON Web Token. A signed token issued by the backend on login. The frontend sends it with protected API requests to prove identity. |
| Entity | A TypeORM TypeScript class that maps to a database table (e.g., `TripSeed`, `User`, `ItineraryCache`). |
| Vibe Tags | Short labels that describe the mood of a trip (e.g., "relaxed," "foodie," "cultural"). Generated by the LLM in the teaser response. |
| CORS | Cross-Origin Resource Sharing. A browser security feature. The backend enables it so the frontend on a different port can make API requests. |
| bcrypt | A password hashing algorithm. Used to safely store user passwords. The hash cannot be reversed to get the original password. |
| Preferences Hash | A SHA-256 fingerprint of the user's preferences object. Used as the cache key for itineraries so the same preferences always return the same cached result. |

---

## Appendix B: Analysis Models

### System Architecture

The system uses a three-tier architecture:

- **Presentation Tier:** React + TypeScript SPA (Vite). Pages: Landing, TripSeed, Teaser, Login, Register, Preferences, Generating, Itinerary. State is shared across pages using React Context (`TripContext`, `AuthContext`).
- **Application Tier:** NestJS REST API. Eight feature modules — Trips, Planner, Estimator, Auth, Users, Itineraries, TeaserCache, ItineraryCache. All validation uses class-validator DTOs. All dependencies are injected through NestJS.
- **Data Tier:** PostgreSQL on Supabase. Four core tables — `users`, `trip_seeds`, `teaser_cache`, `itinerary_cache`. ORM: TypeORM.

### Full User Flow

1. **Landing page:** User views the app and optionally sees a sample itinerary popup.
2. **Trip seed:** User submits a link and summary → `POST /api/trips/seed` → TripSeed saved to DB.
3. **Teaser:** System calls `POST /api/planner/teaser` → OpenAI generates a 3-day preview → cached in `teaser_cache` → displayed.
4. **Login/Register:** User logs in or creates an account → authentication state stored in localStorage.
5. **Preferences:** User fills in a 4-step wizard → `PATCH /api/trips/:id` saves preferences.
6. **Generating page:** Full itinerary generation begins → `POST /api/planner/full` → status messages shown while waiting.
7. **Full itinerary:** OpenAI returns structured JSON → cached in `itinerary_cache` → displayed with activity timeline.
8. **Cost estimate:** `POST /api/estimator/calculate` → breakdown by category shown in sidebar.

---

## Appendix C: To Be Determined List

| # | Item | Notes |
|---|------|-------|
| 1 | Video content processing | The system currently only scrapes HTML from user-submitted URLs. Full support for extracting spoken content from travel videos (e.g., YouTube transcripts) has not been implemented. |
| 2 | Automated test coverage | Unit and integration tests exist only at a basic level. A more comprehensive test suite covering individual services, API endpoints, and edge cases is planned. |
| 3 | Internationalization | The application currently supports English and USD only. Support for additional languages and currencies has not been implemented. |
| 4 | CI/CD pipeline | A continuous integration and deployment environment for automated testing and deployment on code changes has not been configured. |
