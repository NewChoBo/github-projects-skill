# github-projects

Provider-neutral Agent Skill for operating GitHub Projects safely and consistently from AI agents such as GPT/Codex, Claude Code, and other Agent Skills-compatible runtimes.

Repository: `NewChoBo/github-projects-skill`  
Canonical skill identity: `github-projects`

## Skill

Canonical skill source:

```text
skills/github-projects/SKILL.md
```

The skill defines how an agent should:

- discover and resolve GitHub Projects without guessing IDs;
- inspect fields, options, items, and current state before mutation;
- add Issues/PRs or draft items without creating duplicates;
- update project fields idempotently;
- distinguish Project metadata from native Issue/PR/Review/Check truth;
- reconcile desired state with current state;
- verify every material write by reading the resulting state back;
- fail truthfully when the runtime does not expose GitHub Projects capability.

Detailed coverage is documented in:

```text
skills/github-projects/references/capability-matrix.md
```

The intended skill surface includes Project lifecycle, items/drafts, fields/values, views, Project workflows, Project-level status updates, repository/team links, collaborators/permissions, templates, archival/deletion, and cross-repository coordination. Actual technical support remains capability-dependent: the skill uses an available GitHub Projects MCP, `gh project`, REST, or GraphQL backend rather than pretending every backend exposes every operation.

## Capability boundary

This repository does **not** implement a GitHub Projects API wrapper, MCP server, authentication service, scheduler, or orchestration runtime.

The skill consumes whatever GitHub Projects-capable tooling the current runtime actually exposes, for example an official GitHub integration/MCP, GitHub CLI, REST API, or GraphQL API. Tool availability never implies authority.

## Install

GitHub CLI can discover this repository through the standard `skills/*/SKILL.md` layout.

Preview without installing:

```bash
gh skill preview NewChoBo/github-projects-skill github-projects
```

Install for Codex:

```bash
gh skill install NewChoBo/github-projects-skill github-projects --agent codex
```

Install for Claude Code:

```bash
gh skill install NewChoBo/github-projects-skill github-projects --agent claude-code
```

Install at user scope when the skill should be available outside one repository:

```bash
gh skill install NewChoBo/github-projects-skill github-projects --agent codex --scope user
```

Pin a release when reproducibility matters:

```bash
gh skill install NewChoBo/github-projects-skill github-projects@v0.1.0 --agent codex --scope user
```

## CI/CD

Pull requests and pushes that touch the skill run:

```bash
gh skill publish --dry-run
```

through `.github/workflows/skill-ci.yml`.

Releases are **tag-driven**. The only action needed to publish a version is to push a SemVer tag that points to the current `main` commit, for example:

```bash
git checkout main
git pull --ff-only
git tag v0.1.0
git push origin v0.1.0
```

The `Release Skill` workflow then:

1. verifies the tag is SemVer (`vMAJOR.MINOR.PATCH` or a prerelease such as `v0.2.0-rc.1`);
2. verifies the tag points to the current `main` commit;
3. verifies the runner supports `gh skill`;
4. validates the skill with `gh skill publish --dry-run` against the tagged snapshot;
5. creates the GitHub Release from the existing tag with generated release notes;
6. automatically marks prerelease tags containing `-` as GitHub prereleases;
7. reads the release back to verify publication.

A tag that does not point to current `main`, an invalid Skill, or an existing Release fails closed and does not create a replacement release.

## Local validation

From a local clone of this repository:

```bash
gh skill publish --dry-run
```

Consumers can install or pin a tagged release instead of following the default branch.

## Portability

The canonical `SKILL.md` avoids provider-specific frontmatter and private project identities. Provider-specific installation or runtime bindings remain outside the skill semantics.

## Status

Initial skill implementation. Validate in multiple runtimes before treating provider-specific behavior as portable.
