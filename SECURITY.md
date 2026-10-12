# Security policy

This policy covers every repository in the Cosmic Arcana organization that does not carry a
`SECURITY.md` of its own.

## Reporting a vulnerability

**Please report it privately. Do not open a public issue, pull request or discussion for it.**

Open the affected repository, go to the **Security** tab and choose **Report a vulnerability**. That
creates a private advisory that only you and the maintainers can see, where the fix can be prepared
before anything is public.

If a repository does not offer that button, open an issue titled `Security contact request` that
says nothing about the problem itself, and a maintainer will arrange a private channel.

## What helps

- Which repository and which commit or branch (`development` is where changes land first).
- The steps to reproduce it, and what you expected to happen instead.
- What an attacker would gain: reading another user's readings, acting as another user, reaching an
  internal service, running code, exposing a secret.
- Please use made-up data. Readings and questions can be personal, and you do not need real ones to
  show a problem.

## Scope

In scope are the code, configuration and pipelines in this organization's repositories: the
services, the storefront and its backend-for-frontend, the mobile app and browser extension, the
shared contracts, and the CI/CD and release workflows.

Out of scope:

- Third-party services we call (Anthropic, NASA, Auth0, GitHub): report those to their owners.
- Denial of service by sheer volume, social engineering, and physical attacks.
- A vulnerability in a dependency with no demonstrated way to reach it from this code. Report it
  upstream; our dependency alerts already track published advisories.

## Status of the project

Cosmic Arcana is an experiment and is not yet a production service. There are no versioned releases
to patch: the maintained code is the latest `development` and `main` of each repository, and the
contracts package is the one thing tagged (`v0.x`). Some protections are known gaps rather than
oversights, and are documented as such; for example, internal services trust their callers and the
agent token's signature is not verified yet. A report about a documented gap is still welcome, but
it may already be on the list.

## Disclosure

Please give us the chance to fix a problem before you publish it. Once a fix is available we will
publish a security advisory for it on the affected repository.

This policy does not set response times, offer a reward, or grant legal permission to test. Please
act in good faith: use only your own accounts and data, and do not disrupt the service.
