# AI Resume Screening SaaS

> An enterprise-grade, AI-powered hiring platform that parses resumes, scores candidates, and matches them with job descriptions using NLP and LLMs — built for multi-tenant SaaS deployment.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Next.js](https://img.shields.io/badge/Next.js-14-black)](https://nextjs.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.111-009688)](https://fastapi.tiangolo.com/)
[![Python](https://img.shields.io/badge/Python-3.11+-3776AB)](https://python.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5+-3178C6)](https://typescriptlang.org/)

---

## What It Does

- **Parses** resumes in PDF, DOC, and DOCX format using PyMuPDF, pdfplumber, and python-docx
- **Extracts** skills, experience, and education using spaCy NER pipelines
- **Scores** candidates against job descriptions using LangChain + GPT-4 / Groq
- **Ranks** candidates semantically using Sentence Transformers + Pinecone vector search
- **Manages** the full hiring pipeline from application to interview
- **Supports** multiple companies, roles, and teams with full RBAC

---

## Repository Structure

```
ai-resume-screening-saas/
├── frontend/          # Next.js 14 — HR dashboard, upload UI, candidate ranking
├── backend/           # FastAPI — AI scoring, resume parsing, async workers
├── .gitignore
├── LICENSE
└── README.md          # You are here
```

---

## Tech Stack

### Frontend
| | |
|---|---|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript 5+ |
| Styling | Tailwind CSS |
| UI Components | Shadcn UI + Radix UI |
| State | Zustand |
| Data Fetching | React Query + Axios |
| Charts | Recharts |
| Auth | Auth0 / Clerk |
| Payments | Stripe.js |
| Hosting | Vercel |

### Backend
| | |
|---|---|
| Framework | FastAPI (Python 3.11+) |
| ORM | SQLAlchemy 2.0 (async) |
| Database | PostgreSQL 15 |
| Task Queue | Celery + Redis |
| NLP | spaCy + Sentence Transformers |
| LLM | LangChain + OpenAI / Groq |
| Vector DB | Pinecone |
| Storage | AWS S3 / Cloudflare R2 |
| Payments | Stripe |
| Hosting | Render / AWS |

---

## Getting Started

### Prerequisites

- Node.js 20+
- Python 3.11+
- Docker & Docker Compose
- PostgreSQL 15
- Redis

### 1. Clone the repo

```bash
git clone https://github.com/yjaztech/ai-resume-screening-saas.git
cd ai-resume-screening-saas
```

### 2. Start the backend

```bash
cd backend
cp .env.example .env
# Fill in your environment variables
docker compose up --build
```

Backend runs at **http://localhost:8000** — API docs at **http://localhost:8000/docs**

### 3. Start the frontend

```bash
cd frontend
cp .env.example .env.local
# Fill in your environment variables
npm install
npm run dev
```

Frontend runs at **http://localhost:3000**

---

## Features

### For HR Teams
- **Multi-company accounts** — isolated tenants, each with their own jobs and candidates
- **Bulk resume upload** — upload up to 100 resumes at once, processed asynchronously
- **AI candidate ranking** — every candidate scored 0–100 against the job description
- **Skill extraction** — automatic skill, experience, and education tagging
- **Interview pipeline** — kanban-style board to move candidates through hiring stages
- **Interview question generation** — LLM-generated questions tailored to each candidate
- **Analytics dashboard** — hiring funnel metrics, time-to-hire, source breakdown
- **CSV export** — export candidate data for reporting

### For Developers
- **Versioned REST API** — all endpoints under `/api/v1/`
- **Async processing** — resume parsing and AI scoring run as Celery background tasks
- **Role-based access control** — owner / admin / recruiter / viewer roles
- **Multi-tenancy** — every DB query scoped to `company_id` from JWT
- **Webhook support** — Stripe events handled for subscription lifecycle

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                    Frontend (Vercel)                 │
│              Next.js 14 + TypeScript                 │
└────────────────────┬────────────────────────────────┘
                     │ HTTPS / REST
┌────────────────────▼────────────────────────────────┐
│                 Backend (Render / AWS)               │
│                  FastAPI + Python                    │
│                                                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │  API Layer  │  │  Services   │  │   Workers   │  │
│  │  (Routers)  │→ │  (Business  │→ │  (Celery +  │  │
│  │             │  │   Logic)    │  │   Redis)    │  │
│  └─────────────┘  └──────┬──────┘  └─────────────┘  │
└─────────────────────────┼───────────────────────────┘
                           │
          ┌────────────────┼─────────────────┐
          │                │                 │
   ┌──────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐
   │ PostgreSQL  │  │  Pinecone   │  │   AWS S3    │
   │  (primary)  │  │  (vectors)  │  │  (storage)  │
   └─────────────┘  └─────────────┘  └─────────────┘
```

---

## Resume Processing Pipeline

```
Upload (PDF/DOC/DOCX)
        ↓
   Store in S3
        ↓
  Queue Celery Task
        ↓
  Parse Text           ← PyMuPDF / pdfplumber / python-docx
        ↓
  Extract Skills       ← spaCy NER
        ↓
  Generate Embeddings  ← Sentence Transformers → Pinecone
        ↓
  AI Score (0–100)     ← LangChain + GPT-4 / Groq
        ↓
  Save to PostgreSQL
        ↓
  Frontend Updates (polling → ranked candidate list)
```

---

## API Overview

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/v1/auth/login` | Authenticate user |
| `GET` | `/api/v1/jobs` | List company jobs |
| `POST` | `/api/v1/jobs` | Create job with JD |
| `POST` | `/api/v1/resumes/upload` | Upload resumes (bulk) |
| `GET` | `/api/v1/candidates` | Ranked candidates for a job |
| `GET` | `/api/v1/candidates/{id}` | Full candidate profile + scores |
| `PUT` | `/api/v1/pipeline/{id}/stage` | Move candidate in pipeline |
| `GET` | `/api/v1/analytics/dashboard` | Hiring funnel metrics |
| `POST` | `/api/v1/billing/checkout` | Stripe checkout session |
| `POST` | `/api/v1/webhooks` | Stripe event receiver |

Full interactive docs: `http://localhost:8000/docs`

---

## Role-Based Access Control

| Role | Capabilities |
|---|---|
| `owner` | Everything — billing, team, all data |
| `admin` | Jobs, candidates, pipeline, analytics |
| `recruiter` | View jobs, manage pipeline, export CSV |
| `viewer` | Read-only access |

---

## Deployment

| Service | Platform |
|---|---|
| Frontend | Vercel (`vercel --prod`) |
| Backend API | Render Web Service or AWS ECS |
| Celery Worker | Render Background Worker or AWS ECS |
| Database | Render PostgreSQL or AWS RDS |
| Redis | Render Redis or AWS ElastiCache |
| File Storage | AWS S3 or Cloudflare R2 |
| CI/CD | GitHub Actions |

---

## Documentation

- [Frontend README](./frontend/README.md) — setup, structure, env vars, Stripe, deployment
- [Backend README](./backend/README.md) — setup, Docker, API reference, AI pipeline, migrations

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/your-feature`
3. Commit with Conventional Commits: `feat:`, `fix:`, `chore:`
4. Push and open a Pull Request against `main`
5. Ensure all CI checks pass before requesting review

---

## License

MIT — see [LICENSE](./LICENSE)

---

<p align="center">Built by <a href="https://github.com/yjaztech">yjaztech</a></p>
