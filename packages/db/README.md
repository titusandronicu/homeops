# packages/db

PostgreSQL schema, Drizzle ORM, migrations, and query helpers.

- `createDb(connectionString)` — returns a typed Drizzle client
- `recommendations` table — the persistent decision history
- Integration tests run against a real postgres instance (docker in CI)
- Implemented in **Session 3**. See [`SESSIONS.md`](../../SESSIONS.md).
