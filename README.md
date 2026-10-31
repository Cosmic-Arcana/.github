# Cosmic Arcana organization config

Organization-wide context and defaults for every Cosmic Arcana repository. Each service is its own
repository, so none of them shows the whole system; this one does.

```text
.github
├── context/                         the system picture, for people and agents
│   ├── architecture.md              diagram, repositories, flows (ask, history, agent, identity, delivery)
│   ├── invariants.md                rules every repository keeps
│   ├── conventions.md               code, tests, logging, git, dependencies
│   ├── status.md                    built / stubbed / known problems / open product questions
│   └── copilot-org-instructions.md  text for Settings → Copilot → Custom instructions
├── agents/                          Copilot custom agents, available in every org repository
│   ├── cosmic-arcana-engineer.agent.md
│   └── cosmic-arcana-architect.agent.md
├── profile/README.md                public organization page
└── SECURITY.md                      default security policy for every repository
```

Start with [context/architecture.md](context/architecture.md).

## Custom agents

| Agent | Use it for |
| --- | --- |
| `cosmic-arcana-engineer` | Implementing changes with the full system picture and conventions |
| `cosmic-arcana-architect` | Read-only design and PR review against the invariants (`read`, `search`, `github/*`) |

Pick one from the agents dropdown in the Copilot cloud agent panel, in an issue's **Assign to
Copilot**, or in VS Code / JetBrains. They need Copilot Business or Enterprise on the organization.

### How the context reaches an agent

| Layer | Where | Applies to |
| --- | --- | --- |
| Org custom agents | `agents/*.agent.md` here | every repository; picked per task |
| Org custom instructions | org Settings → Copilot (paste [copilot-org-instructions.md](context/copilot-org-instructions.md)) | every Copilot Chat, cloud agent and code review session on github.com |
| Repository instructions | `.github/copilot-instructions.md`, `AGENTS.md`, `CLAUDE.md` in each repo | that repository |
| Repository agents | `.github/agents/*.agent.md` in a repo | that repository; overrides an org agent with the same file name |

An agent runs inside the repository it was started in and cannot open other repositories, so every
agent profile carries the architecture inline. `context/` is the readable copy of the same facts.

### Changing the context

- A new service, invariant or convention: update `context/` and both agent profiles in the same PR,
  and refresh `context-snapshot` in the agents' frontmatter.
- `context/status.md` goes stale fastest; update it when a feature lands, is blocked or deferred.
- One file per agent: `agents/<kebab-case-name>.agent.md`; only `.`, `-`, `_`, letters and digits.
  Frontmatter needs `description`; add `tools` only to restrict.
- Agent bodies stay under 30,000 characters.
- To try an agent before release, put it in `.github/agents/` of this repository first (usable only
  here), then move it to `agents/`.
- This repository is public: no secrets, tenant ids or internal hostnames. MCP server credentials in
  agents go through `${{ secrets.COPILOT_MCP_* }}`.

### Compliance

- Predictions are fiction; no agent presents them as fact or presents NASA data as evidence.
- No agent logs or echoes secrets, tokens or a user's raw question.
