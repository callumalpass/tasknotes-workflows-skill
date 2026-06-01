# Runtime Task Query DTO

`task.query` accepts only the canonical TaskNotes runtime task query DTO. Do not emit compact field-map queries or internal TaskNotes `FilterQuery` objects.

## Shape

```yaml
query:
  where:
    all:
      - field: task.status
        op: ne
        value: done
      - any:
          - field: task.due
            op: lte
            value:
              fn: today
          - field: task.priority
            op: eq
            value: high
  sort:
    - field: task.due
      direction: asc
  group:
    - field: task.status
  limit: 25
  offset: 0
  scope:
    includeArchived: false
    folders:
      - Tasks
```

`where` can be a condition with `field`, `op`, and optional `value`, or a nested predicate using `all`, `any`, or `not`.

## Operators

Canonical operators:

- `eq`, `ne`
- `contains`, `notContains`
- `in`, `notIn`
- `exists`, `missing`
- `lt`, `lte`, `gt`, `gte`
- `isTrue`, `isFalse`

Use `exists`, `missing`, `isTrue`, and `isFalse` without `value`.

## Common Fields

Common query fields include:

- `task.title`
- `file.path`
- `task.status`
- `task.priority`
- `task.tags`
- `task.contexts`
- `task.projects`
- `task.due`
- `task.scheduled`
- `task.archived`
- `task.isCompleted`
- `task.isBlocked`

When available, prefer the live TaskNotes filter field catalog from `api.catalog.filterProperties()` over hard-coded assumptions.

## Date Values

```yaml
value:
  fn: today
```

```yaml
value:
  fn: now
```

```yaml
value:
  fn: date
  value: "2026-06-01"
```

```yaml
value:
  fn: dateAdd
  value:
    fn: today
  amount: 1
  unit: day
```

## Outputs

`task.query` outputs:

- `tasks`: returned task records, usually used for `forEach`
- `count`: number of returned tasks
- `total`: tasks in scope before filtering
- `matched`: tasks matching `where` before `offset` and `limit`
- `returned`: tasks returned after `offset` and `limit`
- `groups`: group details with `key`, `label`, and `taskPaths`
- `groupPaths`: task paths keyed by group key
- `query`: normalized runtime query
- `warnings`: non-fatal query warnings

Grouping does not change which tasks are selected. It adds buckets for reporting, digest notices, or group-level iteration.
