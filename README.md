# Cosmic Arcana organization config

Organization-wide defaults for every Cosmic Arcana repository.

- `profile/README.md` — the public organization profile.
- `agents/` — Copilot custom agents available in every repository of the organization.

## Custom agents

| Agent | Use it for |
| --- | --- |
| `cosmic-arcana-engineer` | Implementing changes with the full system picture and conventions |
| `cosmic-arcana-architect` | Read-only design and PR review against the architecture invariants |

Each service is its own repository, so an agent working in one sees only that service. The agent
profiles carry the rest: service map, event flow, invariants and conventions.

### Adding or changing an agent

- One file per agent: `agents/<kebab-case-name>.agent.md`. The file name is the agent's identity; a
  repository-level agent with the same name overrides this one.
- Frontmatter needs `description`. Add `tools` only to restrict (read-only agents list
  `["read", "search", "github/*"]`).
- The body is self-contained and under 30,000 characters. Agents cannot read other repositories,
  so architecture facts must live in the profile, not in a link.
- When an invariant, service or convention changes, update both profiles in the same commit.
- Never put secrets in a profile. MCP server credentials go through `${{ secrets.COPILOT_MCP_* }}`.

### Compliance

- Predictions are fiction; agents must never present them as fact or present NASA data as evidence.
- No agent may log or echo secrets, tokens or a user's raw question.
