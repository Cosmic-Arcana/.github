# Invariants

Rules every repository keeps. A change that would break one needs an owner decision first; say which
rule and propose an alternative instead of complying silently.

## Product

1. **Readings are fiction.** Never present a prediction as a factual claim about the future.
2. **NASA data is real and symbolic only.** It may color the story; it is never evidence that
   prediction works. UI and agent output keep "fictional prediction" and "real retrieved
   information" visibly distinct.
3. **Never invent business rules.** Spread shapes, card meanings, prediction format, retention and
   similar product rules come from the owner. Until decided, keep the behavior behind a port or a
   flag and record the open question in the issue or PR.

## Randomness

4. **Time, location and NASA / sky data never enter the draw seed.** The draw path
   (`ai-service-api/src/tarot`) imports no celestial, NASA or geolocation code. The UTC-day term in
   `draw-seed.ts` is known debt and must be removed, not extended.
5. **The BFF forwards no location to tarot or ai**, and observer-dependent sky values are not stored.
6. **Production never imports the research labs** (`arcana-rng-lab`, `arcana-prng-fingerprint`) and
   never writes into them. Data crosses only as exported files.

## Events and data

7. **The owner announces.** Only the service that owns a fact produces its event. Only
   tarot-service-api produces `spread.created`.
8. **Persist and announce atomically.** State change and outbox row commit in one transaction; a
   relay publishes. CDC (Debezium) is the planned swap.
9. **Events are thin past-tense facts:** version, `eventId`, ids, `occurredAt`. Never content such as
   cards or prediction text. Consumers re-query the owner.
10. **Consumers are idempotent through an inbox** written in the same transaction as the projection.
    BullMQ `jobId` dedupes only at `add()`; it is not idempotency.
11. **A database per service.** No service writes another's database. Read sides never produce
    events. Projections live in the service that owns the read model.
12. **Side-effecting commands take an `idempotency-key`.** The key check and the effect are atomic
    (transaction or unique constraint), never an `if` in code.
13. **Contracts first.** A cross-service event, resource or MCP schema is defined in
    `cosmic-arcana-sdk` before any service uses it. Parsers reject anything off-contract.

## Services and access

14. **ai-service-api is stateless.** No database; agent context arrives through MCP.
15. **The BFF is the only frontend-facing API.** Clients do not call services directly (mobile is
    the known exception until it moves).
16. **The user is never an MCP tool input.** Identity comes from the token, later OBO + RLS on the
    `cosmic_agent` schema. Previous readings are a tool the agent chooses to call, not a resource.
17. **OBO identity is observability, never authorization**, until signatures are verified.
18. **Authorization lives at the resource**, not duplicated in BFF and MCP.
19. **Outside systems sit behind ports.** Anything that needs owner setup and does not exist yet is
    mocked behind a port.
20. **Account linking only when both emails are verified.**

## Delivery and safety

21. **No credentials in tracked files.** `.env.example` holds names only. Secrets the agent cannot
    create are requested from the owner.
22. **Manifests pin image digests, never `latest`.**
23. **No force push to `main`, no empty commits.**
24. **A new npm package** must exist on the registry and carry a one-line justification (guards
    against slopsquatting).
25. **Logs never contain secrets, tokens, authorization headers, full bodies or the user's raw
    question.** Log a length or a hash.
26. The word "russian" is always lowercase.
