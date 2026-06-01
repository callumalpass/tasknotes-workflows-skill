---
name: tasknotes-workflows
description: Use when creating, reviewing, repairing, or explaining TaskNotes Workflows Markdown automation files for Obsidian. Covers workflow frontmatter, triggers, typed steps, safe mutation defaults, TaskNotes runtime query DTOs, template references, dry-run guidance, and human-readable workflow note bodies.
---

# TaskNotes Workflows

Use this skill to author safe TaskNotes Workflows notes. A workflow is a Markdown note whose YAML frontmatter is executable configuration and whose body explains intent, assumptions, and dry-run checks.

## Core Workflow

1. Write workflow notes under `TaskNotes/Workflows/` unless the user specifies another folder.
2. Use `type: tasknotes-workflow` and `schemaVersion: 1`.
3. Prefer `enabled: false` for any workflow that mutates tasks or vault files.
4. Use `triggers`, `conditions`, `steps`, and `run`; do not invent JavaScript, shell, HTTP, or untyped action blocks.
5. Use typed TaskNotes steps for task changes and typed Obsidian steps for vault changes.
6. Use `task.query` only with the canonical TaskNotes runtime task query DTO.
7. Add `run.noOverlap: true`, a bounded `run.maxTasks`, and `run.onError`.
8. Include a short Markdown body describing what the workflow does, what it mutates, and what to check in dry run.

## Query Rule

For `task.query`, never emit compact field-map queries or internal TaskNotes `FilterQuery` objects. Use the runtime DTO:

```yaml
query:
  where:
    all:
      - field: task.status
        op: ne
        value: done
      - field: task.due
        op: lte
        value:
          fn: today
  sort:
    - field: task.due
      direction: asc
  limit: 25
  scope:
    includeArchived: false
```

Load [references/query-dto.md](references/query-dto.md) when writing or reviewing non-trivial `task.query` inputs.

## Step Catalog

When an Obsidian runtime is available, inspect the live step catalog before relying on a step input shape:

```js
const tasknotes = app.plugins.getPlugin("tasknotes")?.api;
const workflows = tasknotes?.extensions.get("tasknotes-workflows");
workflows?.listStepDefinitions();
```

Use `inputFields` as the contract for YAML input and `outputFields` as the contract for references like `{{steps.query.tasks}}`.

## References

- Load [references/workflow-schema.md](references/workflow-schema.md) for frontmatter fields, trigger kinds, step categories, run policy, and template references.
- Load [references/query-dto.md](references/query-dto.md) for runtime task query fields, operators, grouping, dates, and result outputs.
- Load [references/examples.md](references/examples.md) when the user wants a concrete workflow pattern.

## Review Checklist

- YAML parses and has a stable lowercase `id`.
- Every trigger and step has a stable `id`.
- Mutating workflows start disabled unless the user explicitly wants them enabled.
- `forEach` expressions resolve to arrays and `run.maxTasks` is bounded.
- Task batches use `forEach: "{{steps.query.tasks}}"`.
- Template references are constrained to `workflow`, `trigger`, `vars`, `steps`, `item`, `today`, or `now`.
- The body tells a human how to verify the workflow before enabling it.
