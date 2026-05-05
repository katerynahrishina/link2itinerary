# Link2Itinerary — Project Summary

**Course:** CEN4090L — Software Engineering II, FSU 2026
**Status:** Complete — MVP delivered and demonstrated

---

## What Was Built

Link2Itinerary is a full-stack web application that turns a travel destination link (Airbnb, hotel, attraction) into a personalized, AI-generated itinerary with real-time cost estimates. Users can register, log in, generate teasers and full itineraries, and save them to a personal library.

### Backend (NestJS)

Five feature modules were implemented and fully connected to a live Supabase PostgreSQL database:

| Module | What it does |
|--------|-------------|
| **Auth** | Username/password registration and login using bcrypt + JWT. Tokens expire after 30 days. |
| **Trips** | Full CRUD for trip seeds — create from a travel link, read, update, delete. |
| **Planner** | Generates teaser (3-day) and full day-by-day itineraries via OpenAI GPT-4o. Supports anonymous and authenticated users. |
| **Estimator** | Calculates cost breakdowns (dining, activities, transport, shopping, misc) from a saved itinerary. |
| **Itineraries** | Lists, retrieves, and saves itineraries for the logged-in user. |

Two caching modules prevent redundant OpenAI calls:
- **Teaser cache** — one cached teaser per trip seed (upsert on regeneration)
- **Itinerary cache** — one row per trip + preferences hash combination

### Frontend (React + Vite)

Nine pages were implemented and fully wired to the live backend API:

| Page | Route | Description |
|------|-------|-------------|
| Landing | `/` | Hero section, CTA, project overview |
| Login | `/login` | Register or log in with username/password |
| Trip Seed | `/create/seed` | Paste travel link, enter trip details |
| Teaser | `/trips/:id/teaser` | 3-day quick overview with cost estimate |
| Preferences | `/trips/:id/preferences` | Interests, budget, pace, dietary/accessibility |
| Generating | `/trips/:id/generating` | Loading state while OpenAI generates the itinerary |
| Itinerary | `/trips/:id/itinerary` | Full day-by-day itinerary with cost breakdown |
| My Itineraries | `/my-itineraries` | List of saved itineraries (requires login) |
| Saved Itinerary | `/my-itineraries/:id` | Read-only view of a saved itinerary |

### Database (PostgreSQL via Supabase)

9 tables implemented:

- `users` — registered accounts
- `trip_seeds` — trip information from user-provided links
- `preferences` — user preferences per trip seed
- `teaser_cache` — cached 3-day teaser responses
- `itinerary_cache` — cached full itinerary responses (keyed by trip + preferences hash)
- `itineraries` — normalized itinerary records
- `itinerary_days` — individual days within an itinerary
- `activities` — individual activities per day
- `locations` — reusable venue/location records
- `cost_estimates` — cost breakdowns per itinerary

---

## What Was Excluded from MVP

| Feature | Reason |
|---------|--------|
| Calendar export (ICS) | Scope was deprioritized in favor of the full user auth + save flow |
| Shareable itinerary links | Not implemented — would require public token system |
| Itinerary regeneration with version history | Out of scope |
| Multiple cost tiers (budget/moderate/luxury toggle) | Cost estimation uses data from saved activities, not a separate tier model |

---

## Repository Structure

```
Link2itinerary/
├── README.md                        # Project overview, team, quick start
├── PROJECT-SETUP-SUMMARY.md         # This file
├── frontend/                        # React/Vite app
│   ├── README.md
│   ├── ux-flow.md
│   └── src/
│       ├── pages/                   # 9 page components
│       ├── components/              # Shared UI (header, footer, protected route)
│       ├── context/                 # AuthContext, TripContext
│       ├── services/                # api.ts (live), mocks.ts (dev fallback)
│       └── types/                   # TypeScript API types
├── backend/                         # NestJS API
│   ├── README.md
│   ├── GETTING-STARTED.md
│   ├── api-contracts.md
│   ├── .env.example
│   └── src/
│       ├── auth/
│       ├── trips/
│       ├── planner/
│       ├── estimator/
│       ├── itineraries/
│       ├── teaser-cache/
│       ├── itinerary-cache/
│       └── users/
├── db/
│   ├── schema-plan.md               # Full 9-table schema with SQL
│   └── ERD-notes.md                 # ER diagram with Mermaid
└── docs/
    ├── README.md
    ├── SRS-outline.md
    ├── architecture-overview.md
    ├── backend-progress.md
    ├── frontend-progress.md
    └── DocumentationsMD/MeetingNotes/
```

---

## Key Technical Decisions

**Caching layer:** Rather than calling OpenAI on every page load, generated itineraries are cached in the database. Teasers are cached 1:1 per trip seed. Full itineraries are cached per (trip seed + SHA-256 hash of preferences) — so repeat visits with the same preferences are instant, but changing any preference triggers a fresh generation.

**Optional auth on teaser:** The `POST /api/planner/teaser` endpoint uses an `OptionalJwtAuthGuard` — anonymous users can generate teasers, but if a JWT is present, the result is linked to their account.

**Mock API layer:** The frontend has a `VITE_USE_MOCKS=true` mode (see `services/mocks.ts`) that was used during development before the full backend was connected. All production calls go through `services/api.ts`.

**TypeORM synchronize:** In development, `synchronize: true` auto-creates tables from entity definitions. No separate migration files were maintained — schema changes were made directly to entities.
