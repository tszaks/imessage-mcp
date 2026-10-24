# Contributing to iMessage MCP

Thanks for helping. This server can read and send a person's messages, so the bar for changes is: does it stay simple, and does it keep the user in control of what is sent or deleted?

By taking part you agree to follow the [Code of Conduct](CODE_OF_CONDUCT.md).

## Before you start

- For bugs, open an issue with the template. Do not paste real messages, phone numbers or contact names.
- For features, open an issue first and describe the problem you want to solve.
- Small fixes (typos, clear bugs) can go straight to a pull request.
- Security problems go through [SECURITY.md](SECURITY.md), not a public issue.

## Setup

You need a Mac, Node.js 18 or newer and older than 26 (Node 24 LTS recommended), and npm.

```sh
git clone https://github.com/tszaks/imessage-mcp.git
cd imessage-mcp
npm install
```

`better-sqlite3` is a native module. If `npm install` fails to build it, check your Node version first.

## Run the tests

```sh
npm test
```

The tests use mocks and temporary databases. They do not read your real Messages database and do not send anything. Run the whole suite before you open a pull request.

To try the server by hand, add it to an MCP client as described in the [README](README.md#client-configuration). Use a test recipient, never a real contact, when you try sending.

## Code style

- **Match the surrounding code.** Same naming, same comment density, same idiom as the file you are editing. There is no formatter, so read the neighbors.
- Keep it simple. The server is plain CommonJS. No new dependencies without discussion.
- Comments explain why, not what.

## Safety rules

- Tools that send, react or delete must stay behind an explicit `confirm: true` (and an approval token for batches and thread deletes). Do not remove or weaken a gate to make a change pass.
- Tools that change data stay behind a release flag in `IMESSAGE_MCP_RELEASES`.
- The server makes no network requests. Do not add any.
- Never commit real messages, phone numbers or contact names, not even in tests. Use fake data.

## Pull request checklist

- [ ] `npm test` passes.
- [ ] New behavior has a test.
- [ ] The change is small and focused.
- [ ] The code matches the surrounding style and adds no dependency.
- [ ] A line added under `[Unreleased]` in [CHANGELOG.md](CHANGELOG.md) for anything a user would notice.
- [ ] README tools table or release flags updated if tools or flags changed.

## Using AI agents to contribute

Fine, as long as you read and understand what you submit and the checklist above is true. [AGENTS.md](AGENTS.md) tells coding agents how this repository works.

## License

By contributing you agree that your contribution is licensed under the [MIT License](LICENSE).
