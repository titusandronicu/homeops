# HomeOps

**AI-native home energy control plane** — deterministic decisions, AI explanations, self-hosted.

HomeOps turns six energy signals (battery SOC, PV production, household consumption, grid import/export, and solar forecast) into clear, explainable recommendations. Users review, accept, or reject each one. A persistent history builds over time.

> The decision is always made by deterministic, testable rules.  
> The LLM only explains — it never controls the home.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        apps/web  (Astro SSR)                │
│   Login → Dashboard → Generate → Review → History          │
└────────────────┬──────────────────────┬────────────────────┘
                 │                      │
    ┌────────────▼───────┐   ┌──────────▼──────────┐
    │  packages/engine   │   │   packages/db        │
    │  Deterministic     │   │   Drizzle + Postgres  │
    │  decision rules    │   │   Recommendation log  │
    └────────────────────┘   └─────────────────────┘
                 │
    ┌────────────▼───────────────────┐
    │  Home Assistant (optional)     │
    │  REST/WebSocket — read-only    │
    │  Allowlisted energy entities   │
    └────────────────────────────────┘
```

## Stack

| Layer | Technology |
|---|---|
| Frontend / SSR | Astro 5, TypeScript, Tailwind CSS |
| Decision engine | Pure TypeScript, no runtime deps |
| Database | PostgreSQL + Drizzle ORM |
| AI explanation | Claude API (structured output only) |
| Testing | Vitest (unit), Playwright (e2e) |
| CI/CD | GitHub Actions |
| Security scanning | Gitleaks, Trivy |
| Deployment | Docker Compose, UGREEN NAS |

## Safety boundary

The public repo is safe to run with **synthetic data by default**.

Real Home Assistant integration requires a private `.env.local` with explicitly allowlisted entity IDs. Cameras, people, trackers, locks, alarms, locations, and credentials never enter the project.

## Running locally

```bash
cp .env.example .env.local   # fill in your values
docker compose up -d         # postgres + app
```

See [`docs/local-setup.md`](docs/local-setup.md) for the full guide.

## Sessions

Implementation is spread across focused sessions. See [`SESSIONS.md`](SESSIONS.md) for what has been done and what comes next.

## License

MIT
