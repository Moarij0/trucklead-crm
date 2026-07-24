# 🚚 TruckLead CRM

A premium, ultra-modern **Lead Management CRM** for USA Truck Driver & Owner-Operator leads.
Built with a glassmorphism UI, black/white + blue (`#2563EB`) design, smooth micro-animations,
and a production-ready full-stack architecture.

> Design language inspired by **Linear, Stripe & Notion** — minimal, clean, futuristic.

---

## ✨ Features

| | |
|---|---|
| 📊 **Beautiful Dashboard** | Stat cards, 7-day trend chart, due follow-ups, recent leads |
| 🔍 **Powerful Search** | ⌘K command palette + full-text search across 8 fields |
| 🎛 **Advanced Filters** | Status, priority, equipment, state, due follow-ups |
| 🗂 **Lead Details Panel** | Slide-over drawer with everything about a lead |
| 📝 **Notes** | Add/delete timestamped notes per lead |
| 📞 **Call Status** | One-click call logging + status pipeline |
| ⏰ **Follow-up Reminders** | Set reminders (tomorrow / 3d / 7d), overdue highlighting |
| 📁 **CSV / Excel Export** | Export the current filtered result set |
| 🔐 **Team Login** | JWT authentication, multiple users |
| 📈 **Analytics** | Status pie, top states, equipment mix, conversion rate |
| 🌗 **Dark & Light Mode** | Persisted per user |
| 💀 **Loading Skeletons** | Shimmer placeholders everywhere |
| 🔔 **Toast Notifications** | Slick feedback on every action |
| ⌨️ **Keyboard Shortcuts** | `⌘K` search · `g d/l/a` navigate |
| 🎨 **White-label Branding** | Admin panel: logo, name, colors, favicon, contact — no code |
| 👥 **Admin & Team Management** | Add/disable agents, admin vs agent roles |
| 📥 **CSV / Excel Import** | Wizard: auto-detect columns, map, dedupe, preview, thousands of rows |
| 🐳 **One-command Deploy** | Docker + PostgreSQL + automatic HTTPS (see `DEPLOYMENT.md`) |

### Dashboard cards
Total Leads · Contacted · Interested · Follow-ups · Closed Deals

### Each lead card
Name · Public Business Phone · Business Email · Location · DOT/MC Number ·
Status · Notes · Quick Actions (Call · Email · WhatsApp when available)

---

## 🛠 Tech Stack

- **Frontend:** React 18 + Vite · Tailwind CSS · Framer Motion · Recharts · lucide-react
- **Backend:** FastAPI · SQLAlchemy · Pydantic v2 · JWT (python-jose) · passlib/bcrypt
- **Database:** PostgreSQL (production) — **SQLite by default** for zero-config local dev

---

## 🚀 Quick Start

### 🐳 Production / client deploy (recommended) — one command

```bash
cp .env.example .env    # set passwords, admin login, domain
docker compose up -d --build
```

PostgreSQL + backend + frontend + automatic HTTPS, all wired up. The database
schema is created automatically. Full walkthrough (domains, HTTPS, backups,
provider notes, handover checklist) is in **[DEPLOYMENT.md](DEPLOYMENT.md)**.

---

### 💻 Local development

> **Note:** on **Python 3.14**, install uses SQLite only (no PostgreSQL driver
> needed locally); the Postgres driver ships inside Docker on Python 3.12.

### 1) Backend (FastAPI)

```bash
cd backend
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
# source .venv/bin/activate

pip install -r requirements.txt
copy .env.example .env        # (Windows)  |  cp .env.example .env  (mac/linux)

uvicorn app.main:app --reload --port 8000
```

- API runs at **http://localhost:8000**
- Interactive docs at **http://localhost:8000/docs**
- On first run it **auto-creates tables and seeds ~40 realistic demo leads** + 2 users.

### 2) Frontend (React + Vite)

```bash
cd frontend
npm install
npm run dev
```

- App runs at **http://localhost:5173** (Vite proxies `/api` → `localhost:8000`).

### 3) Log in

```
Email:    demo@truckcrm.com
Password: demo1234
```

---

## 🐘 Switching to PostgreSQL

The app uses SQLAlchemy, so PostgreSQL works with **one env change**. In `backend/.env`:

```env
DATABASE_URL=postgresql+psycopg2://truckcrm:password@localhost:5432/truckcrm
```

Create the database, then start the backend — tables auto-create and seed on first boot.
`psycopg2-binary` is already in `requirements.txt`.

---

## 📡 API Overview

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/auth/login` | Login → JWT |
| `POST` | `/api/auth/register` | Create a team member |
| `GET`  | `/api/auth/me` | Current user |
| `GET`  | `/api/leads` | List (search, filters, sort, pagination) |
| `POST` | `/api/leads` | Create lead |
| `PATCH`| `/api/leads/{id}` | Update lead / status / follow-up |
| `DELETE`| `/api/leads/{id}` | Delete lead |
| `GET`  | `/api/leads/export` | CSV export (respects filters) |
| `POST` | `/api/leads/{id}/notes` | Add note |
| `DELETE`| `/api/leads/{id}/notes/{noteId}` | Delete note |
| `GET`  | `/api/analytics` | Dashboard + charts data |

All `/api/leads` and `/api/analytics` routes require a `Bearer` JWT.

---

## 📁 Project Structure

```
TruckLeadCRM/
├── backend/
│   ├── app/
│   │   ├── main.py           # FastAPI app + CORS + startup seed
│   │   ├── config.py         # env-based settings
│   │   ├── database.py       # SQLAlchemy engine/session
│   │   ├── models.py         # User, Lead, Note
│   │   ├── schemas.py        # Pydantic v2 schemas
│   │   ├── auth.py           # JWT + password hashing
│   │   ├── seed.py           # realistic demo data
│   │   └── routers/          # auth · leads · analytics
│   ├── requirements.txt
│   └── .env.example
└── frontend/
    ├── src/
    │   ├── components/        # Sidebar, Topbar, LeadCard, DetailPanel, CommandPalette...
    │   ├── pages/            # Login, Dashboard, Leads, Analytics
    │   ├── context/         # Auth + Theme
    │   ├── hooks/           # useLeads
    │   └── lib/             # api client + constants
    ├── tailwind.config.js
    └── vite.config.js
```

---

## ⌨️ Keyboard Shortcuts

- `⌘ / Ctrl + K` — open command palette / search
- `g` then `d` — go to Dashboard
- `g` then `l` — go to Leads
- `g` then `a` — go to Analytics
- `⌘ / Ctrl + Enter` — save a note (in the detail panel)

---

## 🔒 Production Notes

- Set a strong `SECRET_KEY` in `.env`.
- Point `DATABASE_URL` at managed PostgreSQL.
- Restrict `CORS_ORIGINS` to your real frontend domain.
- Set `SEED_ON_STARTUP=false` after first deploy.
- Build the frontend with `npm run build` and serve `dist/` behind your CDN / nginx.
