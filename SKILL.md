---
name: tasknotes-workflows
description: Create, review, repair, or explain TaskNotes Workflows Markdown automation for Obsidian using mdbase Runtime 0.2 records, CEL, and the TaskNotes runtime API.
---

# TaskNotes Workflows

Author Markdown notes under `TaskNotes/Workflows/` unless another folder is configured. Frontmatter is executable configuration; preserve the user's Markdown body.

Use `type: runtime_workflow`, `version: 1.0.0`, and `x-tasknotes.format_version: 2`. In an mdbase 0.3 collection, read `mdbase.yaml` and use its first `settings.explicit_type_keys` key instead of `type`; new TaskNotes v5 collections use `mdbase_type`. Write only that key, not `type` as well; the runtime standard pack 0.2.1 or later accepts it. Omitted keys default to `type`; an explicitly empty list needs collection configuration. Do not install packs automatically.

Keep mutating examples disabled by default. Give workflows, triggers and steps stable IDs. Triggers reference `event: { id, version }`; steps reference `action: { id, version }`. Use `for_each`, `run.limits.max_items`, `run.on_error`, and `run.concurrency` rather than legacy camelCase run fields.

## CEL and events

Use an object with exactly one `$expr` key for CEL. Plain strings, including `{{...}}`, are literals. Do not generate Bases formulas or legacy templates for new workflows.

Activation: `event` is a CloudEvent with payload under `event.data`; `workflow` and `trigger` are runtime records; `vars` are evaluated in dependency order; step outputs are `steps.<id>.output`; `item` is the current `for_each` item. Use brackets for hyphenated IDs, `today().addDays(3)` for date arithmetic, and `now()` for the captured instant. Variable dependencies require named/literal keys, not dynamic indexing.

TaskNotes API events expose `event.data.taskId`, `taskPath`, `task`, `before`, `after`, and `changes: { field: { before, after } }` when present. Write transitions as trigger guards:

```yaml
triggers:
  - id: started
    event: { id: task.status.changed, version: 1.0.0 }
    if:
      $expr: 'event.data.before.status == "open" && event.data.after.status == "active"'
    x-tasknotes:
      type: tasknotes.event
```

Use `input.taskId: { $expr: event.data.taskId }` for supported TaskNotes v5 task reads/mutations, or `input.task` for a path. All task reads/writes use the TaskNotes runtime API. Semantic fields follow TaskNotes' configured mappings; arbitrary fields go under `patch.customFrontmatter` (`null` removes them).

For cross-application events/actions, the installed mdbase plugin ID is `mdbase-sync` (API 1, interop profile 0.1). **Allow plugin integrations** must be enabled. The bridge is ephemeral and does not execute workflows; this plugin executes them. Exactly one compatible action provider is required. Explicit `provider` selectors are only needed for intentional provider coupling.

## Run policy

Bound iteration with `run.limits.max_items`, use `run.limits.timeout` where appropriate, and choose `run.on_error: stop|continue`. Concurrency supports `skip|queue|replace|allow` with literal or CEL-valued groups (default workflow ID). Run idempotency defaults to `workflow.id:event.id:trigger.id`, with seven-day persistent reservations. Interop step keys append step ID and item index. Cron uses an IANA timezone or `local`; missed schedules coalesce once.

Automatic runs should be enabled on one synced-vault device. The device-local **Run workflows on this device** setting defaults on for desktop and off for mobile; manual runs remain available. Do not promise exactly-once effects, distributed leases, durable queue recovery, or interrupted-run replay.

## Query and catalog

`task.query.input.query` must use the canonical TaskNotes runtime DTO, not compact field maps or internal `FilterQuery` objects. Load [references/query-dto.md](references/query-dto.md) for fields, operators, grouping and date values.

When a live runtime is available, inspect the installed catalog:

```js
app.plugins.getPlugin("tasknotes")?.api.extensions.get("tasknotes-workflows")?.listStepDefinitions();
```

Use `inputFields` and `outputFields` to verify action shapes. Read [references/workflow-schema.md](references/workflow-schema.md) for the complete format and execution limits, and [references/examples.md](references/examples.md) for runnable authoring patterns.

## Existing workflows

Format v1 and older `workflow`/`tasknotes-workflow` notes still use Bases until explicitly migrated. Use the plugin's review → backup → apply → rollback migration. Unsupported conversions must be reported and the migrated workflow disabled with an explanation. Do not relabel an old expression as CEL without converting it. CEL workflow editing opens the YAML note; the legacy visual formula builder remains for v1 files.

Include a short body explaining intent, mutations, assumptions, and dry-run checks. Verify CEL syntax, unique IDs, query DTOs, bounded iteration, and the exact installed action catalog before enabling a workflow.
