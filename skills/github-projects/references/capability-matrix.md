# GitHub Projects capability matrix

This reference defines the intended coverage of the `github-projects` skill. The skill describes the procedure; the current runtime/tooling determines which operations are technically available.

## Coverage principle

The skill aims to cover the complete operational surface of modern GitHub Projects needed by agents. It does not promise that every backend exposes every operation.

Use the most direct available backend and fall back to another authorized GitHub Projects backend when necessary:

1. GitHub Projects-capable MCP tools;
2. `gh project`;
3. GitHub REST API;
4. GitHub GraphQL `ProjectV2` API.

Never emulate an unsupported Projects operation with unrelated project-management systems unless explicitly authorized.

## Project lifecycle

Covered procedures include:

- list/discover projects;
- resolve exact project identity;
- inspect project metadata;
- create a project when supported;
- update title, description/readme, visibility and open/closed state when supported;
- copy/template operations when supported;
- delete a project only under explicit destructive authority;
- verify the resulting project identity/state after mutation.

## Items and draft items

Covered procedures include:

- list items with full pagination;
- inspect item/content identity;
- add Issues and Pull Requests;
- create draft items when an unbound item is intentionally required;
- update draft items when supported;
- update Project item fields;
- archive/unarchive items;
- delete/remove Project items;
- reorder items when supported;
- list items scoped to a Project view when supported;
- avoid duplicate membership;
- verify native Issue/PR state separately from Project metadata.

## Fields and values

Covered procedures include:

- list and inspect fields;
- create custom fields when supported;
- update field configuration when supported;
- delete fields only when explicitly requested and safe;
- resolve exact field IDs and option IDs;
- read/write text, number, date, single-select, multi-select and iteration values when the backend supports the field type;
- clear field values when supported;
- reconcile desired schema without destructive reset;
- preserve existing single-select options during partial updates.

Native Issue/PR attributes such as assignees, labels, milestones, review state and checks remain native GitHub concerns unless represented by explicit custom Project fields.

## Views

Covered procedures include:

- list/get views when supported;
- create views when supported;
- update/delete views through an available REST/GraphQL backend when supported;
- work with table, board and roadmap layouts;
- configure filters, visible fields, sorting and grouping when the backend exposes them;
- list items for a selected view;
- report `PROJECT_VIEW_CAPABILITY_UNAVAILABLE` instead of inventing success when the current backend lacks view operations.

## Project workflows and automation

Covered procedures include:

- list/inspect Project workflows when available;
- update workflow configuration or enabled state when the backend supports it;
- delete workflows only under explicit authority;
- distinguish workflow-driven item changes from direct agent mutations;
- preserve unrelated automation while changing one workflow.

GraphQL currently exposes ProjectV2 workflow objects and workflow mutations beyond the dedicated `gh project` command surface, so backend fallback may be required.

## Project status updates

Covered procedures include:

- read current/latest Project status updates;
- create Project status updates when supported;
- delete status updates only when explicitly requested;
- preserve status body, date window and status classification semantics;
- distinguish Project-level status updates from item `Status` field values.

## Repository and team links

Covered procedures include:

- inspect linked repositories/teams where available;
- link/unlink a Project to/from a repository;
- link/unlink a Project to/from a team;
- verify the relationship after mutation;
- avoid inferring repository-local permissions or policy from the Project link alone.

## Collaborators and permissions

Covered procedures include:

- inspect Project access where the backend exposes it;
- update user/team collaborators through supported APIs;
- preserve `READ`/`WRITE`/`ADMIN`-class distinctions;
- use least privilege;
- never infer mutation authority merely because the runtime credential is technically capable.

## Templates

Covered procedures include:

- inspect template state;
- mark/unmark organization-owned Projects as templates where GitHub supports it;
- copy a Project/template where supported;
- verify the copied/template Project separately from the source.

## Cross-repository coordination

Covered procedures include:

- aggregate Issues/PRs from multiple repositories;
- retain repository identity for every native record;
- reconcile Project-level shared fields without assuming repository-local labels/branches/release policy are identical;
- verify native repository acceptance/review/check/release evidence before high-level state transitions.

## Backend coverage rule

`gh project` is preferred for ordinary CLI operations but is not treated as the complete GitHub Projects API surface. REST and GraphQL may expose newer or broader functionality, including views, workflows, collaborators, templates, status updates, or linking operations.

When one backend lacks an operation:

1. report the missing capability;
2. determine whether another authorized GitHub Projects backend is available;
3. use the fallback only if semantics remain equivalent and authority permits it;
4. re-read and verify the result;
5. never claim full support from a backend that does not expose the requested operation.
