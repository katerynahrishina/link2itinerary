# Link2Itinerary

**One-line pitch:** Paste a travel link, get an AI-powered personalized itinerary with real-time cost estimates and smart suggestions.

## Problem & Solution

**Problem:** Planning a trip is overwhelming — travelers face scattered information, hidden costs, and time-consuming research across multiple websites.

**Solution:** Link2Itinerary transforms any travel destination link (Airbnb, hotel, attraction) into a comprehensive, personalized itinerary using OpenAI and an intelligent agentic workflow.

**Core user flow:** Paste an Airbnb link → see a 3-day teaser plan → add your preferences → get a full itinerary with cost breakdowns → save it to your account.

## Tech Stack

- **Frontend:** React 18, TypeScript, Vite, React Router v7
- **Backend:** NestJS, TypeScript, TypeORM
- **AI/LLM:** OpenAI GPT-4o
- **Database:** PostgreSQL (hosted via Supabase)
- **Auth:** Passport.js + JWT (bcrypt password hashing)

## Team

| Name | Role | Primary Responsibilities |
|------|------|-------------------------|
| Kateryna Hrishina | Backend Lead & Database Architect | Backend server setup, PostgreSQL schema, REST API endpoints, cost estimator, caching system |
| Jakob Midthun | Planner Module Developer | Planner module (controller, service, module), teaser & full itinerary generation, URL request handling |
| Betty Phipps | Project Lead & Full-Stack Contributor | Team coordination, database setup support, trips module backend, AI itinerary workflow integration |
| Paul Harrington | Frontend Developer & Integration | Frontend login flow, frontend/backend integration, testing with mock and live APIs |
| Myles Stowe | Frontend Architecture & UI/UX | Frontend architecture and file structure, UI/UX design, end-to-end testing |
| Mashhood Syed | Documentation & QA Support | Frontend development, mock frontend, documentation, project progress monitoring |

### Who Owns What

- **Trip Planning Engine:** Jakob Midthun, Betty Phipps
- **LLM Integration & Prompts:** Jakob Midthun
- **Frontend UI/UX:** Myles Stowe, Paul Harrington, Mashhood Syed
- **Cost Estimation:** Kateryna Hrishina
- **Database Schema:** Kateryna Hrishina, Betty Phipps
- **Backend API:** Kateryna Hrishina, Betty Phipps, Jakob Midthun
- **Caching System:** Kateryna Hrishina
- **Testing & QA:** Myles Stowe, Paul Harrington
- **Documentation:** Mashhood Syed (primary), all members contributed

## Repository Map

```
/
├── frontend/          # React/Vite application (9 pages)
├── backend/           # NestJS API server (5 modules)
├── docs/              # Course deliverables and documentation
├── db/                # Database schema and ER diagrams
├── README.md
└── LICENSE
```

### `/frontend/`
React application with nine pages covering the full user journey: landing, login, trip seed creation, teaser view, preferences, generating state, full itinerary, and a "My Itineraries" section backed by the database.

See [frontend/README.md](./frontend/README.md) for details.

### `/backend/`
NestJS API server with five modules: **Auth** (register/login), **Trips** (trip seed CRUD), **Planner** (itinerary generation via OpenAI), **Estimator** (cost calculations), and **Itineraries** (saved itinerary management). Includes a dual caching layer (teaser cache + itinerary cache) to avoid redundant OpenAI calls.

See [backend/README.md](./backend/README.md) for details.

### `/docs/`
Course deliverables including the Software Requirements Specification, architecture documentation, test plans, and meeting notes.

See [docs/README.md](./docs/README.md) for an index.

### `/db/`
Database design including the full schema (9 tables) and ER diagram notes.

See [db/schema-plan.md](./db/schema-plan.md) for details.

## Quick Start

### Prerequisites

- **Node.js** v18+
- **npm** v9+
- A **Supabase** project (PostgreSQL)
- An **OpenAI API key**

### 1. Clone the repository

```bash
git clone <repository-url>
cd Link2itinerary
```

### 2. Set up the backend

```bash
cd backend
npm install
cp .env.example .env
```

Edit `backend/.env` with your values:

```env
PORT=3000
DATABASE_URL=postgresql://postgres.[YOUR-PROJECT-REF]:[YOUR-PASSWORD]@aws-0-[YOUR-REGION].pooler.supabase.com:6543/postgres
OPENAI_API_KEY=sk-your-key-here
JWT_SECRET=your-secret-here
```

Start the backend (runs at http://localhost:3000):

```bash
npm run start:dev
```

### 3. Set up the frontend

```bash
cd ../frontend
npm install
```

Create `frontend/.env`:

```env
VITE_API_BASE_URL=http://localhost:3000/api
VITE_USE_MOCKS=false
```

Start the frontend (runs at http://localhost:5173):

```bash
npm run dev
```

### 4. Database

The backend uses TypeORM with `synchronize: true` in development, so tables are created automatically on first run. No manual migrations needed.

See [backend/GETTING-STARTED.md](./backend/GETTING-STARTED.md) for detailed setup instructions and troubleshooting.

## Project Status

**Current Phase:** Complete — MVP Delivered

**What was built:**
- ✅ Trip seed creation from travel links
- ✅ Teaser itinerary generation (3-day overview via OpenAI)
- ✅ Full itinerary generation with user preferences
- ✅ Cost estimation with category breakdowns
- ✅ User authentication (register/login with JWT)
- ✅ Saved itineraries ("My Itineraries" page, backed by database)
- ✅ Dual caching layer (teaser cache + full itinerary cache — no repeated OpenAI calls)
- ✅ Frontend fully connected to live backend API
- ❌ Calendar export (ICS) — excluded from MVP
- ❌ Shareable itinerary links — excluded from MVP

## Course Information

**Course:** CEN4090L — Software Engineering II
**Academic Year:** 2026
**Institution:** Florida State University
