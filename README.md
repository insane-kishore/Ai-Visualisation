# ChartSense

Upload spreadsheets, get a data-quality report, and receive the best charts — with AI explanations that never invent numbers.

```
/web                 Next.js (App Router, TS, Tailwind, shadcn/ui)
/analysis            FastAPI analysis service (pandas, scipy, scikit-learn)
docker-compose.yml   postgres · redis · minio · analysis · web
```

## Prerequisites

- Node.js 20+ and npm
- Python 3.11
- Docker Desktop (for Postgres, Redis, MinIO — or the whole stack)

## 1. Configure environment

```bash
cp .env.example .env
cp .env.example web/.env.local
```

Fill in at least:

| Variable | How |
|---|---|
| `AUTH_SECRET` | `npx auth secret` or `openssl rand -base64 32` |
| `ANALYSIS_SERVICE_TOKEN` | `openssl rand -hex 32` — same value for web and analysis |
| `ANTHROPIC_API_KEY` | from console.anthropic.com (server-side only) |
| `GOOGLE_CLIENT_ID/SECRET` | optional, Google Cloud Console → OAuth client, redirect `http://localhost:3000/api/auth/callback/google` |

## 2a. Run everything in Docker

```bash
docker compose up --build
```

- Web: http://localhost:3000
- Analysis: http://localhost:8000 (requires `X-Service-Token` header)
- MinIO console: http://localhost:9001 (minioadmin / minioadmin); the `chartsense` bucket is created automatically.

## 2b. Local development (infra in Docker, apps on host)

```bash
docker compose up -d postgres redis minio minio-init
```

Analysis service:

```bash
cd analysis
python -m venv .venv
.venv\Scripts\activate          # Windows  (macOS/Linux: source .venv/bin/activate)
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

Web app:

```bash
cd web
npm install            # also runs `prisma generate`
npm run db:deploy      # apply migrations (use `npm run db:migrate` when changing the schema)
npm run db:seed        # demo user: demo@chartsense.dev / ChartSense-demo-2026
npm run dev
```

## Database

Prisma 7 with the `pg` driver adapter. Schema: `web/prisma/schema.prisma`; connection URL is read in
`web/prisma.config.ts` from `.env.local` / `.env`. The generated client lives in `web/lib/generated`
(git-ignored) and is imported only via `web/lib/db.ts`.

The initial migration adds hand-written CHECK constraints (score 0–100, non-negative counts,
`rowIndexes` capped at 100). Override the seed password with `SEED_DEMO_PASSWORD`.

In Docker, apply migrations once the stack is up:

```bash
docker compose exec web npx prisma migrate deploy
```

## Design system & 3D

- Tokens, glass utilities (`glass`, `glass-strong`, `gradient-border`, `glow-*`, `noise-overlay`,
  `text-gradient`, `shadow-depth-1…4`) live in `web/app/globals.css`; motion tokens in
  `web/lib/motion/tokens.ts`. The 8-colour chart palette is CVD-validated for both themes.
- **Never create a `<Canvas>`.** One global canvas is mounted by `Scene3DProvider`; put 3D in a
  `<View3D poster="/posters/…">` using the lazy components from `@/components/3d`.
- Playground (dev only): http://localhost:3000/dev/3d — tier switcher (HIGH/MEDIUM/LOW/OFF),
  FPS overlay, every 3D and CSS-3D component.
- Posters: with a server running, `npm run posters` renders `public/posters/*.png` using your
  installed Edge/Chrome (set `POSTER_BASE_URL` / `POSTER_BROWSER` to override).
- 3D budget: with a production server running, `npm run check:3d-budget -- /` fails if a page's
  lazy 3D JS exceeds 250 KB gzipped or if three.js leaks into first-load JS
  (`BUDGET_BASE_URL`, `BUDGET_TIER=HIGH|MEDIUM|LOW`). three.js + r3f alone are ~228 KB, so
  every page has ~20 KB for its scenes.

## Quality checks

```bash
# web
npm run lint && npm run typecheck && npm run format:check
# analysis
pytest
```

## Service authentication

The analysis service rejects every request (including docs, which are disabled) unless it carries
`X-Service-Token: $ANALYSIS_SERVICE_TOKEN`. It is only ever called server-side by the web app.

```bash
curl -H "X-Service-Token: $ANALYSIS_SERVICE_TOKEN" http://localhost:8000/health
```
