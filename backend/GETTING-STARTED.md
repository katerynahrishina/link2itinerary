# Getting Started — Backend

A step-by-step guide to set up and run the Link2Itinerary backend locally.

## Prerequisites

- **Node.js** v18 or higher — [download here](https://nodejs.org/)
- **npm** (comes with Node.js)
- **Git** (to clone the repo)
- Access to the team's **Supabase project** (ask the project lead for an invite)
- An **OpenAI API key** — [get one here](https://platform.openai.com/api-keys)

Check your versions:
```bash
node -v   # should be v18+
npm -v    # should be v9+
```

## Setup (one-time)

### 1. Install dependencies

```bash
cd backend
npm install
```

### 2. Create your environment file

```bash
cp .env.example .env
```

### 3. Fill in environment variables

Open `backend/.env` and set the following values:

**DATABASE_URL** — Supabase PostgreSQL connection string:
1. Go to [supabase.com/dashboard](https://supabase.com/dashboard) and open the Link2Itinerary project
2. Click the **"Connect"** button in the top right corner
3. Copy the **Session Pooler** or **Transaction Pooler** URI

It should look like:
```
DATABASE_URL=postgresql://postgres.xxxxxxxxxxxx:YourPassword@aws-0-us-west-2.pooler.supabase.com:6543/postgres
```

**OPENAI_API_KEY** — required for the Planner module to generate itineraries:
```
OPENAI_API_KEY=sk-your-key-here
```

**JWT_SECRET** — any long random string used to sign authentication tokens:
```
JWT_SECRET=some-long-random-secret
```

**PORT** — defaults to 3000, change if that port is in use:
```
PORT=3000
```

## Running the server

```bash
npm run start:dev
```

This starts the server at **http://localhost:3000** with hot reload. The first run will automatically create all database tables via TypeORM's `synchronize` option.

You should see output ending with:
```
[Nest] LOG [NestApplication] Nest application successfully started
```

## Testing the API

### Authentication

**Register a new user:**
```bash
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{ "username": "testuser", "password": "password123" }'
```

Returns `{ "token": "...", "user": { "id": "...", "username": "..." } }`.

**Login:**
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{ "username": "testuser", "password": "password123" }'
```

### Trip Seeds

**Create a trip seed:**
```bash
curl -X POST http://localhost:3000/api/trips/seed \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://airbnb.com/rooms/12345",
    "summary": "Weekend getaway to Paris",
    "location": "Paris, France",
    "checkIn": "2026-05-01",
    "checkOut": "2026-05-05",
    "accommodationName": "Charming apartment in Le Marais",
    "accommodationType": "airbnb"
  }'
```

**List all trip seeds:**
```bash
curl http://localhost:3000/api/trips
```

**Get one trip seed:**
```bash
curl http://localhost:3000/api/trips/PASTE-UUID-HERE
```

**Update a trip seed:**
```bash
curl -X PATCH http://localhost:3000/api/trips/PASTE-UUID-HERE \
  -H "Content-Type: application/json" \
  -d '{ "summary": "Updated summary" }'
```

**Delete a trip seed:**
```bash
curl -X DELETE http://localhost:3000/api/trips/PASTE-UUID-HERE
```

Returns 204 (no content) on success.

### Saved Itineraries (requires JWT)

Replace `YOUR_TOKEN` with the token from login/register.

**List my saved itineraries:**
```bash
curl http://localhost:3000/api/itineraries \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Save an itinerary:**
```bash
curl -X POST http://localhost:3000/api/itineraries/ITINERARY-UUID/save \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Project structure

```
backend/src/
├── auth/                  # Register, login, JWT guards + strategy
├── trips/                 # Trip seed CRUD
├── planner/               # OpenAI itinerary generation
├── estimator/             # Cost calculation
├── itineraries/           # Saved itinerary management
├── teaser-cache/          # Caches teaser responses (avoids repeat OpenAI calls)
├── itinerary-cache/       # Caches full itinerary responses (keyed by preferences hash)
├── users/                 # User entity + service (no controller)
├── common/                # Shared utilities
├── app.module.ts          # Root module (TypeORM config, all module imports)
└── main.ts                # App entry point
```

## Useful commands

| Command | What it does |
|---------|-------------|
| `npm run start:dev` | Start dev server with hot reload |
| `npm run build` | Compile TypeScript to JavaScript |
| `npm run start:prod` | Run the compiled production build |
| `npm run test` | Run unit tests |
| `npm run test:e2e` | Run end-to-end tests |
| `npm run lint` | Check code style with ESLint |
| `npm run format` | Auto-format code with Prettier |

## Troubleshooting

**`ENOTFOUND db.xxx.supabase.co`**
Your connection string uses the old format. Use the pooler URI from the Supabase "Connect" button instead. The hostname should be `aws-0-REGION.pooler.supabase.com`.

**`nest: command not found`**
The NestJS CLI isn't installed globally. Use `npm run start:dev` — it doesn't require the global CLI.

**`Unable to connect to the database. Retrying...`**
Check that `DATABASE_URL` in `.env` is correct with no extra spaces or quotes.

**Port 3000 already in use**
Change `PORT` in `.env` to something else (e.g., `3001`).

**JWT errors on protected routes**
Make sure `JWT_SECRET` in `.env` matches what was used to sign the token. In development, the default `link2itinerary-dev-secret` is used if `JWT_SECRET` is not set.

**Empty response from `GET /api/itineraries`**
You need to be logged in and have saved at least one itinerary. Use the POST `/api/itineraries/:id/save` endpoint first.
