# Status

Snapshot as of **2026-10-31**. Trust the code over this page when they disagree, and update it
when a feature lands, changes, is blocked or deferred.

## Built

| Area | State |
| --- | --- |
| Spreads write side | `POST /spreads` with idempotency, `GET /spreads/:id`, outbox relay with `addBulk` (0.4 s max lag at 100k events) |
| Draw and interpretation | tarot calls ai-service over TCP: `ai.tarot.draw` (78-card deck, deterministic seed, selectable shuffles) and `ai.reading.interpret` (Anthropic, `claude-opus-5`, effort `high`, 60 s timeout, no SDK retry, prompt cache, refusal error). Tests mock the SDK |
| History read side | inbox-deduped projection, parallel processing (~1,000 events/s), cursor pages, single reading |
| Storefront | Ask, readings list and detail, `/agent` live activity, `/watch`, login, privacy, terms; BFF routes for spreads, readings, agent activity, dependency status |
| Auth | Auth0 session + verified bearer at the BFF; GitHub login; verified-email linking Action |
| MCP | `get_previous_readings` over mock data; per-user activity feed with SSE; refused calls recorded |
| Delivery | reusable CI, GHCR images, digest manifests, application validation incl. browser e2e, releases |
| Edge | nginx compose: storefront on `:8080`, backends on loopback `:8081`, client `X-Forwarded-For` dropped, JSON access log, no TLS |
| Org | `SECURITY.md`, secret scanning, push protection, private vulnerability reporting, Dependabot on every repo; wikis; org Project with priorities |

## Stubbed or missing

- NASA service: scaffold only (health, declared TCP patterns, no NASA client). ai → nasa not wired.
- MCP history: mock adapter; no OBO exchange, no PostgreSQL MCP, no RLS schema.
- Token signatures in MCP not verified; authority service not designed.
- Mobile calls tarot directly with stub spreads; waits on a bundle id.
- `REQUIRE_SPREAD_AUTH` off; `GET /spreads/:id` unprotected.
- Streaming readings, `reading.created` events.
- Deploy target undecided; deploy workflow is a stub.

## Known problems

- UTC day still in the draw seed (breaks invariant 4; remove it).
- One BullMQ queue per event type: a second consumer competes instead of receiving a copy.
- Outbox and inbox rows grow forever; no retention sweep.
- Trust of `x-forwarded-for` outside the edge is open.
- Dependabot alerts open across repositories; `next` patched in the storefront only.
- `INFRA_DISPATCH_TOKEN` expired: service images no longer notify infrastructure.
- Weekly schedules in ai / mcp never fire (cron on `development`, default branch `main`).

## Open product questions (owner decides)

Spread types and positions · prediction shape and disclaimer · canonical deck and reversed meanings ·
how NASA data may appear · reading retention and delete · what an agent may see of past readings ·
smartwatch scope · "driver injection" meaning · deploy target · Auth0 `sub` to reading user id.
