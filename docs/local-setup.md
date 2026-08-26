# Local Setup

## Prerequisites

- Node.js 20+
- pnpm 9+ (`npm i -g pnpm`)
- Docker + Docker Compose

## 1. Clone and install

```bash
git clone https://github.com/titusandronicu/homeops.git
cd homeops
pnpm install
```

## 2. Environment

```bash
cp .env.example .env.local
# Edit .env.local — at minimum set SESSION_SECRET
```

For synthetic data (default, no Home Assistant needed) — leave `USE_SYNTHETIC_DATA=true`.

## 3. Start postgres

```bash
docker compose -f docker/docker-compose.yml up -d
```

## 4. Run migrations

```bash
pnpm db:migrate
```

## 5. Start the app

```bash
pnpm dev
# → http://localhost:4321
```

## Required GitHub Actions secrets

Set these in **Settings → Secrets and variables → Actions**:

| Secret | When needed | Notes |
|---|---|---|
| `ANTHROPIC_API_KEY` | Session 4+ | Claude API for explanations |
| `DEPLOY_HOST` | Session 6 | UGREEN NAS IP / hostname |
| `DEPLOY_USER` | Session 6 | SSH user for deploy |
| `DEPLOY_SSH_KEY` | Session 6 | Private key (Ed25519 recommended) |

## Running CI checks locally

```bash
pnpm typecheck   # type check all packages
pnpm lint        # ESLint
pnpm test        # Vitest unit tests
pnpm build       # full build
```
