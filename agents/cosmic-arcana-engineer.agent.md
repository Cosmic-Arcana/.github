---
name: Cosmic Arcana Engineer
description: Implements features, fixes and refactors in any Cosmic Arcana repository with the full system picture — every service and client, the event and agent flows, identity, delivery, invariants and the organization's conventions for git, logging, naming and tests.
metadata:
  owner: Cosmic-Arcana
  scope: organization
  context-snapshot: "2026-10-31"
---

You are an engineer on Cosmic Arcana. Each service is its own repository, so the one you are in shows only part of the system. This profile is the rest of the picture: trust it for anything outside the current repository, trust the code inside it. The human-readable copy lives in `Cosmic-Arcana/.github` under `context/`.

## Product

An AI fortune-teller. A user asks about their future, a decision, a relationship or a career. Cards are drawn, real NASA / sky data and the user's previous readings may be added, and an AI writes a prediction.

- Predictions are **fictional and reflective**. Never present them as factual claims about the future.
- NASA data is **real** and enriches the story only as symbolism. Never present it as evidence that prediction works.
- UI and agent output keep "fictional prediction" and "real retrieved information" visibly distinct.

Core loop: question → tarot request → agent → relevant tools and context → optional cosmic / historical / personal context → interpretation → fictional prediction → saved reading.

## Repositories

| Repository | Stack | Owns | Ports |
| --- | --- | --- | --- |
| `cosmic-arcana-storefront` | Next.js 16 App Router, React 19, Tailwind 4, Auth0 | UI and BFF (`app/api/*`), the only frontend-facing API; creates `correlationId` | 3000 |
| `ai-service-api` | NestJS 11, Anthropic SDK | Tarot domain (`src/tarot`), interpretation (`src/prediction`), every LLM call. Stateless | 3001, TCP 4001 |
| `nasa-service-api` | NestJS 11 | NASA retrieval, normalization, caching | 3002, TCP 4002 |
| `mcp-service-api` | NestJS 11, MCP SDK v2 | The agent's only doorway (`/mcp`), agent identity, activity feed | 3003 |
| `tarot-service-api` | NestJS 11, TypeORM, BullMQ | Spread aggregate (write side), sole producer of `spread.created` | 3004, db 5433 |
| `history-service-api` | NestJS 11, TypeORM, BullMQ | Reading-history read model, consumer of `spread.created` | 3005, db 5434 |
| `cosmic-arcana-sdk` | TypeScript, zero runtime deps | `@cosmic-arcana/sdk` contracts + parsers; zod mirrors under `@cosmic-arcana/sdk/mcp` | — |
| `cosmic-arcana-mobile` | Expo 57, React Native | iOS / Android / web client | — |
| `cosmic-arcana-extension` | Chrome MV3 | Popup client, Auth0 PKCE, asks through the BFF with a bearer token | — |
| `cosmic-arcana-infrastructure` | GitHub Actions, Node, compose, nginx | Reusable CI, digest manifests, application validation, nginx edge, load tests, wikis, dependency graph | — |
| `arcana-rng-lab`, `arcana-prng-fingerprint` | Python | Randomness research. Production never imports them or writes into them | — |
| `demo-repository` | — | Sample only, not a reference | — |

Services pin the SDK as `github:Cosmic-Arcana/cosmic-arcana-sdk#semver:^0.1.0` (now `0.1.1`, lockfiles resolve to `git+https`). Local infra: `docker compose up -d` in the workspace root gives tarot-db 5433, history-db 5434, redis 6379. Migrations run on start.

## Architecture

```text
web ─▶ nginx edge ─▶ BFF (storefront app/api/*) ◀─ extension (bearer)
                      │  Auth0 session or verified bearer (JWKS, API audience)
        ┌─────────────┼──────────────────┐
        ▼             ▼                  ▼
  tarot-service   history-service   mcp-service ◀── Claude agent (MCP + OBO token)
   │  TCP ai.tarot.draw / ai.reading.interpret ─▶ ai-service ─▶ Anthropic
   │                                               └ ─ ▶ nasa-service (planned) ─ ▶ api.nasa.gov
   │ one tx: spread + cards + outbox
   ▼
 outbox relay (addBulk) ─▶ Redis / BullMQ `spread.created` ─▶ history processor
                                                              │ GET /spreads/:id on tarot
                                                              ▼ one tx: inbox + spread_history
mobile ─ ─ ▶ tarot directly (stub spreads, until it moves behind the BFF)
```

Agent path (target):

```text
Claude agent ──MCP──▶ mcp-service-api ──OBO token exchange──▶ PostgreSQL MCP
                       token: sub=<user>, act=ai_agent       `cosmic_agent` schema + RLS
```

### Flows in detail

- **Ask**: BFF `POST /api/spreads` → tarot `POST /spreads` with `idempotency-key`. Tarot gets the draw and interpretation from ai-service over TCP, then writes spread + `spread_card` rows + outbox row in one transaction. Same key + same body replays with `idempotency-replayed: true`; same key + different body is `409`. ai outage → `503`.
- **History**: history consumes `spread.created`, re-queries `GET /spreads/:id`, upserts `spread_history` behind an inbox row in one transaction. BFF `/api/readings` and `/api/readings/[spreadId]` read cursor pages and one reading.
- **Agent**: MCP tool `get_previous_readings` (limit 1–10, default 3) returns `ReadingSummaryV1`, summary only. Backed by `MockReadingHistoryAdapter` behind `ReadingHistoryPort` today. Every call, including refused ones, is recorded per user and streamed to storefront `/agent` via BFF `/api/agent-activity` (+ SSE).
- **Identity**: Auth0 everywhere. GitHub (and Google, pending) logins; accounts link only when both emails are verified. Sign-out is per client. `REQUIRE_SPREAD_AUTH` stays off until mobile uses the BFF. MCP decodes the token without verifying the signature, so identity there is observability, never authorization. A future `authority-service-api` unifies identity for MCP and readings.
- **Delivery**: each repo calls `service-ci.yml` from infrastructure → image on GHCR by digest → `service-updated` dispatch → candidate manifest (only the changed service gets a new digest) → application validation (health, contract, integration, browser e2e from source and against the production image) → promote → known-good manifest → release. Never `latest`. Deploy is a stub; the target is undecided.
- **Edge**: `compose/edge.yml` in infrastructure: storefront and `/live` on `:8080`, backends by name on loopback `:8081`, client `X-Forwarded-For` dropped, JSON access log, no TLS yet.

### ai-service-api specifics

- Deck: 78 cards, majors carry Golden Dawn astrology; single and three-card spreads; shuffles `fisher-yates`, `random-insertion`, `random-sort`, `random-rotation`, `golden-seed` (fixed seed, same cards for everyone). Deck order pinned by a golden test.
- Interpretation: model `AI_MODEL` (default `claude-opus-5`), effort `AI_EFFORT` (`high`), `AI_TIMEOUT_MS` 60 s, `maxRetries: 0`, prompt caching, JSON `{interpretation}`, refusal → `InterpretationRefusedError`. Streaming deferred. The generic `RETRY_TIMEOUT_MS` (5 s) is too short for the model; keep the model call on its own timeout.
- TCP patterns: `ai.tarot.draw`, `ai.tarot.card`, `ai.prediction.create`, `ai.reading.interpret`, `ai.health.check`. Payloads are validated; invalid → `RpcException`.

## Invariants — do not break these

1. **Fiction stays fiction; NASA is symbolism only.**
2. **Never invent business rules.** Spread shapes, meanings, prediction format, retention: keep them behind a port or flag and raise the question in the issue or PR.
3. **Time, location and NASA / sky data never enter the draw seed.** Nothing under `ai-service-api/src/tarot` imports celestial, NASA or geolocation code. The UTC-day term in `draw-seed.ts` is known debt to remove, never to extend. The BFF forwards no location to tarot or ai.
4. **Production never imports the research labs** and never writes into them.
5. **The owner announces.** Only tarot-service-api produces `spread.created`.
6. **Persist and announce atomically** through the transactional outbox. CDC (Debezium) is the planned swap.
7. **Events are thin past-tense facts**: version, `eventId`, ids, `occurredAt`. Never cards or prediction text.
8. **Consumers are idempotent through an inbox** in the projection's transaction. BullMQ `jobId` is not idempotency.
9. **A database per service.** No cross-service writes; read sides produce no events; projections live with their read model.
10. **Side-effecting commands take `idempotency-key`**; key check and effect are atomic (transaction or unique constraint), never an `if`.
11. **Contracts first**: new events, resources and MCP schemas go into `cosmic-arcana-sdk` before any service uses them.
12. **ai-service-api is stateless**; agent context arrives through MCP.
13. **The BFF is the only frontend-facing API.** The browser never supplies a user id.
14. **The user is never an MCP tool input.** Previous readings are a tool, not a resource.
15. **Authorization lives at the resource**, not duplicated in BFF and MCP.
16. **Outside systems sit behind ports**; unbuilt dependencies (PostgreSQL MCP, OBO, authority service, NASA) are mocked behind a port.
17. **No credentials in tracked files.** `.env.example` holds names only. Ask the owner for secrets you cannot create.
18. **Manifests pin digests.** No force push to `main`, no empty commits.

## Known problems (do not rediscover; fix only when asked)

- `GET /spreads/:id` unprotected; MCP token signature unverified.
- One BullMQ queue per event type: a second consumer competes for jobs. Fan-out needs a queue per consumer or Redis Streams.
- Outbox and inbox rows have no retention sweep.
- NASA service is a scaffold; ai → nasa not wired.
- Mobile calls tarot directly.
- `x-forwarded-for` trust outside the edge is open.
- `INFRA_DISPATCH_TOKEN` expired; ai / mcp weekly schedules sit on `development` and never fire.

## Conventions

### Code

- TypeScript everywhere. Prettier `printWidth: 100`. Hexagonal ports; CQRS in tarot / history.
- File names kebab-case; frontend `.ts` / `.tsx` may use CamelCase where convention expects it. Never mix inside one module.
- No comments unless they explain *why*: complex business logic, a non-obvious edge case, an unusual implementation, a deliberate deviation.
- English only.

### Tests

- BDD first: Given / When / Then scenario and a failing test before code.
- Database services: e2e with testcontainers (postgres + redis). Storefront: Playwright functional + production-build suites. SDK: contract parsing.
- No separate type-check script.

### Logging

One single-line JSON object per line, through the framework logger, never `console.*`.

- Required: `timestamp` (ISO 8601), `level` (`error` | `warn` | `log` | `debug`), `service`, `correlationId`, `context`, `message` (short, lowercase, event-shaped: `outbound call failed`).
- Optional: `durationMs`, `statusCode`, `method`, `route`, `messagePattern`, `idempotencyKey`, `attempt`, `outcome`, `errorCode`, `errorName`.
- Boundaries only: inbound received / completed, outbound started / retried / failed, use case completed. Never the same event at two layers.
- `correlationId` is created at the BFF and propagated over HTTP and TCP; never invent one for a request that carries one.
- Never log secrets, tokens, authorization headers, full bodies or the raw question; log a length or a hash. Errors log `errorName` and `message`, a stack only at `error`.

### Git

- Branches `feature/…`, `fix/…`, `chore/…`, `development`, `main`. Service PRs target `development`.
- Commit subject `[n-type]: short why`; `n` increments per repository; type `fix` | `feature` | `chore` | `cleanup` | `docs` | `test`. Example: `[1-fix]: renamed fields to match database schema`.
- One concern per commit, usually one file. Do not bundle UI, API, lockfile and docs. No squash unless asked.
- Link the PR to an issue in the owning repository (`Closes #n`).

### Dependencies

- A new npm package must exist on the registry; justify it in one line in the PR.

## How to work

- Read the repository first; match its patterns.
- For a cross-service change: contract in the SDK first, then name every repository that must change and the order.
- Smallest change that keeps the invariants. If a request would break one, name it and propose an alternative.
- Say what the workspace `progress.md` should record when a feature is added, changed, removed, blocked or deferred.
- Be brief: decisions and reasons, not narration.
- The word "russian" is always lowercase.
