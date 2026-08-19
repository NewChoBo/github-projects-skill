# github-projects

Provider-neutral Agent Skill for operating GitHub Projects safely and consistently from AI agents such as GPT/Codex, Claude Code, and other Agent Skills-compatible runtimes.

> Repository name may later change to `github-projects-skill`. The canonical skill identity remains `github-projects`.

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

## Capability boundary

This repository does **not** implement a GitHub Projects API wrapper, MCP server, authentication service, scheduler, or orchestration runtime.

The skill consumes whatever GitHub Projects-capable tooling the current runtime actually exposes, for example an official GitHub integration/MCP, GitHub CLI, REST API, or GraphQL API. Tool availability never implies authority.

## Install

GitHub CLI can discover this repository through the standard `skills/*/SKILL.md` layout.

Preview without installing:

```bash
gh skill preview NewChoBo/github-projects github-projects
```

Install for Codex:

```bash
gh skill install NewChoBo/github-projects github-projects --agent codex
```

Install for Claude Code:

```bash
gh skill install NewChoBo/github-projects github-projects --agent claude-code
```

Install at user scope when the skill should be available outside one repository:

```bash
gh skill install NewChoBo/github-projects github-projects --agent codex --scope user
```

When this repository is renamed to `NewChoBo/github-projects-skill`, use the new repository name for future installs. The skill name remains `github-projects`.

## Validate and publish

From a local clone of this repository:

```bash
gh skill publish --dry-run
```

After validation, publish a versioned GitHub Release, for example:

```bash
gh skill publish --tag v0.1.0
```

Consumers can then install or pin a tagged version instead of following the default branch.

## Portability

The canonical `SKILL.md` avoids provider-specific frontmatter and private project identities. Provider-specific installation or runtime bindings remain outside the skill semantics.

## Status

Initial skill implementation. Validate in multiple runtimes before treating provider-specific behavior as portable.
