# Changelog

All notable changes to the CapyDB plugin for Claude are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow
[SemVer](https://semver.org/).

## [Unreleased]

## [1.0.0] - 2026-09-26

### Added

- The CapyDB connector: the remote MCP server at `https://mcp.capydb.dev/mcp`, signed in with
  OAuth through the CapyDB dashboard.
- Skills: `capydb` (set up and work with a database), `capydb-safe-changes` (rehearse,
  checkpoint, change, verify, undo) and `capydb-ephemeral` (throwaway databases with no account).
