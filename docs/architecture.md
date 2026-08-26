# Architecture

## Guiding principles

1. **Deterministic core** — `packages/engine` is a pure function. Given the same `EnergySnapshot`, it always produces the same `RecommendationResult`. No LLM in the decision path.
2. **AI explains, never controls** — Claude converts the structured result into a plain-language explanation. It has no write path to anything.
3. **Human in the loop** — every recommendation requires explicit Accept or Reject before it is acted on. The app never auto-applies.
4. **Safe by default** — `USE_SYNTHETIC_DATA=true` is the default. Real Home Assistant integration requires explicit opt-in via private local config.
5. **Testable at every layer** — pure engine (100% unit coverage), DB helpers (integration tests against real postgres), web (unit + Playwright E2E).

## Package map

```
homeops/
├── apps/
│   └── web/              Astro 5 SSR — UI, Astro Actions, session auth
│
├── packages/
│   ├── engine/           Pure TS decision rules — zero runtime deps, 100% coverage
│   └── db/               Drizzle ORM schema, migrations, query helpers
│
├── docker/
│   └── docker-compose.yml   Local dev: postgres
│
└── .github/
    └── workflows/
        ├── ci.yml            typecheck · lint · test+coverage · build
        └── security.yml      Gitleaks · Trivy (fs + image)
```

## Data flow

```
EnergySnapshot (HA or synthetic)
        │
        ▼
  packages/engine
  decide(snapshot) → RecommendationResult
        │
        ▼
  apps/web (Astro Action)
  ├── Save to DB (packages/db)
  ├── Call Claude API → plain-language explanation
  └── Return to UI for user review
        │
        ▼
  User: Accept | Reject
        │
        ▼
  DB status updated — history page
```

## Security boundaries

| What | Included | Excluded |
|---|---|---|
| HA entities | Energy sensors (allowlist) | Cameras, people, locks, alarms, location |
| Secrets | `.env.local` only | Never committed, never logged |
| AI access | Read structured result | No direct home control, no raw HA data |
| Write path | DB recommendation status | Nothing to inverter, nothing to HA |

## CI/CD gates

Every PR and push to `main` must pass:
- Type check (zero errors)
- Lint (zero errors)
- Tests (all pass, coverage thresholds met — see `SESSIONS.md`)
- Gitleaks (zero secrets)
- Trivy (zero unacknowledged CRITICAL/HIGH CVEs)
- Build (clean `astro build`)
