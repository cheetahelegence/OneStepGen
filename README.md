# OneStepGen — Smart Task Support for ADHD Workers

A web app for ADHD workers in Australia who need support managing day-to-day workplace tasks: dump ideas, let AI break them into small steps, plan in a priority matrix, match tasks to your energy, focus one task at a time, and celebrate progress with collectible rewards.

---

## Project Overview

Our target users are ADHD workers in Australia who experience difficulties managing day-to-day workplace tasks, particularly when dealing with unclear instructions, competing priorities, distractions, and overwhelming workloads. These challenges can affect their ability to start tasks, maintain focus, stay organised, and complete work confidently, which may negatively impact both work performance and emotional well-being.

OneStepGen reduces overwhelm through a **guided four-step workflow**, **AI-assisted task breakdown**, **energy-aware planning**, and **in-session support tools** (breathing exercises, rainbow grounding, helplines, and a Melbourne focus map in the nav). Session data stays in the browser (`localStorage`); nothing is stored on our servers by default.

---

## Key Features

### Four-step workflow

| Step | Route | What it does |
|------|--------|----------------|
| 1. AI dump | `/workflow/ai-dump` | Paste text (voice input supported), upload a PDF, or open **Session History** to restore a saved plan; backend extracts 2–5 minute actionable steps with priorities and cognitive-load scores |
| 2. Planner | `/workflow/planner` | Review, add, edit, and reorder tasks in an Eisenhower-style matrix; set your **current energy level** to highlight matching tasks; **save or update** up to 10 named histories |
| 3. Focus (swiper) | `/workflow/swiper` | One-task-at-a-time UI with timer; complete or skip; contextual **tips** by mood; optional **focus lock** |
| 4. Complete | `/workflow/complete` | Session summary and random Australian animal reward |

**Workflow progress** is stored in `localStorage`. You can only open steps you have already unlocked (`maxReachedStep`). When returning from outside the workflow, the app resumes your **last viewed step** (within unlocked steps), not always the furthest step reached.

### Energy level matching

- The AI rates each sub-task with `parent_task_cognitive_load` (1–5: low → high mental effort).
- In **Planner** and **Focus**, choose **Low / Normal / High** energy to highlight tasks that fit how you feel right now (all tasks stay visible).
- Logic lives in `src/utils/energyMatching.js`.

### Session history

- Save named task plans from **Planner** (max **10** entries in `localStorage` key `taskHistory`).
- Open **Session History** on **AI Dump** to reload a saved plan into a new session.
- Updating an existing history overwrites that entry; deleting removes it from the list.

### AI backend (`aiTool/`)

- Text and PDF input → Groq (Llama 3) task extraction → semantic urgency/importance scoring → cognitive-load metadata
- Supports multiple API keys (`GROQ_API_KEY_1`, `GROQ_API_KEY_2`, …) with rotation on rate limits
- FastAPI for local dev; `lambda_handler.py` for AWS Lambda deployment
- See [aiTool/README.md](aiTool/README.md) and [aiTool/FRONTEND_INTEGRATION.md](aiTool/FRONTEND_INTEGRATION.md) for API details

### Support & wellbeing

- **Floating support button** (global): box breathing, rainbow grounding, helpful resources / helplines (`SupportModal` → `SupportMenu`)
- **Focus map** (navbar, top right): Melbourne CBD quiet places and crowd data (`NavBar` → `QuietPlacesModal` → `QuietPlacesPanel`; uses City of Melbourne open data)
- **Focus step tips**: mood-based workplace tips via `TipsPanel` and `src/data/tips.JS`
- **About** page: employment/disability and psychological distress visualisations

### Rewards

- Collect Australian animal cards after completing a session; view the collection at `/reward`

### Privacy & access

- Planning data stored locally in the browser
- Optional **site password gate**: **server** mode (recommended — HttpOnly cookie via API) or **client** mode (phrase baked into the build)
- Privacy and terms pages: `/privacy`, `/terms`

---

## Tech Stack

| Layer | Technologies |
|--------|----------------|
| Frontend | Vue 3, Vue Router 5, Vite 8, Bootstrap 5 |
| Maps / charts | Leaflet, ECharts, D3, TopoJSON |
| Voice input | Web Speech API (`en-AU`) |
| State | Browser `localStorage` (sessions, task history, rewards) |
| Backend | Python 3.10+, FastAPI, Groq API (Llama 3) |
| Deploy (optional) | AWS Lambda + API Gateway + static hosting (e.g. S3 + CloudFront) |

**Node:** `^20.19.0` or `>=22.12.0` (see `package.json` `engines`)

---

## My Contributions

This was a team capstone project. My main contributions focused on frontend workflow integration, client-side task management, and cloud deployment.

- Implemented page routing and multi-step workflow navigation
- Developed task creation, prioritisation, editing, and deletion logic
- Built session history features using browser `localStorage`
- Contributed to frontend integration across the AI task breakdown, planning, focus, and completion workflow
- Worked on cloud deployment and domain configuration
- Contributed to the serverless deployment setup using API Gateway and AWS Lambda

---

## Project Status

This project was developed as a university capstone project and is no longer actively maintained.

The repository is preserved as a portfolio and reference implementation. Some external services, API endpoints, environment variables, or deployment configurations may no longer be active.

The local setup instructions below reflect the original development environment and may require minor updates depending on current dependency versions and API availability.

---

## How to Run

### Frontend (repository root)

```bash
git clone https://github.com/cheetahelegence/OneStepGen.git
cd OneStepGen
npm install
npm run dev
```

Open the URL Vite prints (typically `http://localhost:5173`).

**Other scripts**

```bash
npm run build    # production build → dist/
npm run preview  # preview production build
npm run lint     # oxlint + eslint
npm run format   # prettier (src/)
```

**Environment variables (frontend)**

Create `.env.development` or `.env.local` in the **project root** (Vite loads both in dev):

| Variable | Purpose |
|----------|---------|
| `VITE_API_BASE_URL` | Backend base URL. **Local dev:** `/api` (proxied to `http://127.0.0.1:8000` via `vite.config.js`). **Production:** absolute API Gateway URL |
| `VITE_SITE_GATE` | `server` \| `client` — password gate mode |
| `VITE_SITE_ACCESS_PASSWORD` | Client-only gate phrase (build-time; only when `VITE_SITE_GATE=client`) |
| `VITE_SKIP_SITE_GATE` | `true` to skip gate in development |
| `VITE_MELBOURNE_ODATA_BASE` | Optional proxy path for Melbourne open data (quiet places), e.g. `/melbourne-ods-api/api/explore/v2.1/catalog/datasets` |

> **Local CORS tip:** Do not point `VITE_API_BASE_URL` at `http://127.0.0.1:8000` unless your backend allows your dev origin. Use `/api` and the Vite proxy instead.

### AI backend (`aiTool/`)

```bash
cd aiTool
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env       # if present; see below
```

In **`aiTool/.env`** (not the frontend env file), set at least:

```env
GROQ_API_KEY_1=your-groq-api-key
```

Optional: `GROQ_API_KEY_2`, … for rotation when rate-limited.

```bash
uvicorn main:app --reload
```

API docs: `http://127.0.0.1:8000/docs`

**Full local stack:** run both servers; set `VITE_API_BASE_URL=/api` in the frontend env file and restart `npm run dev` after any env change.

---

## Project Structure

```
OneStepGen/
├── public/                 # Static assets, focus map CSVs, chart data, animal images
├── src/
│   ├── assets/             # Images, fonts, global CSS
│   ├── components/         # TaskCard, timers, support panels, modals, …
│   ├── data/               # Animals, tips (Focus step), map sources
│   ├── router/
│   │   ├── index.js        # Routes + workflow step guards
│   │   └── workflow.js     # Step state, session & history localStorage API
│   ├── utils/
│   │   ├── siteAccess.js   # Site password gate (client / server)
│   │   └── energyMatching.js
│   ├── views/              # Home, workflow (4 steps), About, Reward, Privacy, Terms
│   ├── App.vue
│   └── main.js
├── aiTool/                 # FastAPI AI pipeline
│   ├── main.py
│   ├── lambda_handler.py
│   ├── utils/              # pdf_parser, ai_processor, scoring
│   ├── requirements.txt
│   └── README.md
├── index.html
├── vite.config.js
└── package.json
```

### Routes & views

All user-facing pages are registered in `src/router/index.js`:

| Path | Component |
|------|-----------|
| `/` | `HomeView.vue` |
| `/workflow/ai-dump` | `AIDump.vue` — input, voice, PDF, session history |
| `/workflow/planner` | `PlannerView.vue` — matrix, energy matching, save history |
| `/workflow/swiper` | `TaskSwipper.vue` — focus mode, timer, mood tips |
| `/workflow/complete` | `CompleteView.vue` — session completion |
| `/reward` | `RewardView.vue` — animal collection |
| `/about` | `AboutView.vue` — data visualisations |
| `/privacy` | `PrivacyView.vue` |
| `/terms` | `TermsView.vue` |

Global UI (not separate routes): `NavBar` (includes **Focus map**), `SitePasswordGate`, `FloatingSupportButton` / `SupportModal`, workflow `BottomNav`.

### localStorage keys (reference)

| Key | Purpose |
|-----|---------|
| `onestep-current-session` | Active workflow session |
| `taskHistory` | Up to 10 saved named plans |
| `onestep-animal-collection` | Earned reward animals |

---

## Target Users

ADHD workers in Australia who experience difficulties managing day-to-day workplace tasks, particularly when dealing with:

- Unclear instructions
- Competing priorities
- Distractions
- Overwhelming workloads

These challenges can affect their ability to start tasks, maintain focus, stay organised, and complete work confidently, which may negatively impact both work performance and emotional well-being.

---

## Related Documentation

- [aiTool/README.md](aiTool/README.md) — AI pipeline, scoring model, API testing
- [aiTool/FRONTEND_INTEGRATION.md](aiTool/FRONTEND_INTEGRATION.md) — Connecting the Vue app to the API

---

## Future Improvements

- Deeper emotional support integrations
- Progress reward visualisation enhancements
- Focus awareness check-ins
- End-to-end user data export / backup (optional; currently client-only storage)
