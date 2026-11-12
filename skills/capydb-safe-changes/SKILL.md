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
   for its job with `get_job`, run the statement on the preview with `run_preview_sql` and check
   the result there (also with `run_preview_sql`) first. The connector cannot open a connection
   string itself, so `run_preview_sql` is how SQL reaches a preview.
3. Create a checkpoint with `create_restore_point` and a short `label` such as
   `before-orders-migration`. This is what makes the change undoable. Which kind depends on the
   plan (`get_project` returns it):
   - **Ship and Business:** kind `pitr` pinned to now.
   - **Vibe:** restores come from backups only - a `pitr` point would be refused when you try to
     restore it. Take `create_backup`, wait for its job with `get_job`, find its `backup_key` with
     `list_backups`, and create the restore point with kind `backup` and that key.

## Making the change

- Use `execute_sql`, one statement per call, with a `WHERE` clause that selects exactly the rows
  you mean. An `UPDATE` or `DELETE` without `WHERE`, and any `TRUNCATE`, is refused by CapyDB -
  read the refusal and scope the statement instead of working around it. `DO` blocks and `CALL`
  are refused too, because their effect cannot be checked from the text; the user can run them
  in the dashboard's SQL Studio or with `capydb sql`.
- On Postgres 18 projects, append `RETURNING old.*, new.*` to `UPDATE` and `DELETE`, so the
  exact values you overwrote are recorded in the result and in SQL history.

## After the change

1. Verify with `query_sql` (row counts, spot checks) and `get_schema` for schema changes.
2. Tell the user what changed, how you verified it, and the restore point's label.
3. Once the user is satisfied, `delete_restore_point` removes the checkpoint; leave it if they
   are not sure yet.

## Undoing a change

`restore` with the restore point into a **new preview** (`preview_name`), then compare the
preview with production: `run_preview_sql` reads the preview, `query_sql` reads production.
Recover rows by copying them back with `execute_sql`, or have the user promote the preview from
the dashboard. Never try to overwrite production from the connector; it is deliberately not
possible.

## When CapyDB answers 429

A `429` is CapyDB's rate limit (per caller), not a failed change. Wait a few seconds and retry the
same call once; don't loop on it or start other calls meanwhile. If it keeps coming back, stop and
tell the user.
