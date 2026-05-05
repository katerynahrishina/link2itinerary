# Architecture Overview
## Link2Itinerary

**Version:** 1.0 (Draft)
**Date:** [To be filled]
**Authors:** Link2Itinerary Team

---

## 1. System Overview

Link2Itinerary is a full-stack web application that uses hosted Large Language Models (LLMs) and a deterministic consistency engine to generate personalized travel itineraries from user-provided links.

### High-Level Architecture

```
┌─────────────┐
│   Browser   │  (React Frontend)
└──────┬──────┘
       │ HTTPS/JSON
       ▼
┌─────────────┐
│  NestJS API │  (Backend Server)
└──────┬──────┘
       │
       ├─────────────┐
       │             │
       ▼             ▼
┌──────────┐   ┌──────────────┐
│PostgreSQL│   │ LLM Provider │
│ Database │   │ (OpenAI)     │
└──────────┘   └──────────────┘
```

---

## 2. Architectural Patterns

### 2.1 Overall Pattern: Three-Tier Architecture

1. **Presentation Layer:** React frontend (client-side)
2. **Application Layer:** NestJS backend (business logic, API)
3. **Data Layer:** PostgreSQL database + external LLM APIs

### 2.2 Backend Pattern: Modular Monolith

The NestJS backend is organized into domain-driven modules:
- **Trips Module:** Trip seed CRUD
- **Planner Module:** Itinerary generation orchestration
- **Estimator Module:** Cost calculation

Benefits:
- Clear separation of concerns
- Easy to test individual modules
- Future scalability (modules can become microservices if needed)

### 2.3 Frontend Pattern: Component-Based Architecture

React components organized by feature:
- **Pages:** Top-level route components
- **Feature Components:** Domain-specific (trip-seed, teaser, itinerary)
- **Common Components:** Reusable UI elements

State management via Context API for shared state.

---

## 3. Agentic Workflow Design

The core innovation of Link2Itinerary is the **agentic workflow** that orchestrates LLM calls and validation.

### 3.1 Workflow Steps

```
1. Link Scraping Agent
   ↓
2. Context Builder Agent
   ↓
3. Teaser Planner Agent
   ↓
4. Full Planner Agent
   ↓
5. Consistency Engine (Deterministic)
   ↓
6. Cost Estimator Agent
```

### 3.2 Agent Descriptions

#### Agent 1: Link Scraping Agent
**Purpose:** Extract metadata from user-provided link

**Input:**
- URL (Airbnb, hotel, attraction)

**Output:**
- Location (city, country)
- Check-in and check-out dates
- Accommodation name and type
- Additional metadata (images, reviews, etc.)

**Implementation:**
- LLM-based extraction (send URL + HTML snippet)
- Fallback: Manual parsing or external scraping API

**LLM Prompt Example:**
```
Extract trip information from this link:
URL: https://airbnb.com/rooms/12345
HTML Snippet: [first 500 chars]

Return JSON:
{
  "location": "Paris, France",
  "checkIn": "2026-05-01",
  "checkOut": "2026-05-05",
  "accommodation": {
    "name": "Charming apartment in Le Marais",
    "type": "airbnb"
  }
}
```

---

#### Agent 2: Context Builder Agent
**Purpose:** Enrich trip with location-specific data

**Input:**
- Location (from Agent 1)
- Dates (from Agent 1)

**Output:**
- List of top attractions
- Popular restaurants
- Local events during visit
- Weather forecast (if available)
- Transportation options

**Implementation:**
- LLM knowledge base (for general info)
- Optional: External APIs (Google Places, weather APIs)

**LLM Prompt Example:**
```
Provide context for a trip to Paris, France from May 1-5, 2026.

Return JSON:
{
  "topAttractions": ["Eiffel Tower", "Louvre Museum", ...],
  "restaurants": ["Le Comptoir du Relais", ...],
  "localEvents": ["Spring festival in Latin Quarter"],
  "weather": "Mild, 15-20°C, occasional rain",
  "transportation": "Metro, buses, bikes"
}
```

---

#### Agent 3: Teaser Planner Agent
**Purpose:** Generate quick 3-day overview to engage user

**Input:**
- Trip metadata (location, dates, accommodation)
- Context data (from Agent 2)

**Output:**
- 3 daily themes
- 3-5 highlights per day
- Rough cost estimate

**Implementation:**
- Single LLM call with structured JSON output
- Fast (< 10 seconds)

**LLM Prompt Example:**
```
Create a 3-day teaser itinerary for Paris, May 1-5, 2026.

Trip details:
- Staying at "Charming apartment in Le Marais"
- Interests: Not yet specified (keep general)

Return JSON:
{
  "days": [
    {
      "date": "2026-05-01",
      "theme": "Arrival & Local Exploration",
      "highlights": ["Check-in", "Seine River walk", "Dinner at bistro"]
    },
    ...
  ],
  "estimatedCost": { "min": 400, "max": 600 }
}
```

---

#### Agent 4: Full Planner Agent
**Purpose:** Generate detailed day-by-day itinerary based on preferences

**Input:**
- Trip metadata
- Context data
- User preferences (interests, budget, pace, dietary)

**Output:**
- Complete itinerary with activities for each day
- Each activity: time, duration, title, description, location, cost

**Implementation:**
- One or more LLM calls (may batch days or iterate)
- Structured JSON schema enforcement

**LLM Prompt Example:**
```
Create a detailed 5-day itinerary for Paris, May 1-5, 2026.

Trip details:
- Accommodation: Le Marais apartment (123 Rue de Rivoli)
- User preferences:
  - Interests: Museums, Food, Architecture
  - Budget: Moderate
  - Pace: Relaxed (2-3 activities/day)
  - Dietary: None

Return JSON (see backend/api-contracts.md for full schema):
{
  "days": [
    {
      "date": "2026-05-01",
      "activities": [
        {
          "time": "14:00",
          "duration": 60,
          "title": "Check-in",
          "description": "...",
          "location": { ... },
          "estimatedCost": 0
        },
        ...
      ]
    },
    ...
  ]
}
```

---

#### Component 5: Consistency Engine (Deterministic)
**Purpose:** Validate and fix logical issues in LLM-generated itinerary

**Input:**
- Raw itinerary from Agent 4

**Output:**
- Validated, corrected itinerary

**Validation Rules:**
1. **No Time Overlaps:** Activities must not overlap in time
2. **Realistic Timing:** Travel time between locations accounted for
3. **Operating Hours:** Attractions open during scheduled time
4. **Geographic Clustering:** Minimize backtracking across city
5. **Daily Pacing:** Respect user's pace preference (relaxed = fewer activities)

**Implementation:**
- Deterministic TypeScript/Python code (not LLM-based)
- Rule-based system
- If issues found, either:
  - Auto-fix (adjust times, reorder activities)
  - Flag for LLM re-generation

**Example Validation:**
```typescript
function validateNoOverlaps(activities: Activity[]): boolean {
  for (let i = 0; i < activities.length - 1; i++) {
    const current = activities[i];
    const next = activities[i + 1];
    const currentEnd = addMinutes(current.time, current.duration);
    if (currentEnd > next.time) {
      // Overlap detected, adjust next.time or flag error
      return false;
    }
  }
  return true;
}
```

---

#### Component 6: Cost Estimator
**Purpose:** Calculate cost estimates for itinerary

**Input:**
- Itinerary with activities

**Output:**
- Total cost (min/max)
- Breakdown by category (dining, activities, transport, misc)

**Implementation:**
- LLM-based for rough estimates (using knowledge base)
- Optional: External pricing APIs (OpenTable, GetYourGuide, etc.)

**LLM Prompt Example:**
```
Estimate costs for this itinerary in Paris:

Activities:
- Dinner at Le Comptoir du Relais (French bistro)
- Louvre Museum entry
- Seine River cruise

Return JSON:
{
  "breakdown": {
    "dining": { "min": 180, "max": 240 },
    "activities": { "min": 150, "max": 180 },
    "transport": { "min": 50, "max": 70 }
  },
  "total": { "min": 480, "max": 620 }
}
```

---

## 4. Consistency Engine Details

### 4.1 Why Deterministic?

LLMs can generate creative itineraries but may produce logically inconsistent plans (e.g., visiting a museum after closing time, scheduling overlapping activities). The consistency engine adds a **deterministic layer** to catch and correct these issues.

### 4.2 Key Rules

| Rule | Description | Fix Strategy |
|------|-------------|--------------|
| No overlaps | Activities must not overlap in time | Adjust start times or remove activity |
| Travel time | Account for transit between locations | Add buffer time or reorder activities |
| Operating hours | Attractions must be open | Shift time or replace activity |
| Pacing | Match user's pace preference | Remove activities if too packed |
| Geography | Minimize distance between consecutive activities | Reorder for geographic clustering |

### 4.3 Algorithm Outline

```typescript
function validateItinerary(itinerary: Itinerary): Itinerary {
  for (const day of itinerary.days) {
    // Step 1: Check for time overlaps
    if (!validateNoOverlaps(day.activities)) {
      day.activities = fixOverlaps(day.activities);
    }

    // Step 2: Validate operating hours
    for (const activity of day.activities) {
      if (!isOpen(activity.location, activity.time)) {
        activity.time = suggestAlternativeTime(activity);
      }
    }

    // Step 3: Optimize geographic order
    day.activities = reorderByProximity(day.activities);

    // Step 4: Check pacing
    if (day.activities.length > user.paceLimit) {
      day.activities = prioritizeActivities(day.activities, user.interests);
    }
  }

  return itinerary;
}
```

---

## 5. Data Model

### 5.1 Core Entities

```
┌─────────────┐
│  TripSeed   │
└──────┬──────┘
       │ 1:N
       ▼
┌─────────────┐
│  Itinerary  │
└──────┬──────┘
       │ 1:N
       ▼
┌─────────────┐
│ItineraryDay │
└──────┬──────┘
       │ 1:N
       ▼
┌─────────────┐
│  Activity   │
└─────────────┘
```

### 5.2 Entity Descriptions

See [db/schema-plan.md](../db/schema-plan.md) for detailed schema.

**TripSeed:**
- Stores user-provided URL, summary, and extracted metadata
- Links to one itinerary

**Itinerary:**
- Represents a generated plan (teaser or full)
- Includes preferences and status

**ItineraryDay:**
- One day within an itinerary
- Contains ordered activities

**Activity:**
- Individual activity (e.g., "Visit Louvre")
- Includes time, duration, location, cost, etc.

---

## 6. Technology Stack

### Frontend
- **Framework:** React 18+
- **Language:** TypeScript
- **Routing:** React Router
- **State:** Context API
- **Styling:** Tailwind CSS or Material-UI
- **Build:** Vite

### Backend
- **Framework:** NestJS
- **Language:** TypeScript
- **ORM:** TypeORM or Prisma
- **Database:** PostgreSQL
- **LLM Clients:** OpenAI SDK

### Infrastructure
- **Hosting:** Vercel (frontend), Railway/AWS (backend)
- **CI/CD:** GitHub Actions
- **Monitoring:** Not configured for MVP

---

## 7. Security Considerations

1. **API Keys:** Stored in environment variables, never in code
2. **Input Validation:** All user inputs sanitized (prevent XSS, SQL injection)
3. **Rate Limiting:** Protect against abuse (limit LLM calls per user/IP)
4. **HTTPS Only:** All communication encrypted

---

## 8. Scalability Considerations

**Current Scope (MVP):**
- Single server deployment
- ~50 concurrent users
- Single PostgreSQL instance

**Future Scalability:**
- **Horizontal Scaling:** Add more backend instances behind load balancer
- **Database Replication:** Read replicas for high traffic
- **Caching:** Redis for frequently accessed itineraries
- **Async Processing:** Background jobs for slow LLM calls (use queues like Bull)
- **Microservices:** Break modules into separate services if needed

---

## 9. API Design Principles

- **RESTful:** Standard HTTP methods (GET, POST, PATCH, DELETE)
- **Resource-based URLs:** `/api/trips/:id`, `/api/planner/full`
- **JSON Responses:** Consistent structure with status codes
- **Error Handling:** Meaningful error messages with codes
- **Versioning:** (Future) `/api/v1/...` for breaking changes

See [backend/api-contracts.md](../backend/api-contracts.md) for full API spec.

---

## 10. Deployment Architecture (Planned)

```
┌──────────────────┐
│   Vercel CDN     │  (Frontend - React build)
└────────┬─────────┘
         │ HTTPS
         ▼
┌──────────────────┐
│  Railway/AWS     │  (Backend - NestJS API)
└────────┬─────────┘
         │
         ├─────────────┐
         │             │
         ▼             ▼
┌─────────────┐  ┌─────────────┐
│ PostgreSQL  │  │ OpenAI API  │
│ (Railway)   │  │ (External)  │
└─────────────┘  └─────────────┘
```

**Steps:**
1. Frontend deployed to Vercel (auto-deploy on git push)
2. Backend deployed to Railway or AWS Elastic Beanstalk
3. Database hosted on Railway PostgreSQL or AWS RDS
4. Environment variables configured in deployment platform

---

## 11. Monitoring & Observability

**Planned Metrics:**
- Request latency (API response times)
- LLM call success/failure rates
- Database query performance
- User flow completion rates (seed → teaser → full itinerary)

**Tools (Future):**
- Logging: Winston (NestJS), CloudWatch (AWS)
- Error Tracking: Sentry
- Analytics: Google Analytics or Mixpanel

---

## 12. Development Workflow

1. **Local Development:**
   - Frontend: `npm run dev` (Vite dev server)
   - Backend: `npm run start:dev` (NestJS watch mode)
   - Database: Local PostgreSQL or Docker container

2. **Version Control:**
   - Git branches: `main`, `develop`, `feature/*`
   - Pull requests for code review
   - CI runs tests before merge

3. **Testing:**
   - Unit tests during development
   - Integration tests before merging to `develop`
   - E2E tests before deploying to `main`

4. **Deployment:**
   - Merge to `main` triggers auto-deployment
   - Staging environment for pre-production testing (optional)

---

## 13. Future Enhancements (Post-MVP)

1. **User Authentication:** Save trips, view history
2. **Itinerary Sharing:** Shareable public links for itineraries
3. **Calendar Export:** ICS file generation for calendar apps
4. **Itinerary Regeneration:** Update preferences and regenerate with version history
5. **Multiple Cost Tiers:** Budget/moderate/luxury cost estimation options
6. **Mobile App:** React Native or Flutter version
7. **Multi-language Support:** Itineraries in different languages
8. **Booking Integration:** Direct booking via affiliate links

---

## Appendix A: Sequence Diagrams

### User Creates Full Itinerary

```
User → Frontend: Paste Airbnb link
Frontend → Backend: POST /api/trips/seed
Backend → LLM: Extract metadata
LLM → Backend: Return location, dates, etc.
Backend → Database: Save trip seed
Backend → Frontend: Return trip seed
Frontend → User: Show metadata preview

User → Frontend: Click "Generate Teaser"
Frontend → Backend: POST /api/planner/teaser
Backend → LLM: Generate teaser
LLM → Backend: Return teaser itinerary
Backend → Frontend: Return teaser
Frontend → User: Display teaser

User → Frontend: Customize preferences
Frontend → Backend: PATCH /api/trips/:id
Backend → Database: Update preferences
Backend → Frontend: Confirm update

User → Frontend: Click "Generate Itinerary"
Frontend → Backend: POST /api/planner/full
Backend → LLM: Generate full itinerary
LLM → Backend: Return raw itinerary
Backend → ConsistencyEngine: Validate
ConsistencyEngine → Backend: Validated itinerary
Backend → Database: Save itinerary
Backend → Frontend: Return full itinerary
Frontend → User: Display itinerary
```

---

## Appendix B: Diagram Conventions

- **Boxes:** System components
- **Arrows:** Data flow
- **→:** Synchronous call
- **⇢:** Asynchronous call

(Future: Add actual diagrams using draw.io, Mermaid, or similar)
