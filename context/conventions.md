# Conventions

## Code

- TypeScript everywhere. NestJS 11 for services, Next.js 16 App Router for the storefront, Expo for
  mobile. Prettier `printWidth: 100`.
- File names are kebab-case. Frontend `.ts` / `.tsx` files may use CamelCase where frontend
  convention expects it. Never mix conventions inside one module.
- Hexagonal: outside systems behind ports; CQRS commands and queries in the write/read services.
- No comments unless they explain *why*: complex business logic, a non-obvious edge case, an unusual
  implementation, a deliberate deviation from best practice.
- English only in code, docs and commits.

## Tests

- BDD first: Given / When / Then scenario and a failing test before the code.
- Database services: e2e with testcontainers (postgres + redis); Docker required.
- Storefront: Playwright, a functional suite from source and a production-build suite.
- `cosmic-arcana-sdk`: contract parsing tests; parsers reject anything off-contract.
- No separate script only for type checking.

## Logging

Every log line is one single-line JSON object through the framework logger. Never `console.*`.

| Field | Rule |
| --- | --- |
| `timestamp` | ISO 8601 |
| `level` | `error` \| `warn` \| `log` \| `debug` |
| `service` | emitting service name |
| `correlationId` | created at the BFF, propagated across every HTTP and TCP hop; never re-invented |
| `context` | emitting class or module |
| `message` | short, lowercase, event-shaped: `outbound call failed` |

Optional where meaningful: `durationMs`, `statusCode`, `method`, `route`, `messagePattern`,
`idempotencyKey`, `attempt`, `outcome`, `errorCode`, `errorName`.

- Log at boundaries only: inbound received / completed, outbound started / retried / failed, use case
  completed. Never the same event at two layers.
- Errors log `errorName` and `message`; a stack only at `error` level.
- Never secrets, tokens, authorization headers, full bodies or the raw question.

## Git

- Branches: `feature/…`, `fix/…`, `chore/…`, `development`, `main`.
- Commit subject: `[n-type]: short why`. `n` increments per repository; `type` is `fix`, `feature`,
  `chore`, `cleanup`, `docs` or `test`. Example: `[1-fix]: renamed fields to match database schema`.
- One concern per commit, usually one file or one tiny behavior. Do not bundle UI, API, lockfile
  and docs. Do not squash unless the owner asks.
- No empty commits, no secrets, no force push to `main`.
- PRs target `development` in service repos (`main` in infrastructure and `.github`); an issue in
  the owning repository, linked with `Closes #n`, and a card on the org Project.

## Dependencies

- Check the package exists on the registry and justify it in one line in the PR before installing.
- Services pin `@cosmic-arcana/sdk` through git (`github:Cosmic-Arcana/cosmic-arcana-sdk#semver:…`),
  lockfiles resolve to `git+https`.

## Working style

- Read the repository before changing it; match its patterns.
- Smallest change that keeps the [invariants](invariants.md).
- Answers are short: decisions and reasons, not implementation narration.
- Do not ask permission to commit or push; stop only for secrets you cannot create or undefined
  product rules.
