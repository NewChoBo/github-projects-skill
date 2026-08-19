---
name: github-projects
description: Use this skill when working with GitHub Projects to inspect projects, list and filter project items, manage project fields and values, add Issues or Pull Requests to a project, reconcile desired project state, or verify GitHub Projects mutations across MCP, GitHub CLI, REST, or GraphQL-capable runtimes.
---

# GitHub Projects

Operate GitHub Projects as a portfolio and coordination surface while preserving native GitHub Issues, Pull Requests, Reviews, Checks, Releases, refs, and repository files as the authoritative detail/evidence surfaces for their own facts.

This skill defines how to use GitHub Projects safely and consistently. It does not implement a GitHub API wrapper and does not assume a specific provider, model, MCP host, or CLI is available.

## Capability discovery

Before performing material work, identify the GitHub Projects capability actually exposed by the current runtime.

Preferred backends, in order of directness when available:

1. GitHub Projects tools exposed by an MCP server, such as GitHub's official `projects` toolset;
2. GitHub CLI `gh project` commands when shell execution and authenticated `gh` are available;
3. GitHub Projects REST or GraphQL APIs through an authorized tool/runtime;
4. a project-specific adapter that preserves equivalent semantics.

Do not infer capability from provider identity alone. If no valid Projects read/write capability is available, return or persist a truthful capability blocker such as `PROJECTS_CAPABILITY_UNAVAILABLE`. Do not silently substitute Issue labels, Milestones, Notion, Linear, or another project-management system unless current policy explicitly authorizes that fallback.

Read capability and write capability are distinct. Never claim a mutation succeeded when the runtime exposes read-only Projects access.

## Project identity

Resolve a project using enough information to identify it unambiguously:

- owner login;
- owner type when the backend requires it (`user` or organization);
- project number or stable project ID;
- title only as a discovery aid, not as the sole mutation identity when multiple matches are possible.

When creating a project, record the returned project identity and reuse it for all subsequent calls in the same operation.

Do not invent project IDs, item IDs, field IDs, option IDs, iteration IDs, or view IDs.

## Read workflow

For material Project inspection, follow this sequence unless the backend provides an equivalent atomic operation:

1. resolve the project;
2. list project fields and capture exact field names, IDs, types, and options;
3. list project items with all required fields;
4. paginate to completion unless the user explicitly requested a bounded page/range;
5. join Project metadata with native Issue/PR state only when the answer requires both;
6. state explicitly when data is incomplete because a page, field, repository, or capability could not be read.

When using GitHub's official Projects MCP tooling, follow its server guidance: list fields before item filtering or mutation, use exact returned field names, and paginate while `hasNextPage` is true.

Do not count nested field values, content objects, or pagination records as separate Project items.

## Filtering and queries

Construct filters from the user's actual intent and only from fields that exist in the target project.

Examples:

- open issues -> include Issue type and open state;
- merged pull requests -> include PR type and merged state;
- current iteration -> use the actual iteration field name returned by the project;
- priority/status/team filters -> use the actual Project field and option values, not assumed organization conventions.

Preserve the same query, requested fields, and page size across pagination cursors unless the backend requires otherwise.

For list/show/get/fetch requests, enumerate the requested records and total. Summarize instead of enumerating only when the user asked to analyze, summarize, report, or provide an overview.

## Mutation workflow

Use idempotent read-before-write behavior.

Before a mutation:

1. resolve the target project and item;
2. read the current field/value or membership state;
3. resolve the exact field and option IDs required by the backend;
4. determine whether the requested desired state already exists;
5. avoid unnecessary writes when current state already matches;
6. perform the smallest mutation needed;
7. re-read the mutated subject and verify the resulting state.

Never report success from the mutation response alone when a verification read is available.

When a write partially succeeds, report the exact completed and incomplete parts. Do not retry unrelated mutations blindly.

## Adding Issues and Pull Requests

Before adding an Issue or Pull Request to a Project:

- identify the exact repository and Issue/PR number;
- inspect existing Project items to avoid duplicate membership where practical;
- preserve the native Issue/PR as the authoritative source for its body, open/closed/merged state, review status, checks, and comments;
- treat the Project item as routing/portfolio metadata around that native record.

Do not create a draft Project item when an existing Issue or PR is the intended durable owner unless the user explicitly wants an unbound draft item.

## Project fields

Always list fields before field-dependent reads, filters, or writes.

For each field, distinguish its actual type, for example:

- text;
- number;
- date;
- single-select;
- iteration;
- built-in/system fields;
- other backend-supported types.

For single-select and iteration fields, resolve the exact returned option or iteration identity. Never fabricate option IDs from labels.

Do not assume every backend can create or modify every Project field type. If field creation/update is unsupported by the exposed backend, report the precise capability gap instead of pretending the schema was reconciled.

## Schema reconciliation

When the user provides a desired Project schema, treat reconciliation as a comparison, not a destructive reset.

1. read existing fields and options;
2. classify each desired element as `PRESENT`, `MISSING`, `DIFFERENT`, `UNSUPPORTED`, or `AMBIGUOUS`;
3. preserve existing compatible fields rather than recreating them;
4. create or update only what current capability and authority support;
5. do not delete extra fields/options merely because they are absent from the desired schema unless deletion was explicitly requested;
6. verify the resulting schema after mutation;
7. report unsupported differences separately.

Prefer `ensure`/reconcile semantics over unconditional create operations.

## Views

Treat Project views as presentation/routing configuration, not canonical work state.

Before modifying a view, inspect current view capability and current configuration when the backend exposes it. Preserve unrelated filters, grouping, sorting, visible fields, and layout unless the requested operation intentionally changes them.

If the current MCP/CLI/API surface does not expose view management, report `PROJECT_VIEW_CAPABILITY_UNAVAILABLE`; do not claim the view was configured.

## Status and workflow semantics

Project Status or custom workflow fields are coordination metadata. They do not replace native GitHub evidence.

Do not infer any of the following solely from a Project field:

- `DONE` -> acceptance criteria satisfied;
- `REVIEW` -> a formal PR review exists;
- `PASSED` -> tests/checks actually passed;
- `MERGED` -> the pull request is merged;
- `RELEASED` -> a tag/release/artifact exists.

When material, verify native GitHub evidence directly.

Likewise, closing an Issue or merging a Pull Request does not automatically authorize a specific Project status unless the project's workflow defines that mapping.

## Cross-repository Projects

Projects may aggregate Issues and Pull Requests from multiple repositories.

When reconciling such Projects:

- retain repository identity on every native work item;
- avoid assuming labels, milestones, branch names, release policies, or field semantics are identical across repositories;
- use Project fields for shared cross-repository routing only where the field has explicit common meaning;
- inspect repository-local acceptance/review/release evidence before changing high-level portfolio state.

## Concurrency and stale state

Project state can change between reads and writes.

For material mutations:

- keep the read-to-write window small;
- re-resolve item/field identity if the project schema or membership changed;
- after a failed write caused by stale identity or changed schema, re-read before retrying;
- do not retry with guessed IDs;
- avoid two agents performing overlapping schema or field mutations without an explicit owner.

## Deletion and archival

Treat deletion and archival as separate actions.

Prefer archival when the goal is to remove an item from active operational views while retaining history. Delete only when the user or governing workflow actually requires removal and the backend semantics are understood.

Before deleting a Project item or field, inspect dependencies and confirm the action does not destroy unique operational state that should remain on a native Issue/PR or another durable record.

## Verification checklist

Before declaring a GitHub Projects operation complete, verify applicable items:

- correct owner and project resolved;
- required project pages fully read;
- exact fields/options/iterations resolved from current state;
- duplicate membership avoided;
- mutation was authorized and technically supported;
- write was idempotent where practical;
- mutation result was re-read and verified;
- native Issue/PR/Review/Check/Release facts were not inferred from Project metadata;
- unsupported capabilities or partial failures are explicit;
- no unrelated fields/items/views were changed.

## Provider/runtime neutrality

The logical procedure in this skill must remain valid across GPT, Codex, Claude, GitHub Copilot, local CLI agents, and future runtimes.

Backend-specific syntax belongs to the runtime/tool adapter:

- GitHub MCP tool calls when available;
- `gh project` commands when available;
- REST/GraphQL calls when available.

Do not fork this skill into provider-specific variants merely because the invocation syntax differs.

## Safety and data handling

Never persist credentials, OAuth tokens, PATs, private chain-of-thought, or unnecessary conversation transcripts in Project items or repository records.

Respect GitHub visibility boundaries. A public Project, Issue, Pull Request, comment, repository file, commit, or release must contain only information safe for public disclosure.

Use the least-privileged Projects permission sufficient for the requested action. A runtime having broad GitHub credentials does not grant permission to mutate unrelated projects or repositories.

## Completion states

Use truthful outcomes such as:

- `PROJECTS_OPERATION_VERIFIED`;
- `NO_CHANGE_REQUIRED`;
- `PARTIAL_UPDATE`;
- `PROJECTS_CAPABILITY_UNAVAILABLE`;
- `PROJECTS_WRITE_UNAVAILABLE`;
- `PROJECT_FIELD_CAPABILITY_UNAVAILABLE`;
- `PROJECT_VIEW_CAPABILITY_UNAVAILABLE`;
- `PROJECT_IDENTITY_AMBIGUOUS`;
- `PROJECT_STATE_CHANGED_RETRY_REQUIRED`.

Do not invent success when the Project mutation or verification could not be performed.
