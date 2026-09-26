---
name: capydb-ephemeral
description: Get a throwaway Postgres database in seconds with no CapyDB account - for trying an app, running a test suite, a demo, or a one-off experiment - and keep it later if the user wants. Use when the user wants a database right now and has not signed up, or asks for a temporary or disposable database.
---

# Throwaway databases with no account

`create_ephemeral_database` works before the user has signed in. It creates a real Postgres
database that is deleted, with its data, 72 hours after creation unless it is claimed.

1. Call `create_ephemeral_database` (optionally with a `name`). It returns a `project_id`, a
   `claim_token` and a `claim_url`. The claim token is the database's only credential and is
   shown exactly once: keep it for the calls below and give the user the `claim_url`. Never write
   either into a committed file, a commit message or a chat summary.
2. Poll `get_ephemeral_database` with the `project_id` and `claim_token` every couple of seconds
   until `state` is `ready`; the result then includes the connection strings. Write them into the
   app's environment file rather than into chat.
3. Tell the user when it expires and how to keep it: open the `claim_url`, or ask you to run
   `claim_ephemeral_database` after signing in to CapyDB. Claiming moves it into their workspace
   as a normal project and needs an active plan.
4. When the user is done and does not want to keep it, call `destroy_ephemeral_database` to free
   its slot instead of waiting out the 72 hours.

The number of unclaimed ephemeral databases is capped. A `503` means every slot is in use: tell
the user and suggest trying later or creating a project with an account, rather than retrying in
a loop.
