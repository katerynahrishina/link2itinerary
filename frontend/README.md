# Frontend — Link2Itinerary UI

React/Vite web application for Link2Itinerary. Provides the full user-facing experience from landing to viewing and saving AI-generated travel itineraries.

## Pages

### 1. Landing Page
**Route:** `/`

Hero section with the project pitch, a "Get Started" CTA, and feature highlights.

### 2. Login Page
**Route:** `/login`

Combined register/login form. Submits to `/api/auth/register` or `/api/auth/login`. On success, stores the JWT in `localStorage` under `link2itinerary.auth.token` and updates `AuthContext`.

### 3. Trip Seed Page
**Route:** `/create/seed`

Collects trip details: travel link URL, destination, check-in/check-out dates, accommodation name and type. Submits to `POST /api/trips/seed`.

### 4. Teaser Page
**Route:** `/trips/:id/teaser`

Calls `POST /api/planner/teaser` with the trip ID. Displays a 3-day overview with themes, highlights, and a cost estimate range. Includes a teaser popup modal with trip summary details.

### 5. Preferences Page
**Route:** `/trips/:id/preferences`

Multi-step form for collecting: interests (museums, food, nightlife, nature, etc.), budget tier, travel pace, dietary restrictions, and accessibility needs. On submit, navigates to the Generating page.

### 6. Generating Page
**Route:** `/trips/:id/generating`

Loading state shown while `POST /api/planner/full` runs. Displays a progress animation while waiting for the OpenAI response.

### 7. Itinerary Page
**Route:** `/trips/:id/itinerary`

Full day-by-day itinerary view. Shows activities with times, descriptions, and cost estimates. Includes a cost breakdown section. If the user is logged in, an "Add to My Itineraries" button calls `POST /api/itineraries/:id/save`.

### 8. My Itineraries Page
**Route:** `/my-itineraries`

Protected route (requires login). Lists all itineraries the user has saved, pulled from `GET /api/itineraries`. Includes a teaser popup modal for quick preview.

### 9. Saved Itinerary Page
**Route:** `/my-itineraries/:id`

Protected route. Read-only view of a single saved itinerary, loaded from `GET /api/itineraries/:id`.

## Directory Structure

```
frontend/src/
├── pages/
│   ├── LandingPage.tsx
│   ├── LoginPage.tsx
│   ├── TripSeedPage.tsx
│   ├── TeaserPage.tsx
│   ├── PreferencesPage.tsx
│   ├── GeneratingPage.tsx
│   ├── ItineraryPage.tsx
│   ├── MyItinerariesPage.tsx
│   └── SavedItineraryPage.tsx
├── components/
│   ├── ProtectedRoute.tsx           # Redirects to /login if no auth token
│   └── common/
│       ├── AppHeader.tsx
│       ├── AppFooter.tsx
│       ├── LoadingState.tsx
│       └── ErrorState.tsx
├── context/
│   ├── AuthContext.tsx              # Auth state (user, token, login/logout)
│   └── TripContext.tsx              # Current trip state
├── services/
│   ├── api.ts                       # Live API calls (fetch + JWT headers)
│   └── mocks.ts                     # Mock responses for development
├── types/
│   └── api.ts                       # TypeScript types for all API shapes
├── App.tsx                          # Router setup and route definitions
└── main.tsx                         # Entry point
```

## State Management

**Auth:** `AuthContext` stores the current user and JWT. The token is persisted to `localStorage` under `link2itinerary.auth.token`. `ProtectedRoute` wraps pages that require login.

**Trip flow:** `TripContext` holds the current trip seed data as the user progresses through the creation flow.

## API Service Layer

All backend calls go through `services/api.ts`. A `VITE_USE_MOCKS=true` environment variable switches to the mock implementations in `services/mocks.ts` — useful for frontend development without a running backend.

Protected endpoints (planner/full, itineraries) automatically attach the JWT from `localStorage` in the `Authorization: Bearer` header.

## Technology Stack

- **Framework:** React 18, TypeScript
- **Build Tool:** Vite 5
- **Routing:** React Router v7
- **Testing:** Vitest
- **Linting:** ESLint

## Development Setup

```bash
cd frontend
npm install
```

Create `frontend/.env`:

```env
VITE_API_BASE_URL=http://localhost:3000/api
VITE_USE_MOCKS=false
```

Start the dev server (runs at http://localhost:5173):

```bash
npm run dev
```

Other commands:

| Command | Description |
|---------|-------------|
| `npm run dev` | Start dev server with hot reload |
| `npm run build` | Production build |
| `npm run preview` | Preview the production build |
| `npm run test` | Run unit tests with Vitest |
| `npm run lint` | Check code style |
