---
name: capydb-safe-changes
description: Make a risky change to a CapyDB database safely - a schema migration, a bulk UPDATE or DELETE, dropping a column or table, an import over existing data. Use before running any statement that rewrites or removes existing rows or schema in a CapyDB project, or when the user asks to undo such a change.
---

# Changing a CapyDB database without losing data

A statement that drops the wrong column looks exactly like one that succeeded. This loop makes
every risky change reversible and verified.

## Before the change

1. Confirm the target: `get_project` (name, environment, Postgres version). If the project is
   production, say so to the user and get an explicit go-ahead for the specific statement.
2. Rehearse when the change is non-trivial: `create_preview_database` with mode `clone`, wait
   for its job with `get_job`, run the statement against the preview's own connection string
   (`get_preview_connection_strings`) and check the result there first.
3. Create a checkpoint: `create_restore_point` with a short `label` such as
   `before-orders-migration`, pinned to a PITR timestamp of now or to the key of a backup you
   just took with `create_backup`. This is what makes the change undoable.

## Making the change

- Use `execute_sql`, one statement per call, with a `WHERE` clause that selects exactly the rows
  you mean. An `UPDATE` or `DELETE` without `WHERE`, and any `TRUNCATE`, is refused by CapyDB -
  read the refusal and scope the statement instead of working around it.
- On Postgres 18 projects, append `RETURNING old.*, new.*` to `UPDATE` and `DELETE`, so the
  exact values you overwrote are recorded in the result and in SQL history.

## After the change

1. Verify with `query_sql` (row counts, spot checks) and `get_schema` for schema changes.
2. Tell the user what changed, how you verified it, and the restore point's label.
3. Once the user is satisfied, `delete_restore_point` removes the checkpoint; leave it if they
   are not sure yet.

## Undoing a change

`restore` with the restore point into a **new preview** (`preview_name`), then compare the
preview with production using `query_sql` on each. Recover rows by copying them back with
`execute_sql`, or have the user promote the preview from the dashboard. Never try to overwrite
production from the connector; it is deliberately not possible.
