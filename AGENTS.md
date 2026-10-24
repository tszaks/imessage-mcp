# iMessage MCP

An MCP server (Node.js, CommonJS, stdio) that reads the macOS Messages database and sends messages through AppleScript. Dependencies: `@modelcontextprotocol/sdk` and `better-sqlite3`.

This file is for AI agents that work **on** this code. If you only use the server, read the [README](README.md).

## Build, test, run
- Node 18 up to but not including 26. Use Node 24 (`better-sqlite3` does not build on Node 26). On Homebrew: `PATH=/opt/homebrew/opt/node@24/bin:$PATH`.
- Install: `npm install` (or `npm ci`).
- Test: `npm test` (`node --test`). Run the whole suite before you say something works. Tests use mocks and temporary databases.
- Run: `node index.js` (stdio server; waits for an MCP client).
- Never run the server against the real Messages database to test sending. Do not call `send_message`, `send_message_batch`, `react_to_message` or any mutation tool for real. Use the injected `runScript` and temporary databases the tests use.
- Do not touch `~/Library/Messages`, and do not launch the Messages app.

## Code map
- `index.js`: everything else. Tool registration (the `tools` list in the `ListTools` handler), tool handlers, `attributedBody` decoder, contact cache (AddressBook SQLite), AppleScript builders, release-flag gating (`IMESSAGE_MCP_RELEASES`), backup and delete logic.
- `lib/batch.js`: batch preview, approval tokens.
- `lib/outreach.js`: follow-up classification for `find_outreach_followups`.
- `test/`: `mcp-tools.test.js`, `mutation-tools.test.js`, `batch.test.js`, `outreach.test.js`.

## Conventions
- Match the surrounding code: naming, comment density, idiom. Comments say why, not what.
- Plain CommonJS. No new dependencies without discussion.
- Sending, reacting and deleting stay behind `confirm: true` (plus an approval token for batches and `delete_threads`). Do not weaken a gate.
- Mutation and UI-automation tools stay behind release flags. A tool that needs a flag must not be listed without it.
- The server makes no network requests. Do not add any.
- Never put real messages, phone numbers or contact names in code, tests or docs.
- When you add or change a tool or release flag, update the README tools table and release flags table, and `CHANGELOG.md`.
