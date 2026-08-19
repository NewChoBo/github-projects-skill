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

## Portability

The canonical `SKILL.md` avoids provider-specific frontmatter and private project identities. Provider-specific installation or runtime bindings should remain outside the skill semantics.

## Status

Initial skill implementation. Validate in multiple runtimes before treating provider-specific behavior as portable.
