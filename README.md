# CapyDB for Claude

Managed Postgres for your agent. This plugin connects Claude to [CapyDB](https://capydb.dev) and
teaches it the workflows that keep your data safe: create a database and wire its connection
string into your app, branch a preview to try a change, checkpoint before risky SQL, and restore
without touching production.

It works in Claude on the web, desktop and mobile, in Cowork, and in Claude Code.

## What's inside

- **The CapyDB connector** (`.mcp.json`): the remote MCP server at `https://mcp.capydb.dev/mcp`,
  with tools for projects, connection strings, SQL, schema, preview databases, backups,
  restore points, imports, observability and jobs.
- **Skills**
  - `capydb` - set up and work with a CapyDB database: create, connect, query, preview, back up.
  - `capydb-safe-changes` - the rehearse, checkpoint, change, verify loop for migrations and
    destructive SQL, and how to undo a change.
  - `capydb-ephemeral` - a throwaway Postgres database with no account, deleted after 72 hours
    unless you keep it.

## Use it

Install the plugin, then connect the CapyDB connector from the plugin's **Connectors** tab (in
Claude Code it connects directly; run `/mcp` to sign in). Ask for what you need:

- "Create a Postgres database for this app and put the connection string in `.env`."
- "Give me a throwaway database to run the test suite against."
- "Add a `status` column to orders and backfill it - safely."
- "Why is the dashboard query slow?"

Throwaway databases work before you sign in. The first time a tool needs your account, Claude
asks you to sign in to CapyDB in your browser, where you choose the workspace it may use. You need
a CapyDB account with an active plan to create lasting databases; new accounts start with a free
month.

## Safety

- Tools declare whether they only read or can change data, so Claude asks before anything
  destructive. Reading SQL (`query_sql`) runs in a read-only transaction; changing SQL is a
  separate tool (`execute_sql`).
- CapyDB refuses an `UPDATE` or `DELETE` without `WHERE` and any `TRUNCATE`.
- Restoring over a production database is not available to the connector at all: restores go
  into a preview database that you promote yourself.
- The connection is an agent API key limited to project, backup and read-only organization
  access. It cannot manage API keys or change your organization, it expires after 90 days, and you
  can revoke it at any time under **Settings → API keys** in the CapyDB dashboard.

## Data

The plugin itself contains only instructions and the connector's address; it runs no code on
your machine and stores nothing. Tool calls go from Claude to CapyDB at `mcp.capydb.dev`, which
forwards them to the CapyDB API (`api.capydb.dev`) with your connection's key. Queries run
against your own database and their results return to your conversation. Nothing is sent to any
other service. CapyDB records API activity and SQL statements in your project's audit trail and
SQL history, as it does for the dashboard and CLI. See the
[privacy policy](https://capydb.dev/privacy).

## Support

Documentation: [docs.capydb.dev](https://docs.capydb.dev/docs/guides/integrations/claude) ·
Email: hello@capydb.dev

## License

MIT
