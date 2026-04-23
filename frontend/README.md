
# Resume Screening SaaS — Frontend

> AI-powered hiring platform built with Next.js 14, TypeScript, and Tailwind CSS. Multi-tenant, role-based, and production-ready.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript 5+ |
| Styling | Tailwind CSS |
| UI Components | Shadcn UI + Radix UI |
| State Management | Zustand |
| Data Fetching | React Query (TanStack) + Axios |
| Charts | Recharts |
| Auth | Auth0 / Clerk |
| Payments | Stripe.js |
| Deployment | Vercel |

---

## Prerequisites

- Node.js 20+
- npm 10+ or pnpm 9+
- Backend API running (see `resume-screening-backend`)

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/your-org/resume-screening-frontend.git
cd resume-screening-frontend
```

### 2. Install dependencies

```bash
npm install
# or
pnpm install
```

### 3. Configure environment variables

```bash
cp .env.example .env.local
```

Edit `.env.local`:

```env
# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_API_URL=http://localhost:8000/api/v1

# Auth0
NEXT_PUBLIC_AUTH0_DOMAIN=your-tenant.auth0.com
NEXT_PUBLIC_AUTH0_CLIENT_ID=your_client_id
AUTH0_CLIENT_SECRET=your_client_secret
AUTH0_SECRET=a-long-random-secret-string

# Clerk (alternative to Auth0 — use one)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/login
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard

# Stripe
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

### 4. Run development server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

---

## Project Structure

```
resume-screening-frontend/
├── app/
│   ├── (auth)/                   # Public auth pages
│   │   ├── login/page.tsx
│   │   └── register/page.tsx
│   ├── (dashboard)/              # Protected app pages
│   │   ├── layout.tsx            # Sidebar + navbar shell
│   │   ├── dashboard/page.tsx    # Analytics overview
│   │   ├── jobs/
│   │   │   ├── page.tsx          # Job listings
│   │   │   ├── new/page.tsx      # Create job
│   │   │   └── [id]/page.tsx     # Job detail + candidates
│   │   ├── candidates/
│   │   │   ├── page.tsx          # Ranked candidate list
│   │   │   └── [id]/page.tsx     # Candidate profile
│   │   ├── resumes/
│   │   │   ├── page.tsx
│   │   │   └── upload/page.tsx   # Bulk upload
│   │   ├── pipeline/page.tsx     # Interview kanban board
│   │   └── settings/
│   │       ├── page.tsx
│   │       ├── billing/page.tsx  # Stripe subscription
│   │       └── team/page.tsx     # RBAC + invites
│   ├── api/
│   │   ├── auth/[...nextauth]/route.ts
│   │   └── webhooks/stripe/route.ts
│   └── layout.tsx
├── components/
│   ├── ui/                       # Shadcn base components
│   ├── layout/                   # Sidebar, Navbar, PageWrapper
│   ├── dashboard/                # StatsCard, CandidateChart
│   ├── candidates/               # RankingTable, ScoreBar, SkillBadges
│   ├── jobs/                     # JobForm, JDEditor
│   ├── resumes/                  # DropZone, UploadProgress
│   └── pipeline/                 # KanbanBoard, PipelineCard
├── hooks/                        # useCandidates, useJobs, useAuth, useResumeUpload
├── lib/
│   ├── api.ts                    # Axios instance + auth interceptors
│   ├── auth.ts                   # Auth0 / Clerk helpers
│   ├── stripe.ts                 # Stripe client
│   └── utils.ts
├── store/                        # Zustand stores
├── types/                        # TypeScript interfaces
├── middleware.ts                 # Auth guard + RBAC route protection
├── next.config.ts
└── tailwind.config.ts
```

---

## Available Scripts

```bash
npm run dev          # Start dev server (http://localhost:3000)
npm run build        # Production build
npm run start        # Start production server
npm run lint         # ESLint
npm run type-check   # TypeScript check (tsc --noEmit)
npm run format       # Prettier format
```

---

## Authentication & RBAC

Authentication is handled via **Auth0** or **Clerk** (configure one in `.env.local`). Route protection is enforced in `middleware.ts` using Next.js middleware — unauthenticated requests are redirected to `/login` before any page component renders.

Role-based access is enforced at both the route and component level:

| Role | Access |
|---|---|
| `owner` | All features, billing, team management |
| `admin` | Jobs, candidates, pipeline, analytics |
| `recruiter` | View jobs, manage pipeline, export CSV |
| `viewer` | Read-only access to candidates and analytics |

Roles are included in the JWT claims and validated against `lib/permissions.ts`.

---

## Multi-Tenancy

Every authenticated user belongs to a **company (tenant)**. The `company_id` is embedded in the JWT and automatically appended to all API requests via the Axios interceptor in `lib/api.ts`. Users never see data from other tenants.

---

## Resume Upload Flow

1. User drops files on `DropZone.tsx` (PDF, DOC, DOCX — max 10MB each, up to 100 files)
2. Files are uploaded directly to the backend via multipart form
3. Backend queues a Celery task for each file
4. Frontend polls `/api/v1/resumes/status` using React Query for live progress updates
5. On completion, candidates appear in the ranked view

---

## Stripe Billing

Stripe is integrated for SaaS subscriptions. The webhook handler at `app/api/webhooks/stripe/route.ts` listens for:

- `checkout.session.completed` — activate subscription
- `invoice.payment_succeeded` — renew subscription
- `customer.subscription.deleted` — downgrade / cancel

Test locally with the Stripe CLI:

```bash
stripe listen --forward-to localhost:3000/api/webhooks/stripe
```

---

## Environment Variables Reference

| Variable | Required | Description |
|---|---|---|
| `NEXT_PUBLIC_API_URL` | Yes | Backend base URL |
| `NEXT_PUBLIC_AUTH0_DOMAIN` | Yes* | Auth0 tenant domain |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Yes* | Clerk publishable key |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Yes | Stripe public key |
| `STRIPE_SECRET_KEY` | Yes | Stripe secret key |
| `STRIPE_WEBHOOK_SECRET` | Yes | Stripe webhook signature |

*Use either Auth0 or Clerk, not both.

---

## Deployment (Vercel)

```bash
vercel --prod
```

Set all environment variables in the Vercel dashboard under **Settings → Environment Variables**. Enable the Vercel Edge Network for global CDN distribution.

Make sure `NEXT_PUBLIC_API_URL` points to your production backend URL.

---

## Code Style

- ESLint with `eslint-config-next` + `@typescript-eslint`
- Prettier for formatting
- Absolute imports via `@/` alias (configured in `tsconfig.json`)
- Components use named exports; pages use default exports

---

## Contributing

1. Fork the repo and create a feature branch: `git checkout -b feat/your-feature`
2. Commit using Conventional Commits: `feat:`, `fix:`, `chore:`
3. Open a pull request against `main`
4. Ensure `npm run lint` and `npm run type-check` pass before pushing

---

## License

MIT — see [LICENSE](./LICENSE)
