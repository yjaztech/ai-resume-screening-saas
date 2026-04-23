
# Resume Screening SaaS — Backend

> Production-grade FastAPI backend powering AI resume screening, semantic search, async job processing, and SaaS billing.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI (Python 3.11+) |
| Validation | Pydantic v2 |
| ORM | SQLAlchemy 2.0 (async) |
| Database | PostgreSQL 15 |
| Migrations | Alembic |
| Task Queue | Celery + Redis |
| Vector DB | Pinecone (managed) / Weaviate (self-hosted) |
| NLP | spaCy, Sentence Transformers |
| LLM | LangChain + OpenAI / Groq |
| Resume Parsing | PyMuPDF, pdfplumber, python-docx |
| Auth | Auth0 / Clerk (JWT verification) |
| Storage | AWS S3 / Cloudflare R2 |
| Payments | Stripe |
| Containerization | Docker + Docker Compose |

---

## Prerequisites

- Python 3.11+
- Docker & Docker Compose
- PostgreSQL 15 (or use the Docker Compose service)
- Redis (or use the Docker Compose service)

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/your-org/resume-screening-backend.git
cd resume-screening-backend
```

### 2. Configure environment variables

```bash
cp .env.example .env
```

Edit `.env`:

```env
# App
APP_ENV=development
SECRET_KEY=your-super-secret-key-minimum-32-chars
ALLOWED_ORIGINS=http://localhost:3000

# Database
DATABASE_URL=postgresql+asyncpg://postgres:password@localhost:5432/resume_screening

# Redis
REDIS_URL=redis://localhost:6379/0

# Auth (Auth0)
AUTH0_DOMAIN=your-tenant.auth0.com
AUTH0_AUDIENCE=https://api.your-app.com
AUTH0_ALGORITHMS=RS256

# Auth (Clerk — alternative)
CLERK_SECRET_KEY=sk_test_...
CLERK_PUBLISHABLE_KEY=pk_test_...

# OpenAI / Groq
OPENAI_API_KEY=sk-...
GROQ_API_KEY=gsk_...

# Pinecone
PINECONE_API_KEY=your-pinecone-key
PINECONE_ENVIRONMENT=us-east-1
PINECONE_INDEX_NAME=resume-embeddings

# AWS S3
AWS_ACCESS_KEY_ID=your-key-id
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_S3_BUCKET=resume-screening-uploads
AWS_REGION=us-east-1

# Cloudflare R2 (alternative to S3)
R2_ACCOUNT_ID=your-account-id
R2_ACCESS_KEY_ID=your-key-id
R2_SECRET_ACCESS_KEY=your-secret
R2_BUCKET_NAME=resume-screening-uploads

# Stripe
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PRICE_ID_STARTER=price_...
STRIPE_PRICE_ID_PRO=price_...
STRIPE_PRICE_ID_ENTERPRISE=price_...
```

### 3. Start with Docker Compose (recommended)

```bash
docker compose up --build
```

This starts:
- `api` — FastAPI on port 8000
- `worker` — Celery worker
- `db` — PostgreSQL on port 5432
- `redis` — Redis on port 6379

### 4. Run database migrations

```bash
docker compose exec api alembic upgrade head
```

### 5. (Optional) Run locally without Docker

```bash
python -m venv .venv
source .venv/bin/activate       # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# Run API
uvicorn app.main:app --reload --port 8000

# Run Celery worker (separate terminal)
celery -A app.workers.celery_app worker --loglevel=info
```

API is available at [http://localhost:8000](http://localhost:8000).  
Interactive docs: [http://localhost:8000/docs](http://localhost:8000/docs)

---

## Project Structure

```
resume-screening-backend/
├── app/
│   ├── main.py                   # FastAPI app factory, middleware, router registration
│   ├── config.py                 # Pydantic BaseSettings — all env vars typed
│   ├── dependencies.py           # DI providers: get_db, get_current_user, get_s3
│   │
│   ├── api/
│   │   └── v1/
│   │       ├── auth.py           # Login, token refresh, logout
│   │       ├── companies.py      # Tenant management
│   │       ├── jobs.py           # CRUD + job description management
│   │       ├── resumes.py        # Upload, parse trigger, list
│   │       ├── candidates.py     # Ranked list, profile, status update
│   │       ├── scoring.py        # Trigger re-score, get scores
│   │       ├── pipeline.py       # Interview stages, move candidate
│   │       ├── analytics.py      # Dashboard metrics, funnel data
│   │       ├── billing.py        # Stripe checkout, portal, plans
│   │       └── webhooks.py       # Stripe event handling
│   │
│   ├── core/
│   │   ├── security.py           # JWT decode + RBAC enforcement
│   │   ├── permissions.py        # Role definitions (owner/admin/recruiter/viewer)
│   │   ├── exceptions.py         # Custom HTTP exceptions
│   │   └── middleware.py         # Request logging, rate limiting
│   │
│   ├── models/                   # SQLAlchemy ORM models
│   │   ├── company.py
│   │   ├── user.py
│   │   ├── job.py
│   │   ├── resume.py
│   │   ├── candidate.py
│   │   ├── score.py
│   │   └── subscription.py
│   │
│   ├── schemas/                  # Pydantic request/response schemas
│   │   ├── job.py
│   │   ├── resume.py
│   │   ├── candidate.py
│   │   └── score.py
│   │
│   ├── services/                 # Business logic layer
│   │   ├── resume_parser.py      # PyMuPDF + pdfplumber + python-docx
│   │   ├── ai_scorer.py          # LangChain chain — scores resume vs JD
│   │   ├── job_matcher.py        # Semantic similarity ranking
│   │   ├── skill_extractor.py    # spaCy NER pipeline
│   │   ├── interview_gen.py      # LLM interview question generation
│   │   ├── vector_search.py      # Pinecone upsert / query
│   │   ├── storage.py            # S3 / R2 upload, presigned URLs
│   │   └── billing.py            # Stripe SDK calls
│   │
│   ├── workers/
│   │   ├── celery_app.py         # Celery + Redis broker config
│   │   └── tasks/
│   │       ├── parse_resume.py   # Extract text from uploaded file
│   │       ├── score_candidate.py# AI scoring task
│   │       ├── embed_resume.py   # Generate + upsert vectors
│   │       └── bulk_process.py   # Batch upload orchestrator
│   │
│   ├── db/
│   │   ├── session.py            # Async SQLAlchemy engine + session factory
│   │   ├── base.py               # DeclarativeBase
│   │   └── migrations/           # Alembic versions
│   │
│   └── utils/
│       ├── logger.py
│       ├── pagination.py
│       └── validators.py
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── conftest.py
│
├── docker/
│   ├── Dockerfile                # API image
│   └── Dockerfile.worker         # Celery worker image
│
├── docker-compose.yml
├── docker-compose.prod.yml
├── pyproject.toml
├── requirements.txt
└── .env.example
```

---

## API Reference

Interactive Swagger docs available at `/docs`. ReDoc at `/redoc`.

### Core Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/v1/auth/login` | Exchange token for session |
| `GET` | `/api/v1/jobs` | List all jobs for company |
| `POST` | `/api/v1/jobs` | Create a new job |
| `POST` | `/api/v1/resumes/upload` | Upload resume(s) — triggers async parsing |
| `GET` | `/api/v1/resumes/status/{task_id}` | Check parse/score task status |
| `GET` | `/api/v1/candidates` | Ranked candidates for a job |
| `GET` | `/api/v1/candidates/{id}` | Candidate profile + score breakdown |
| `PUT` | `/api/v1/pipeline/{candidate_id}/stage` | Move candidate through interview stages |
| `GET` | `/api/v1/analytics/dashboard` | Funnel metrics and hiring analytics |
| `POST` | `/api/v1/billing/checkout` | Create Stripe checkout session |
| `POST` | `/api/v1/billing/portal` | Open Stripe customer portal |
| `POST` | `/api/v1/webhooks` | Stripe webhook receiver |

---

## Resume Processing Pipeline

```
Upload → S3 Storage → Celery Task Queued
           ↓
     parse_resume.py       (PyMuPDF / pdfplumber / python-docx)
           ↓
     skill_extractor.py    (spaCy NER — skills, experience, education)
           ↓
     embed_resume.py        (Sentence Transformers → Pinecone upsert)
           ↓
     score_candidate.py     (LangChain chain → GPT-4 / Groq → score 0-100)
           ↓
     DB updated → Frontend polling resolves
```

Each step is an independent Celery task. Failures at any step are retried up to 3 times with exponential backoff.

---

## AI Scoring

Scoring is performed by `services/ai_scorer.py` using a LangChain chain:

1. Resume text + job description are combined into a structured prompt
2. GPT-4 (or Groq Llama 3) returns a JSON score object with:
   - Overall match score (0–100)
   - Skills match score
   - Experience relevance score
   - Education fit score
   - Short justification text
3. Scores are stored in the `scores` table and linked to both `candidate` and `job`

Semantic similarity (via Pinecone) provides a fast pre-filter before the LLM call, reducing token costs at scale.

---

## Database Schema Overview

| Table | Key Columns |
|---|---|
| `companies` | `id`, `name`, `plan`, `stripe_customer_id` |
| `users` | `id`, `company_id`, `email`, `role` |
| `jobs` | `id`, `company_id`, `title`, `description`, `status` |
| `resumes` | `id`, `company_id`, `s3_key`, `filename`, `parsed_text` |
| `candidates` | `id`, `resume_id`, `job_id`, `pipeline_stage` |
| `scores` | `id`, `candidate_id`, `job_id`, `overall`, `skills`, `experience` |
| `subscriptions` | `id`, `company_id`, `stripe_subscription_id`, `plan`, `status` |

Every table includes `created_at`, `updated_at` timestamps. All queries filter by `company_id` for tenant isolation.

---

## Multi-Tenancy & Security

- Every database query filters by `company_id` extracted from the verified JWT
- Row-level isolation enforced in SQLAlchemy query layer via `dependencies.py`
- RBAC roles checked on every protected endpoint via `core/security.py`
- File uploads are namespaced by `{company_id}/{job_id}/` in S3

---

## Running Tests

```bash
# Unit tests
pytest tests/unit -v

# Integration tests (requires running DB and Redis)
pytest tests/integration -v

# Full suite with coverage
pytest --cov=app --cov-report=html
```

---

## Database Migrations

```bash
# Create a new migration
alembic revision --autogenerate -m "add_pipeline_stage_to_candidates"

# Apply all pending migrations
alembic upgrade head

# Rollback one step
alembic downgrade -1
```

---

## Docker Compose Services

| Service | Port | Description |
|---|---|---|
| `api` | 8000 | FastAPI application |
| `worker` | — | Celery worker (no exposed port) |
| `db` | 5432 | PostgreSQL 15 |
| `redis` | 6379 | Redis (broker + result backend) |

```bash
# Start all services
docker compose up -d

# View logs
docker compose logs -f api
docker compose logs -f worker

# Rebuild after code changes
docker compose up --build api worker

# Stop everything
docker compose down
```

---

## Production Deployment

### Render

1. Create a **Web Service** — connect your GitHub repo, set build command to `pip install -r requirements.txt`, start command to `uvicorn app.main:app --host 0.0.0.0 --port $PORT`
2. Create a **Background Worker** service using `celery -A app.workers.celery_app worker --loglevel=info`
3. Add a **PostgreSQL** database and a **Redis** instance from the Render dashboard
4. Set all environment variables in Render's **Environment** tab

### AWS (ECS / App Runner)

Use `docker-compose.prod.yml` for ECS task definitions. Set `DATABASE_URL` to your RDS instance and `REDIS_URL` to your ElastiCache endpoint.

---

## Environment Variables Reference

| Variable | Required | Description |
|---|---|---|
| `DATABASE_URL` | Yes | PostgreSQL async connection string |
| `REDIS_URL` | Yes | Redis connection string |
| `SECRET_KEY` | Yes | App secret (min 32 chars) |
| `AUTH0_DOMAIN` | Yes* | Auth0 tenant domain |
| `CLERK_SECRET_KEY` | Yes* | Clerk secret key |
| `OPENAI_API_KEY` | Yes | OpenAI API key |
| `PINECONE_API_KEY` | Yes | Pinecone API key |
| `AWS_S3_BUCKET` | Yes | S3 bucket name |
| `STRIPE_SECRET_KEY` | Yes | Stripe secret key |
| `STRIPE_WEBHOOK_SECRET` | Yes | Stripe webhook signature |

*Use either Auth0 or Clerk, not both.

---

## Code Style

- `ruff` for linting and import sorting
- `black` for formatting
- `mypy` for type checking

```bash
ruff check app/
black app/
mypy app/
```

---

## Contributing

1. Fork and create a feature branch: `git checkout -b feat/your-feature`
2. Follow Conventional Commits: `feat:`, `fix:`, `chore:`
3. Ensure all tests pass: `pytest`
4. Open a pull request against `main`

---

## License

MIT — see [LICENSE](./LICENSE)
