# Contributing to HomeOps

## Branching strategy

HomeOps follows **Git Flow**. All work flows through `develop`; `main` only ever receives finished, CI-green releases.

```
main        ─────────────────────●────────────────────────●──▶
                                ↑ v0.1.0                  ↑ v0.2.0
                               merge                     merge
develop     ──────●────●────●──┴──●────●────●────●────●──┴──▶
                  │    │    │         │    │    │    │
feature/…   ──────┘    │    │         │    │    │    └──────▶
feat/…                 └────┘         │    │    └─────────▶
fix/…                                 └────┘
```

### Branches

| Branch | Purpose | Who creates it |
|---|---|---|
| `main` | Production-ready releases only | CI/CD (via release merge) |
| `develop` | Integration branch — all features land here first | Permanent |
| `feat/<short-name>` | New feature (mapped to a SESSIONS.md deliverable) | Developer |
| `fix/<short-name>` | Bug fix — branches from `develop` (or `main` for hotfixes) | Developer |
| `release/<version>` | Release preparation — version bump, changelog, final QA | Developer |
| `hotfix/<short-name>` | Urgent fix on `main` — merges back into both `main` and `develop` | Developer |

### Rules

- **Never commit directly to `main` or `develop`.** All changes come via pull request.
- **Feature branches start from `develop`**, not `main`.
- **CI must be green** before any PR is merged — no exceptions.
- **Squash-merge** feature and fix branches into `develop` to keep history linear.
- **Merge commit** (no squash) when merging `release/*` or `hotfix/*` into `main` so the release commit is visible in history.
- Branch names use lowercase kebab-case: `feat/engine-decision-rules`, `fix/coverage-threshold`.

### Typical session workflow

```bash
# 1. Start from a clean develop
git checkout develop && git pull origin develop

# 2. Create a feature branch for the session's deliverable
git checkout -b feat/engine-decision-rules

# 3. Work, commit often with conventional commit messages
git commit -m "feat(engine): add critical-battery rule"
git commit -m "test(engine): cover surplus-export branch"

# 4. Push and open a PR → develop (not main)
git push -u origin feat/engine-decision-rules
gh pr create --base develop --title "feat(engine): deterministic decision rules"

# 5. CI green → squash-merge PR → delete branch

# 6. When a session is complete and develop is stable, open a release PR
git checkout -b release/0.2.0 develop
# bump version, update SESSIONS.md, update CHANGELOG.md
gh pr create --base main --title "release: v0.2.0 — engine complete"
```

### Conventional commits

All commit messages follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <short description>

Types:  feat | fix | docs | test | refactor | chore | ci | perf | security
Scopes: engine | db | web | docker | ci | deps
```

Examples:
```
feat(engine): add time-of-use peak-hour rule
fix(db): handle null reviewedAt in recommendation query
test(engine): achieve 100% coverage on all rule branches
ci: add postgres service for db integration tests
docs: update local-setup with migrate step
security: bump drizzle-orm to patch CVE-xxxx-yyyy
```

### Pull request checklist

Before marking a PR ready for review:

- [ ] Branch is up to date with `develop` (or `main` for hotfixes)
- [ ] `pnpm typecheck` passes locally
- [ ] `pnpm lint` passes locally
- [ ] `pnpm test` passes locally (and coverage thresholds are met)
- [ ] PR title follows conventional commit format
- [ ] SESSIONS.md updated if a deliverable is completed

## First-time setup

```bash
git clone https://github.com/titusandronicu/homeops.git
cd homeops
git checkout develop
pnpm install
cp .env.example .env.local
docker compose -f docker/docker-compose.yml up -d
```

See [`docs/local-setup.md`](docs/local-setup.md) for full instructions.
