# Queue SaaS

Real-time queue management for hospitals and clinics in India. Modular monolith: NestJS on Fastify, PostgreSQL (source of truth), Redis (infrastructure only), Socket.IO (delivery only).

Status: **Phase 2 (database)** on top of the Phase 1 foundation. No business logic yet. Schema notes: `docs/DATABASE.md`. See `docs/READINESS.md` for what is proven and what is not.

## Prerequisites
Node 22.12+ (built with 22.22.2), pnpm 12.10.1, PostgreSQL 16 and Redis 7 (or Docker for `docker-compose.yml`, NOT VERIFIED).

## Quick start
```bash
pnpm install --frozen-lockfile
bash scripts/dev-services.sh start        # PostgreSQL :55432, Redis :56379 (fake dev passwords)
export DATABASE_URL=postgres://qs_admin:qs_admin_dev_pw@127.0.0.1:55432/qs_dev
pnpm db:migrate
REDIS_URL=redis://127.0.0.1:56379 REDIS_USERNAME=ops REDIS_PASSWORD=devpass-ops pnpm --filter @qs/api start
```
`APP_ROLE` is `api`, `realtime`, `worker` or `all` (default). `GET /health` makes no dependency calls. `GET /ready` checks the database, pending migrations and Redis (Redis is required only for `realtime`).

## Scripts
| Command | What it does |
|---|---|
| `pnpm typecheck` / `lint` / `format:check` | Static checks |
| `pnpm test` | Unit tests (no services needed) |
| `pnpm test:integration` | Needs the dev services. Set `QS_EVIDENCE_DIR` to save the Redis ACL result |
| `pnpm db:migrate` / `db:status` | Forward-only SQL migrations |
| `pnpm build` | Compile all packages |

## Layout
`apps/api` (NestJS), `apps/web` (placeholder), `packages/shared` (error catalogue, money, request IDs), `db/migrations`, `infrastructure/{docker,redis}`, `scripts`, `docs`.

## Deviations from the design (recorded on purpose)
- `packages/config` is folded into the root configs until a second consumer exists.
- `apps/web` is a placeholder. Next.js arrives in Phase 8.
- OpenTelemetry is a skeleton only. No exporter is wired.
- Rate limiting is deferred (planned with a Redis store).
- pnpm overrides: `fastify` (one copy, so plugin types agree) and `cookie` 1.1.1 (Jest cannot load the ESM-only 2.x that Fastify's test `inject` now needs).
- Sentry 11 has no `sendDefaultPii`. Every `dataCollection` category is switched off instead.
- A no-transaction migration must be a single statement (a multi-statement query runs in an implicit transaction, which `CREATE INDEX CONCURRENTLY` rejects).
