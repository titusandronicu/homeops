# HomeOps — Session Log

Each session ends with **CI green on `main`**: tests passing, coverage met, security scans clean.  
Nothing merges without a green pipeline. Each session is a shippable increment.

---

## CI/CD gates (apply to every session)

| Check | Tool | Threshold |
|---|---|---|
| Type checking | `tsc --noEmit` | zero errors |
| Linting | ESLint (flat config) | zero errors |
| Unit tests | Vitest | all pass |
| Coverage — engine | Vitest + v8 | **100%** (pure functions) |
| Coverage — db | Vitest + v8 | ≥ 80% |
| Coverage — web | Vitest + v8 | ≥ 70% |
| Secret scanning | Gitleaks | zero findings |
| Dependency CVEs | Trivy (`CRITICAL,HIGH`) | zero unacknowledged |
| Build | `astro build` | succeeds |

E2E (Playwright) is added in Session 5 and gates merges from that point on.

---

## Session 1 — Repo, CI/CD, branch protection ✅ (2026-08-26)

**Done when:** pipeline runs end-to-end (even on near-empty repo), branch protection active.

- [x] Public GitHub repo (`titusandronicu/homeops`)
- [x] Monorepo skeleton: `apps/web`, `packages/db`, `packages/engine`, `docker`, `docs`
- [x] `.gitignore`, `.gitleaks.toml`, `.env.example`
- [x] GitHub Actions — `ci.yml` (typecheck · lint · test · build)
- [x] GitHub Actions — `security.yml` (Gitleaks · Trivy)
- [x] Docker Compose skeleton (postgres + app stub)
- [x] `README.md`, `SESSIONS.md`, `docs/architecture.md`, `docs/local-setup.md`
- [x] Branch protection: PR required · CI must be green · no force-push to `main`

---

## Session 2 — `packages/engine` (deterministic decision rules)

**Done when:** engine is fully tested, coverage 100%, CI green.

Deliverables:
- `EnergySnapshot`, `RecommendationResult`, `ActionKind` types
- Decision rules: critical battery · surplus export · solar forecast · time-of-use peak
- `decide()` is a pure function — zero I/O, zero side effects
- Synthetic fixture data for repeatable dev/demo runs
- Vitest unit tests — every rule branch covered

CI gates added this session:
- `packages/engine` coverage **100%** enforced in `vitest.config.ts`

---

## Session 3 — `packages/db` (schema + migrations + queries)

**Done when:** schema migrates cleanly, query helpers tested, CI green.

Deliverables:
- `recommendations` table (Drizzle schema + migration file)
- `createDb()` factory with postgres.js connection pooling
- Query helpers: `insertRecommendation`, `listRecommendations`, `updateStatus`
- Vitest integration tests run against a **real postgres** (docker service in CI)
- Seed script with synthetic data

CI gates added this session:
- `packages/db` coverage ≥ 80% enforced
- Postgres service container added to `ci.yml`
- `db:migrate` runs as a CI step before tests

---

## Session 4 — `apps/web` core MVP (dashboard + recommendation flow)

**Done when:** full user flow works end-to-end with synthetic data, CI green.

Deliverables:
- Login page (password-protected, no OAuth complexity)
- Energy dashboard — synthetic snapshot display with live refresh
- "Generate recommendation" → engine → AI explanation → DB save
- Accept / Reject action with optimistic UI
- Recommendation history page (paginated)
- Claude API integration for plain-language explanation (structured prompt only)
- Astro Actions for all mutations

CI gates added this session:
- `apps/web` build must succeed (`astro build`)
- `apps/web` unit coverage ≥ 70%
- `ANTHROPIC_API_KEY` added as GitHub Actions secret (used in CI smoke test only)

---

## Session 5 — E2E tests + Home Assistant integration

**Done when:** Playwright tests cover core flow, HA adapter works behind a flag, CI green.

Deliverables:
- Playwright tests: login · generate recommendation · accept/reject · history
- Home Assistant REST/WebSocket adapter (read-only, allowlisted entities only)
- `USE_SYNTHETIC_DATA=true` flag keeps repo safe without HA credentials
- Graceful fallback when HA is unreachable
- No private entity IDs committed

CI gates added this session:
- Playwright E2E added to `ci.yml` (runs with synthetic data)
- HA adapter tests mocked — no real HA in CI

---

## Session 6 — Production hardening + UGREEN NAS deploy

**Done when:** production Docker image builds, deploys to NAS, health check passes, CI green.

Deliverables:
- Multi-stage `Dockerfile` for `apps/web` (build → slim runtime)
- `docker-compose.prod.yml` with resource limits and restart policies
- GitHub Actions `deploy.yml` — SSH + `docker compose pull && up -d` on merge to `main`
- `/healthz` endpoint
- Trivy image scan in CI (blocks on CRITICAL)
- UGREEN NAS deployment verified and documented in `docs/deployment.md`

CI gates added this session:
- Docker image build in CI (`docker build`)
- Trivy image scan (`trivy image`) added alongside filesystem scan
- Deploy workflow fires only after all other jobs pass

---

## MVP definition

Sessions 1–4 delivered and CI green = **MVP complete**.  
Sessions 5–6 = production-grade and portfolio-ready.
