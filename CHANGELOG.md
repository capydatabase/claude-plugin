# Changelog

All notable changes to the CapyDB plugin for Claude are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow
[SemVer](https://semver.org/).

## [Unreleased]

### Added

- The `capydb` skill mentions the Postgres 19 readings of `get_observability` (plan-cache counts,
  lock waits, autovacuum priority) and the REPACK fix schema lint gives for bloat on 19.

### Changed

- **The `capydb-safe-changes` checkpoint works on every plan.** It pinned a point-in-time restore
  point, which Vibe cannot restore; on Vibe the skill now takes a backup, waits for it, and pins a
  `backup` restore point to its key. The `capydb` skill says point-in-time restores need Ship or
  Business.
- Both skills run SQL on a preview with `run_preview_sql` (they told the agent to use the
  preview's connection string, which the connector cannot open), say that a `429` is the rate
  limit (wait, retry once, then stop), and `capydb-safe-changes` notes that `execute_sql` refuses
  `DO` and `CALL`.
- The README no longer states a tool count for the connector (it said fifty; the server has
  seventy). It names what the tools cover instead.

## [1.0.0] - 2026-09-26

### Added

- The CapyDB connector: the remote MCP server at `https://mcp.capydb.dev/mcp`, signed in with
  OAuth through the CapyDB dashboard.
- Skills: `capydb` (set up and work with a database), `capydb-safe-changes` (rehearse,
  checkpoint, change, verify, undo) and `capydb-ephemeral` (throwaway databases with no account).
