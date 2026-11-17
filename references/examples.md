# Workflow examples

Use the collection's configured type key in place of `type` where required. All examples start disabled.

## Manual active-task count

```markdown
---
type: runtime_workflow
id: show-active-count
version: 1.0.0
name: Show active task count
enabled: false
triggers:
  - id: manual
    event: { id: tasknotes-workflows.manual, version: 1.0.0 }
    x-tasknotes: { type: manual }
steps:
  - id: active
    action: { id: task.query, version: 1.0.0 }
    input:
      query:
        where: { field: task.status, op: eq, value: active }
        limit: 25
  - id: notice
    action: { id: notice.show, version: 1.0.0 }
    input:
      message:
        $expr: '"Active tasks: " + string(steps.active.output.count)'
run:
  concurrency: { policy: skip }
  limits: { max_items: 25, timeout: 30s }
  on_error: stop
x-tasknotes: { format_version: 2 }
---

# Show active task count

Dry-run first and check the selected tasks and count before enabling. This workflow reads tasks and shows a notice.
```

## Start time when a task becomes active

```yaml
type: runtime_workflow
id: start-time
version: 1.0.0
name: Start time when active
enabled: false
triggers:
  - id: active
    event: { id: task.status.changed, version: 1.0.0 }
    if: { $expr: 'event.data.after.status == "active"' }
    x-tasknotes: { type: tasknotes.event }
steps:
  - id: start
    action: { id: time.start, version: 1.0.0 }
    input:
      taskId: { $expr: event.data.taskId }
run:
  concurrency:
    group: { $expr: event.data.taskId }
    policy: queue
  limits: { max_items: 1, timeout: 30s }
  on_error: stop
x-tasknotes: { format_version: 2 }
```

Body guidance: verify the configured active status and dry-run a sample event. This starts a real timer, so enable automatic execution on one device only.
