# Workflow Examples

## Manual Active Count

```markdown
---
type: tasknotes-workflow
schemaVersion: 1
id: show-active-count
name: Show active task count
enabled: false
description: Show a notice with the number of active tasks.
triggers:
  - id: manual
    type: manual
conditions: []
steps:
  - id: active
    type: task.query
    input:
      query:
        where:
          field: task.status
          op: eq
          value: active
        limit: 25
  - id: notice
    type: notice.show
    input:
      message: "Active tasks: {{steps.active.count}}"
run:
  mode: sequential
  noOverlap: true
  source: tasknotes-workflows
  maxTasks: 25
  onError: stop
---

# Show active task count

Run this manually from the command palette. Dry-run should show the notice text without mutating tasks.
```

## Morning Overdue Review

```markdown
---
type: tasknotes-workflow
schemaVersion: 1
id: morning-overdue-review
name: Morning overdue review
enabled: false
description: Find overdue open tasks every morning and mark them high priority.
triggers:
  - id: every-morning
    type: cron
    schedule: "0 9 * * *"
    timezone: local
conditions: []
steps:
  - id: overdue
    type: task.query
    input:
      query:
        where:
          all:
            - field: task.due
              op: lt
              value:
                fn: today
            - field: task.status
              op: notIn
              value:
                - done
                - cancelled
        sort:
          - field: task.due
            direction: asc
        limit: 50
        scope:
          includeArchived: false
  - id: mark-high
    type: task.patch
    forEach: "{{steps.overdue.tasks}}"
    input:
      task: "{{item.path}}"
      patch:
        priority: high
run:
  mode: sequential
  noOverlap: true
  source: tasknotes-workflows
  maxTasks: 50
  onError: continue
---

# Morning overdue review

Dry-run before enabling. Confirm the query selects only overdue incomplete tasks, then enable if the priority changes are desired.
```

## Grouped Status Digest

```yaml
type: tasknotes-workflow
schemaVersion: 1
id: grouped-status-digest
name: Grouped status digest
enabled: false
description: Show grouped counts for non-archived tasks.
triggers:
  - id: manual
    type: manual
conditions: []
steps:
  - id: tasks
    type: task.query
    input:
      query:
        where:
          field: task.archived
          op: isFalse
        group:
          - field: task.status
        limit: 100
  - id: notice
    type: notice.show
    input:
      message: "Found {{steps.tasks.count}} tasks across {{steps.tasks.groups.length}} status group(s)."
run:
  mode: sequential
  noOverlap: true
  source: tasknotes-workflows
  maxTasks: 25
  onError: stop
```
