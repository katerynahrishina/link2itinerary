# Backend — Link2Itinerary API

NestJS REST API server for Link2Itinerary, handling authentication, trip management, LLM-powered itinerary generation, cost estimation, and data persistence via Supabase PostgreSQL.

## Architecture

The backend is organized into five NestJS feature modules plus two caching modules:

| Module | Routes | Purpose | Built by |
|--------|--------|---------|----------|
| **Auth** | `/api/auth` | User registration and login (bcrypt + JWT) | Kateryna Hrishina |
| **Trips** | `/api/trips` | Trip seed CRUD (create, read, update, delete) | Kateryna Hrishina, Betty Phipps |
| **Planner** | `/api/planner` | Teaser and full itinerary generation via OpenAI | Jakob Midthun |
| **Estimator** | `/api/estimator` | Cost calculation from saved itinerary data | Kateryna Hrishina |
| **Itineraries** | `/api/itineraries` | Saved itinerary management (list, view, save) | Kateryna Hrishina |
| **Teaser Cache** | — | Caches teaser results to avoid redundant OpenAI calls | Kateryna Hrishina |
| **Itinerary Cache** | — | Caches full itineraries keyed by trip + preferences hash | Kateryna Hrishina |

## API Endpoints

### Auth — `/api/auth`

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/api/auth/register` | Public | Register a new user. Returns JWT token + user object. |
| POST | `/api/auth/login` | Public | Login with username/password. Returns JWT token + user object. |

### Trips — `/api/trips`

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/api/trips/seed` | Public | Create a trip seed from a travel link. |
| GET | `/api/trips` | Public | List all trip seeds (most recent first). |
| GET | `/api/trips/:id` | Public | Get a single trip seed by UUID. |
| PATCH | `/api/trips/:id` | Public | Partially update a trip seed. |
| DELETE | `/api/trips/:id` | Public | Delete a trip seed (returns 204). |

### Planner — `/api/planner`

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/api/planner/from-url` | Public | Extract trip metadata from a URL (scraper + OpenAI). |
| POST | `/api/planner/teaser` | Optional JWT | Generate a 3-day teaser itinerary. Result cached. If JWT present, itinerary is linked to the user. |
| POST | `/api/planner/full` | **JWT required** | Generate a full day-by-day itinerary with preferences. Result cached by trip + preferences hash. |

### Estimator — `/api/estimator`

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/api/estimator/calculate` | Public | Calculate a cost breakdown from a saved itinerary. Body: `{ itineraryId }`. |

### Itineraries — `/api/itineraries`

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| GET | `/api/itineraries` | **JWT required** | List all saved itineraries for the current user. |
| GET | `/api/itineraries/:id` | **JWT required** | Get full detail for one saved itinerary. |
| POST | `/api/itineraries/:id/save` | **JWT required** | Mark an itinerary as saved ("Add to My Itineraries"). |

## Technology Stack

- **Framework:** NestJS 11, TypeScript
- **Database ORM:** TypeORM 0.3
- **LLM Client:** OpenAI SDK 6
- **Auth:** Passport.js, `@nestjs/jwt`, bcrypt
- **Validation:** class-validator, class-transformer
- **Web Scraping:** Cheerio
- **Testing:** Jest, Supertest
- **Code Quality:** ESLint, Prettier

## Environment Variables

Copy `.env.example` to `.env` and fill in your values:

```env
# Server
PORT=3000

# Supabase PostgreSQL
DATABASE_URL=postgresql://postgres.[PROJECT-REF]:[PASSWORD]@aws-0-[REGION].pooler.supabase.com:6543/postgres

# OpenAI (required for Planner module)
OPENAI_API_KEY=sk-your-key-here

# JWT secret (can be any long random string; default dev secret is used if omitted)
JWT_SECRET=your-secret-here
```

## Directory Structure

```
backend/src/
├── auth/                        # Auth module (register, login, JWT guards)
│   ├── dto/                     #   Register/Login DTOs
│   ├── guards/                  #   JwtAuthGuard, OptionalJwtAuthGuard
│   ├── strategies/              #   JWT strategy
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   └── auth.module.ts
├── trips/                       # Trip seeds module (CRUD)
│   ├── dto/
│   ├── entities/trip-seed.entity.ts
│   ├── trips.controller.ts
│   ├── trips.service.ts
│   └── trips.module.ts
├── planner/                     # Itinerary generation module
│   ├── dto/
│   ├── types/
│   ├── planner.controller.ts
│   ├── planner.service.ts
│   └── planner.module.ts
├── estimator/                   # Cost estimation module
│   ├── dto/
│   ├── types/
│   ├── estimator.controller.ts
│   ├── estimator.service.ts
│   └── estimator.module.ts
├── itineraries/                 # Saved itinerary management
│   ├── itineraries.controller.ts
│   ├── itineraries.service.ts
│   └── itineraries.module.ts
├── teaser-cache/                # Caches teaser OpenAI responses
│   ├── entities/teaser-cache.entity.ts
│   └── teaser-cache.module.ts
├── itinerary-cache/             # Caches full itinerary responses
│   ├── entities/itinerary-cache.entity.ts
│   └── itinerary-cache.module.ts
├── users/                       # User management (service only)
│   ├── entities/user.entity.ts
│   ├── users.service.ts
│   └── users.module.ts
├── common/ids.ts                # Shared UUID utilities
├── app.module.ts                # Root module (TypeORM config, global imports)
└── main.ts                      # Entry point (starts server on PORT)
```

## Development Setup

```bash
cd backend
npm install
cp .env.example .env
# Edit .env with your values
npm run start:dev
```

Server starts at **http://localhost:3000** with hot reload.

See [GETTING-STARTED.md](./GETTING-STARTED.md) for detailed instructions and troubleshooting.

## Useful Commands

| Command | Description |
|---------|-------------|
| `npm run start:dev` | Start dev server with hot reload |
| `npm run build` | Compile TypeScript |
| `npm run start:prod` | Run compiled production build |
| `npm run test` | Run unit tests |
| `npm run test:e2e` | Run end-to-end tests |
| `npm run lint` | Check code style |
| `npm run format` | Auto-format with Prettier |

See [api-contracts.md](./api-contracts.md) for full request/response specifications.
