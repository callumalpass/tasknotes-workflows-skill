# Workflow Schema Reference

Workflow files are Markdown notes with YAML frontmatter. The default folder is `TaskNotes/Workflows/`; the default Base view is `TaskNotes/Views/workflows.base`.

## Required Frontmatter

```yaml
type: tasknotes-workflow
schemaVersion: 1
id: short-lowercase-id
name: Human readable name
enabled: false
description: One sentence.
triggers: []
conditions: []
steps: []
run:
  mode: sequential
  noOverlap: true
  source: tasknotes-workflows
  maxTasks: 25
  onError: stop
```

Use lowercase ids with letters, numbers, dots, underscores, and dashes. `enabled: false` is the safe default for workflows that write tasks or vault files.

## Trigger Types

```yaml
triggers:
  - id: manual
    type: manual

  - id: status-active
    type: tasknotes.event
    event: task.status.changed
    from: open
    to: active

  - id: every-morning
    type: cron
    schedule: "0 9 * * *"
    timezone: local

  - id: every-half-hour
    type: interval
    every: 30m

  - id: project-note-opened
    type: obsidian.workspace
    event: file-open
    path:
      glob: "Projects/**/*.md"
```

TaskNotes event workflows should leave `allowSelfTrigger` unset unless reacting to workflow-originated writes is intentional.

## Conditions

Conditions can appear at workflow level or step level:

```yaml
conditions:
  - field: trigger.after.status
    operator: is
    value: active
```

Condition operators: `is`, `isNot`, `in`, `notIn`, `exists`, `missing`, `contains`, `startsWith`, `before`, `after`, `onOrBefore`, `onOrAfter`.

## Template References

References use constrained template expressions; arbitrary JavaScript is not evaluated.

```text
{{workflow.id}}
{{workflow.name}}
{{trigger.after.path}}
{{trigger.path}}
{{steps.query.tasks}}
{{steps.parents.tasks[0].path}}
{{item.path}}
{{today}}
{{now}}
```

If a string is exactly one reference, the underlying value is preserved. Embedded references are stringified.

## Step Types

Task read steps: `task.query`, `task.get`, `task.parents`, `task.subtasks`, `task.dependencies`, `task.blocking`, `task.relationships`.

Task write steps: `task.create`, `task.patch`, `task.set`, `task.move`, `task.archive`, `task.unarchive`, `task.complete`, `task.uncomplete`, `task.reschedule`, `task.setDue`, `task.clearDue`, `task.setScheduled`, `task.clearScheduled`, `task.addTag`, `task.removeTag`, `task.addProject`, `task.removeProject`, `task.addContext`, `task.removeContext`, `task.addDependency`, `task.removeDependency`.

Other steps: `time.start`, `time.stop`, `time.appendEntry`, `notice.show`, `obsidian.openFile`, `obsidian.createNote`, `obsidian.appendNote`, `obsidian.updateFrontmatter`, `obsidian.moveFile`, `workflow.stop`.

Use `forEach` for batch steps:

```yaml
steps:
  - id: query
    type: task.query
    input:
      query:
        where:
          field: task.status
          op: eq
          value: active

  - id: mark-high
    type: task.patch
    forEach: "{{steps.query.tasks}}"
    input:
      task: "{{item.path}}"
      patch:
        priority: high
```
