# Copilot organization instructions

GitHub keeps organization-wide Copilot instructions in settings, not in a file. Paste the block below
into **Organization → Settings → Copilot → Custom instructions** and save. It then applies to Copilot
Chat on github.com, the cloud agent and code review in every repository (Copilot Business or
Enterprise required). Keep this file and the setting in sync.

```text
You are working in one repository of Cosmic Arcana, an AI fortune-teller split into one repository per
service under the Cosmic-Arcana organization. The full system picture lives in
Cosmic-Arcana/.github under context/ and in the org custom agents cosmic-arcana-engineer and
cosmic-arcana-architect; prefer those agents for cross-service work.

Product: readings are fiction; NASA data is real and symbolic only, never evidence. Keep fiction and
retrieved facts visibly distinct. Never invent business rules (spreads, meanings, prediction format,
retention): keep them behind a port or flag and raise the question.

Hard rules:
- Time, location and NASA data never enter the tarot draw seed.
- Only the owning service produces its event; state + outbox in one transaction; events carry ids
  only; consumers dedupe through an inbox in the projection's transaction.
- A database per service. ai-service-api is stateless. The BFF is the only frontend-facing API.
- Cross-service contracts go into cosmic-arcana-sdk first.
- The user is never an MCP tool input; identity comes from the token.
- Production code never imports arcana-rng-lab or arcana-prng-fingerprint.
- Logs: single-line JSON through the framework logger, never console.*, propagate correlationId
  from the BFF, never log secrets, tokens or the user's raw question.
- Commits: [n-type]: short why (n per repository; fix|feature|chore|cleanup|docs|test), one concern
  per commit, no force push to main, no secrets.
- kebab-case file names (frontend TS may use CamelCase). Comments only for non-obvious why.
- BDD: scenario and failing test before code.
- English only. The word "russian" is always lowercase.

Be brief: decisions and reasons, not narration.
```
