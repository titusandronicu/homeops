# HomeOps

**An AI-native control plane for home energy management.**

[![CI](https://github.com/titusandronicu/homeops/actions/workflows/ci.yml/badge.svg)](https://github.com/titusandronicu/homeops/actions/workflows/ci.yml)
[![Security](https://github.com/titusandronicu/homeops/actions/workflows/security.yml/badge.svg)](https://github.com/titusandronicu/homeops/actions/workflows/security.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

HomeOps sits between your energy data and your decisions. It watches battery state, solar production, household consumption, grid flow, and weather forecasts — then tells you, in plain language, what to do next and why. You approve. It logs. You learn.

> **The rule engine always decides. The AI only explains. You always approve.**

---

## The problem

A modern home with solar panels and a battery generates a constant stream of energy data. Inverter apps show it. Home automation platforms collect it. But between *data* and *action* there is a gap that no tool fills well:

- **When should I charge the battery from the grid?** Before a cloudy stretch? During off-peak tariff hours? When the forecast shows a weak solar day?
- **When is it worth exporting surplus?** When the battery is full and production exceeds consumption — but by how much, and for how long?
- **Why did the system recommend what it did?** And was I right to accept or reject it last time?

These decisions require combining multiple signals, applying domain knowledge, and making a call under uncertainty. Doing it manually every day is exhausting. Delegating it blindly to an algorithm you cannot inspect is uncomfortable.

HomeOps is the middle path: structured, explainable, auditable recommendations that a human reviews before anything happens.

---

## The approach

Most "smart home AI" projects make one of two mistakes:

1. **Pure automation** — an opaque algorithm that makes changes without explanation or approval. Fast, but untrustworthy.
2. **Pure dashboards** — beautiful charts that still leave all the thinking to the user. Visible, but not actionable.

HomeOps is built on a different architecture:

```
Deterministic rules → structured result → AI explanation → human decision
```

Each layer has a single, well-defined job:

| Layer | Responsibility | Why it matters |
|---|---|---|
| **Rule engine** | Evaluate signals, produce a typed decision | Pure function — testable, auditable, deterministic |
| **AI (Claude)** | Translate the decision into plain language | Explains the *why* without inventing it |
| **Human** | Accept or reject before anything happens | Trust is earned, not assumed |
| **Database** | Persist every recommendation and outcome | Patterns emerge; you learn your own system |

The engine never calls an API. The AI never writes to anything. The human is never bypassed.

---

## Design principles

### 1. Correctness over cleverness
The decision engine is a pure TypeScript function: `decide(snapshot) → result`. Same input, same output, every time. No randomness, no network calls, no hidden state. It is fully unit-tested with 100% branch coverage enforced by CI. You can read every rule in one file and understand exactly why a recommendation was made.

### 2. Explainability as a first-class requirement
Every recommendation includes the specific reasons that triggered it — machine-readable strings produced by the rule that fired. The AI receives those reasons and translates them into a paragraph. The user always sees both. Accepting a recommendation is an informed choice, not an act of faith.

### 3. Minimal trust surface
The integration with Home Assistant is read-only and restricted to an explicit allowlist of energy-related entity IDs. No cameras. No people sensors. No location data. No locks. No alarms. None of that enters this system. The AI receives a small, typed object — action, confidence, reasons — and nothing else.

### 4. Human in the loop, always
No recommendation is acted on automatically. The app surfaces a decision, waits for explicit approval or rejection, and records the outcome. The history page is not just a log — it is the feedback loop through which the system proves its value over time.

### 5. Self-hosted and portable
HomeOps runs on Docker Compose. No cloud subscription. No vendor lock-in. No ongoing platform fee beyond the LLM API call. The target deployment is a home NAS or small server — but it runs anywhere Docker does.

### 6. Safe by default, private by design
The public repository ships with synthetic data. Real home integration is opt-in, configured entirely through a local environment file that is never committed. The repo contains no IP addresses, no hostnames, no entity IDs, and no personal data. Anyone can clone it, run it, and see the full system working without exposing any real infrastructure.

---

## Architecture

### High-level

```
┌─────────────────────────────────────────────────────────────────┐
│                       Browser / UI                              │
│                    Astro 5 · SSR · Tailwind                    │
│                                                                 │
│   [Login] → [Dashboard] → [Generate] → [Review] → [History]   │
└───────────────────────┬─────────────────────┬───────────────────┘
                        │ Astro Actions        │ Drizzle queries
               ┌────────▼────────┐   ┌────────▼────────┐
               │ packages/engine │   │  packages/db     │
               │                 │   │                  │
               │  decide()       │   │  PostgreSQL       │
               │  Pure function  │   │  Recommendations  │
               │  100% coverage  │   │  + history        │
               └────────┬────────┘   └─────────────────┘
                        │ EnergySnapshot
               ┌────────▼──────────────────────┐
               │  Data source                  │
               │                               │
               │  Synthetic (default)          │
               │  Home Assistant REST (opt-in) │
               └───────────────────────────────┘
                                 │
                        ┌────────▼────────┐
                        │   Claude API    │
                        │                 │
                        │  Input:         │
                        │  Typed result   │
                        │                 │
                        │  Output:        │
                        │  Plain-language │
                        │  explanation    │
                        └─────────────────┘
```

### Request flow for a recommendation

```
1.  User clicks "Generate recommendation"
        │
2.  Astro Action collects EnergySnapshot
    (synthetic fixture or Home Assistant REST call)
        │
3.  decide(snapshot) → RecommendationResult
    { action, confidence, reasons[], snapshot, decidedAt }
        │
4.  RecommendationResult → Claude API
    Structured prompt · read-only · no raw sensor data
    ← plain-language explanation string
        │
5.  { result, explanation } saved to PostgreSQL
        │
6.  UI: recommended action + explanation + Accept / Reject
        │
7.  User decides → DB status updated → audit trail complete
```

### Monorepo structure

```
homeops/
├── apps/
│   └── web/                  Astro 5 SSR application
│       ├── src/pages/        Route handlers and UI pages
│       ├── src/lib/          Server-side logic and HA adapter
│       └── src/components/   Reusable UI components
│
├── packages/
│   ├── engine/               Deterministic decision engine
│   │   ├── src/types.ts      EnergySnapshot, RecommendationResult
│   │   ├── src/rules.ts      Rule implementations (pure functions)
│   │   └── src/rules.test.ts 100% branch coverage, enforced by CI
│   │
│   └── db/                   Database layer
│       ├── src/schema/       Drizzle table definitions
│       ├── src/queries/      Typed query helpers
│       └── migrations/       SQL migration files (version-controlled)
│
├── docker/
│   ├── docker-compose.yml       Local dev — postgres
│   └── docker-compose.prod.yml  Production — postgres + app
│
├── docs/
│   ├── architecture.md       Deeper architectural notes
│   └── local-setup.md        Full setup guide with secrets reference
│
└── .github/
    └── workflows/
        ├── ci.yml            Typecheck · lint · test+coverage · build
        └── security.yml      Gitleaks · Trivy (fs + image, weekly)
```

---

## Technology choices

| Concern | Choice | Rationale |
|---|---|---|
| **Framework** | [Astro 5](https://astro.build) SSR | Zero client JS by default; server actions without a separate API layer; TypeScript-first |
| **Styling** | [Tailwind CSS](https://tailwindcss.com) | Utility-first; consistent design tokens; zero unused CSS in production |
| **Decision engine** | Pure TypeScript | No framework needed for a pure function; maximum portability and testability |
| **Database** | PostgreSQL | Battle-tested; excellent JSON column support for snapshot storage |
| **ORM / migrations** | [Drizzle ORM](https://orm.drizzle.team) | Type-safe queries without magic; schema-first with explicit, version-controlled migrations |
| **AI explanation** | [Claude API](https://anthropic.com) | Best-in-class instruction following for constrained, structured-input tasks |
| **Unit testing** | [Vitest](https://vitest.dev) | Native ES modules; TypeScript-first; fast; built-in coverage with v8 |
| **E2E testing** | [Playwright](https://playwright.dev) | Cross-browser; reliable selectors; first-class TypeScript support |
| **Secret scanning** | [Gitleaks](https://github.com/gitleaks/gitleaks) | Scans every commit; blocks merges with leaked credentials; SARIF output |
| **CVE scanning** | [Trivy](https://aquasecurity.github.io/trivy) | Filesystem + container image scanning; weekly scheduled runs |
| **CI/CD** | GitHub Actions | Native integration; SARIF upload to Security tab; matrix-ready |
| **Deployment** | Docker Compose | Reproducible environments; runs locally, on a NAS, or on a VPS without modification |

---

## CI/CD pipeline

Every pull request and push to `main` or `develop` runs the full pipeline. **Nothing merges on red.**

```
Push / PR
    │
    ├── Typecheck & Lint  ──── tsc --noEmit  ·  ESLint strict flat config
    │
    ├── Unit Tests  ─────────── Vitest with coverage thresholds per package
    │   ├── packages/engine     100% branch coverage (pure functions — no excuse)
    │   ├── packages/db         ≥ 80%  (integration tests against real postgres in CI)
    │   └── apps/web            ≥ 70%
    │
    ├── Build  ──────────────── astro build (clean output required)
    │
    ├── Gitleaks  ───────────── Full git history scanned for committed secrets
    │
    └── Trivy  ──────────────── CRITICAL/HIGH CVEs in deps and container image
                                 SARIF results uploaded to GitHub Security tab
```

Gitleaks and Trivy also run on a **weekly schedule** to catch newly published CVEs against unchanged code.

Branching follows **Git Flow**: features and fixes target `develop`; only release and hotfix branches target `main`. See [`CONTRIBUTING.md`](CONTRIBUTING.md) for the complete workflow with examples.

---

## The bigger picture

HomeOps starts as a home energy tool, but the underlying pattern is more general:

**Structured signals → deterministic rules → AI explanation → human approval → persistent audit trail**

This is a template for building trustworthy AI-assisted tooling for any domain where:
- Decisions have real-world consequences
- The reasoning behind a decision matters as much as the decision itself
- An audit trail has value
- An operator wants to remain in control

**Near-term roadmap for HomeOps itself:**

- **Time-of-use tariff awareness** — incorporate electricity pricing so the engine knows when grid import is cheap versus expensive
- **Multi-day planning** — combine multi-day solar forecasts with tariff schedules to plan charge/discharge cycles ahead of time
- **Anomaly detection** — flag when consumption or production deviates significantly from the established pattern
- **Push notifications** — surface high-priority recommendations via Telegram or ntfy without requiring a dashboard visit
- **Homelab operations** — apply the same architecture to infrastructure decisions: backup scheduling, service scaling, incident detection

The energy use case is the starting point. The architecture is the point.

---

## Safety and privacy

**This repository is safe to run as-is.** Synthetic data is used by default. No real infrastructure details, credentials, entity IDs, hostnames, or personal data appear anywhere in this repo.

Real home integration requires a private `.env.local` that is never committed. The application enforces an entity allowlist server-side. The AI receives only a small typed result object — never raw sensor data, never user identity, never network information.

See [`SECURITY.md`](SECURITY.md) for the full responsible disclosure policy and a detailed description of the privacy boundaries.

---

## Running locally

```bash
git clone https://github.com/titusandronicu/homeops.git
cd homeops
pnpm install

cp .env.example .env.local
# Set SESSION_SECRET — leave USE_SYNTHETIC_DATA=true to run without Home Assistant

docker compose -f docker/docker-compose.yml up -d
pnpm db:migrate
pnpm dev
# → http://localhost:4321
```

See [`docs/local-setup.md`](docs/local-setup.md) for the complete guide.

---

## Project status

| Session | Scope | Status |
|---|---|---|
| 1 — Repo & CI/CD | Skeleton, GitHub Actions, branch protection, all docs | ✅ Complete |
| 2 — Decision engine | Pure TS rules, 100% Vitest coverage | Planned |
| 3 — Database layer | Drizzle schema, migrations, query helpers | Planned |
| 4 — Web MVP | Dashboard, recommendation flow, Claude integration | Planned |
| 5 — HA integration | Real sensor data, E2E Playwright tests | Planned |
| 6 — Production | Docker image, NAS deployment, deploy pipeline | Planned |

**MVP = Sessions 1–4 complete with CI green throughout.**

See [`SESSIONS.md`](SESSIONS.md) for the full plan with per-session CI gates and deliverables.

---

## License

MIT — see [LICENSE](LICENSE).
