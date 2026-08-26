# HomeOps

**AI-native home energy control plane** — deterministic decisions, AI explanations, human approval, self-hosted.

[![CI](https://github.com/titusandronicu/homeops/actions/workflows/ci.yml/badge.svg)](https://github.com/titusandronicu/homeops/actions/workflows/ci.yml)
[![Security](https://github.com/titusandronicu/homeops/actions/workflows/security.yml/badge.svg)](https://github.com/titusandronicu/homeops/actions/workflows/security.yml)

---

## What is this?

HomeOps is a self-hosted control plane for home energy management. It collects real-time energy signals, applies deterministic decision rules to produce a recommendation, asks an LLM to explain that recommendation in plain language, and then waits for a human to approve or reject it before anything happens.

Nothing in the home is ever changed automatically. The human is always in the loop.

---

## The problem

Modern homes generate a lot of energy data — solar production, battery state, grid import/export, consumption — but very little of it is actionable. Inverter apps show charts. Home automation platforms collect sensors. But translating that data into a clear, confident decision ("charge the battery now", "export surplus to the grid") still requires manual attention and domain knowledge.

HomeOps closes that gap by doing three things that are rarely combined:

1. **Structured reasoning** — A deterministic rule engine evaluates the energy snapshot and produces a typed result: action, confidence score, and machine-readable reasons.
2. **Human-readable explanation** — A language model translates that structured result into a short, plain-language explanation. It has no access to raw data and no write path to anything.
3. **Persistent history** — Every recommendation, whether accepted or rejected, is stored. Patterns emerge over time.

---

## Design goals

### Correctness over cleverness
The decision engine is pure TypeScript — no runtime dependencies, no I/O, no side effects. Given the same inputs it always produces the same output. It is fully unit-tested with 100% coverage enforced by CI. The LLM cannot override or modify the decision; it can only describe it.

### Explainability
Every recommendation is accompanied by the specific reasons that triggered it. Users can read exactly why the system suggested what it did. Accepting or rejecting a recommendation is an informed choice, not an act of faith.

### Minimal trust surface
The integration with Home Assistant is read-only and restricted to an explicit allowlist of energy-related entities. No cameras, no people sensors, no location data, no locks, no alarms — none of that enters this system. The AI receives only structured, pre-filtered data.

### Self-hosted and portable
HomeOps runs on Docker Compose. It requires no cloud subscription, no vendor lock-in, and no ongoing fees beyond the LLM API. The entire system can run on a NAS, a small server, or a Raspberry Pi with enough RAM.

### Safe by default
The public repository ships with synthetic data. Real home integration is opt-in via a local `.env` file that is never committed. Anyone can clone this repo, run it, and see the full system working without exposing any real infrastructure.

### Portfolio-quality engineering
This project is also a demonstration of how to build a small but complete production system responsibly: typed end-to-end, tested at every layer, security-scanned on every commit, deployed through a proper CI/CD pipeline, and documented so that someone else could run it.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     apps/web  (Astro SSR)                   │
│   Login → Dashboard → Generate → Explain → Review → Log    │
└────────────────┬──────────────────────┬────────────────────┘
                 │                      │
    ┌────────────▼───────┐   ┌──────────▼──────────┐
    │  packages/engine   │   │   packages/db        │
    │  Deterministic     │   │   Drizzle + Postgres  │
    │  decision rules    │   │   Recommendation log  │
    │  100% unit tested  │   │   + query helpers     │
    └────────────────────┘   └─────────────────────┘
                 ▲
    ┌────────────┴───────────────────┐
    │  Home Assistant (optional)     │
    │  Read-only · allowlisted only  │
    │  Graceful fallback to synth.   │
    └────────────────────────────────┘
```

**Data flow:**
1. Energy snapshot collected (synthetic or Home Assistant REST)
2. `decide(snapshot)` → typed `RecommendationResult` (engine)
3. `RecommendationResult` → Claude API → plain-language explanation
4. Recommendation + explanation saved to PostgreSQL
5. User reviews on dashboard → Accept or Reject
6. Status updated in DB — history page reflects the decision

---

## Stack

| Layer | Technology |
|---|---|
| Frontend / SSR | Astro 5, TypeScript, Tailwind CSS |
| Decision engine | Pure TypeScript — zero runtime deps |
| Database | PostgreSQL + Drizzle ORM |
| AI explanation | Claude API (structured input, read-only) |
| Unit tests | Vitest (100% engine coverage enforced) |
| E2E tests | Playwright |
| CI/CD | GitHub Actions |
| Secret scanning | Gitleaks |
| CVE scanning | Trivy |
| Deployment | Docker Compose |

---

## Safety and privacy

**Public repo safety:** The repository is safe to clone and run. Synthetic data is used by default. No real infrastructure details, credentials, entity IDs, hostnames, or network topology are committed anywhere in this repo.

**Home Assistant integration:** Enabled only through a private local `.env` file. Restricted to an explicit allowlist of energy entities. No cameras, people trackers, locks, alarms, or location sensors.

**AI boundary:** The language model receives only the structured `RecommendationResult` — a typed object with an action, confidence score, and reasons. It never sees raw sensor data, user identity, or any network information.

See [`SECURITY.md`](SECURITY.md) for the full responsible disclosure and privacy policy.

---

## Running locally

```bash
git clone https://github.com/titusandronicu/homeops.git
cd homeops
cp .env.example .env.local   # fill in SESSION_SECRET at minimum
docker compose -f docker/docker-compose.yml up -d
pnpm install
pnpm db:migrate
pnpm dev                     # → http://localhost:4321
```

See [`docs/local-setup.md`](docs/local-setup.md) for the full guide including required GitHub Actions secrets.

---

## Project status

Implemented in focused sessions — each one ends with CI green. See [`SESSIONS.md`](SESSIONS.md) for what is done and what comes next.

**MVP** = Sessions 1–4 complete and CI green.  
**Production-ready** = Sessions 1–6 complete.

---

## License

MIT
