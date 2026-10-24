<h1 align="center">iMessage MCP</h1>

<p align="center">
  An MCP server that lets AI agents read, search and (with your confirmation) send iMessages and texts on your Mac, including photo attachments.
</p>

<p align="center">
  <a href="https://github.com/tszaks/imessage-mcp/actions/workflows/ci.yml"><img src="https://github.com/tszaks/imessage-mcp/actions/workflows/ci.yml/badge.svg" alt="CI"></a>
  <a href="https://github.com/tszaks/imessage-mcp/releases/latest"><img src="https://img.shields.io/github/v/release/tszaks/imessage-mcp" alt="Latest release"></a>
  <a href="LICENSE"><img src="https://img.shields.io/github/license/tszaks/imessage-mcp" alt="MIT license"></a>
  <img src="https://img.shields.io/badge/macOS-required-blue" alt="macOS">
  <img src="https://img.shields.io/badge/Node-18%E2%80%9325-green" alt="Node 18 to 25">
</p>

```text
You:    What did Mom text me today?
Agent:  (calls get_conversation with contact "Mom", last 24 hours)
        Mom, 9:14 AM: Are you still coming Sunday?
        Mom, 9:15 AM: [attachment: IMG_2041.HEIC]
You:    Show me the photo.
Agent:  (calls get_attachment, converts HEIC to JPEG, looks at the image)
        It is a photo of a garden with a new fence.
```

This server reads the Messages database on your Mac (`~/Library/Messages/chat.db`) and exposes it to MCP clients such as Claude Code, Codex and Claude Desktop. It is for people who want an agent to help with their texts: catch up on a thread, search for a flight confirmation, review which prospects replied, or draft and send a reply after you approve it.

It exists because other iMessage tools only read the `text` column of `chat.db`. On macOS 14 and later most messages live in the `attributedBody` column instead, so those tools return empty or partial threads. This server decodes `attributedBody` and returns the full message text.

## Features

- **Complete message text.** Decodes Apple's `NSAttributedString` blob so modern messages are not dropped. See [How it works](#how-it-works).
- **Photos the agent can see.** `get_attachment` returns images as image content. HEIC photos are converted to JPEG with macOS `sips`.
- **Name lookup.** Contact names resolve through the local AddressBook databases. The Contacts app does not need to be running.
- **Send behind a confirmation.** `send_message`, `send_message_batch` and `react_to_message` do nothing unless `confirm` is `true`. Batches also need an approval token from a preview call.
- **iMessage and SMS/RCS.** Choose a service, or let `auto` pick one. An opt-in flag retries by SMS/RCS when a blue bubble fails.
- **Outreach helpers.** Structured chat lists, follow-up detection and delivery-failure reports.
- **Dangerous tools stay off by default.** Deleting and UI-driven editing are behind [release flags](#release-flags).

## Install

Requirements:

- A Mac with the Messages app signed in.
- Node.js 18 or newer and older than 26 (`engines` in `package.json`). Node 24 LTS is recommended: the locked native SQLite dependency (`better-sqlite3`) does not build on Node 26.
- A terminal or app with the [macOS permissions](#permissions-and-privacy) below.

The package is not on npm. Install from source:

```bash
git clone https://github.com/tszaks/imessage-mcp.git
cd imessage-mcp
npm install
node index.js   # prints "iMessage MCP server v... running on stdio", then waits for an MCP client
```

Press Ctrl-C to stop it. Normally your MCP client starts it for you.

## Client configuration

Replace `/path/to/imessage-mcp` with the folder you cloned, and use the full path to a supported Node (find it with `which node`; Homebrew on Apple silicon is usually `/opt/homebrew/bin/node`). Desktop apps do not inherit your shell PATH, and a bare `node` can pick an older system Node that crashes the native module.

### Claude Code

```bash
claude mcp add imessage -- /opt/homebrew/bin/node /path/to/imessage-mcp/index.js
```

Or add this to a project `.mcp.json`:

```json
{
  "mcpServers": {
    "imessage": {
      "command": "node",
      "args": ["/path/to/imessage-mcp/index.js"]
    }
  }
}
```

### Codex

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.imessage]
command = "/opt/homebrew/bin/node"
args = ["/path/to/imessage-mcp/index.js"]
```

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "imessage": {
      "command": "/opt/homebrew/bin/node",
      "args": ["/path/to/imessage-mcp/index.js"]
    }
  }
}
```

Restart the client after editing. Grant permissions to the app that launches the server (the client or its terminal), not to Node itself.

## Tools

The server registers these tools (from `index.js`). Tools marked with a flag appear only when that [release flag](#release-flags) is set in `IMESSAGE_MCP_RELEASES`.

| Tool | What it does |
|------|--------------|
| `read_recent_messages` | Read the latest messages across all conversations |
| `search_messages` | Search by contact name, phone number or message text |
| `get_conversation` | Get a thread with a contact or number, optionally limited to the last N hours |
| `list_chats_structured` | List recent chats as JSON: chat ID, handles, last sender, preview, group status, counts |
| `get_conversation_by_chat_id` | Get a thread as structured JSON by chat ID |
| `find_outreach_followups` | Find SMS outreach threads that need review or follow-up, with risk labels |
| `get_attachment` | Return the image or file attached to a message (images are viewable, HEIC converted) |
| `send_message` | Send an iMessage or SMS/RCS. Requires `confirm: true` |
| `detect_message_service` | Recommend iMessage, SMS/RCS or auto for a recipient from local history (best effort) |
| `send_message_batch` | Preview a batch, then send it with `confirm: true` and the approval token |
| `list_delivery_failures` | List failed or pending outgoing messages and SMS/RCS recoveries |
| `list_recent_chats` | List recent conversations by activity |
| `lookup_contact` | Find a phone number or email by contact name |
| `react_to_message` | Send a tapback (love, like, dislike, laugh, emphasize, question). Requires `confirm: true` |
| `delete_messages` | Delete exact message rows from the local database, with a backup first. Flag: `message_mutation_tools` |
| `delete_threads` | Preview and delete whole local threads with an approval token. Flag: `message_mutation_tools` |
| `edit_message` | Experimental: edit a recent outgoing iMessage through the Messages UI. Flag: `experimental_message_ui_actions` |
| `undo_send_message` | Experimental: undo send through the Messages UI. Flag: `experimental_message_ui_actions` |

The `read_receipts` flag adds read-receipt fields to existing results instead of adding a tool.

## Usage

Example prompts:

- "Show me my recent messages."
- "Search my messages for 'flight confirmation'."
- "Find outreach prospects from the last 7 days who replied and need follow-up."
- "Preview this batch of 10 follow-up texts." (returns exact recipients, messages, warnings and an approval token before anything sends)
- "Show recent delivery failures."
- "Show me the photo from message 538516."
- "Send 'Running 10 min late' to +1234567890." (the agent must show you the message first, then pass `confirm: true`)

## Release flags

Release flags are opt-in through the MCP server environment:

```json
{
  "mcpServers": {
    "imessage": {
      "command": "node",
      "args": ["/path/to/imessage-mcp/index.js"],
      "env": {
        "IMESSAGE_MCP_RELEASES": "auto_sms_fallback,cleanup_failed_imessage_after_sms_fallback,message_mutation_tools,experimental_message_ui_actions,read_receipts"
      }
    }
  }
}
```

| Flag | Behavior |
|------|----------|
| `auto_sms_fallback` | `send_message` and `send_message_batch` verify recent outgoing rows after an auto iMessage send. If Messages marks the blue bubble failed and the recipient is phone-based, the MCP retries through SMS/RCS. |
| `cleanup_failed_imessage_after_sms_fallback` | After `auto_sms_fallback` successfully retries by SMS/RCS, the MCP makes a best-effort UI cleanup pass to delete the failed blue iMessage bubble and reports whether cleanup was verified. This never mutates `chat.db` directly. |
| `message_mutation_tools` | Exposes `delete_messages` and `delete_threads`. These tools make a timestamped backup of `chat.db`, `chat.db-wal`, and `chat.db-shm` before direct local Messages database cleanup. `delete_messages` deletes exact message row IDs without confirmation. `delete_threads` requires one exact-batch approval token for the full requested chat ID list. |
| `experimental_message_ui_actions` | Exposes `edit_message` and `undo_send_message`. These tools use Messages UI automation, validate that the target is a recent outgoing iMessage, and verify against `chat.db` after the action. They fail clearly when Messages does not expose the menu action or the message is outside Apple's allowed window. |
| `read_receipts` | Adds read receipt fields to structured conversation results and read status hints to `get_conversation` text output. Supports outgoing 1:1 iMessage and RCS rows when macOS has synced `date_read`. SMS, group chats, and inbound messages are reported as unsupported. Missing `date_read` means not read or unavailable, which can also mean receipts are disabled or not synced. |

### Mutation tools

`delete_messages` accepts exact message ROWIDs:

```json
{ "message_ids": ["538516", "538517"] }
```

It does not require confirmation. It reports deleted IDs, missing IDs, backup location, and any remaining rows found during verification.

`delete_threads` is a two-step exact-batch flow:

```json
{ "chat_ids": [101, 102] }
```

The preview returns one `approval_token` for that ordered list and the thread metadata shown in the preview. To delete, send the same ordered `chat_ids`, `confirm: true`, and that token:

```json
{ "chat_ids": [101, 102], "confirm": true, "approval_token": "..." }
```

Changing the order, list, or token fails the request.

`edit_message` and `undo_send_message` are experimental because Messages does not expose first-class AppleScript commands for those actions. They open the conversation, find the visible outgoing bubble by text snippet, use the contextual menu, then verify the result. They are intended to affect actual Messages behavior, not fake local-only DB edits.

Optional tuning:

```bash
IMESSAGE_MCP_SEND_VERIFY_DELAY_MS=2500
```

This controls how long the MCP waits before checking the local Messages database for the new outgoing row.

## Permissions and privacy

This server can read your whole message history and contacts, and send messages as you. Run it only with agents and clients you trust.

**What it reads (locally)**

- `~/Library/Messages/chat.db`: all messages, handles, chats and attachment metadata. The database is opened read-only except for the two [mutation tools](#mutation-tools).
- `~/Library/Messages/Attachments/`: files you ask for with `get_attachment`.
- `~/Library/Application Support/AddressBook/Sources/*/AddressBook-v22.abcddb`: contact names, numbers and emails, read-only, cached in memory.

**What it can change**

- Send messages and tapbacks through AppleScript, only with `confirm: true`.
- With `message_mutation_tools`: delete rows in `chat.db`. A timestamped backup of `chat.db`, `chat.db-wal` and `chat.db-shm` goes to `~/Library/Messages/mcp-backups/` first.
- With `experimental_message_ui_actions` or `cleanup_failed_imessage_after_sms_fallback`: it opens Messages and clicks through its menus.
- HEIC conversion writes a temporary JPEG to `/tmp`.

**What it sends**

The server makes no network requests of its own and has no analytics. Message text and images you ask for are returned to your MCP client, and from there to whatever model that client uses. Check your client's and model provider's privacy terms. Messages you send go out through the Messages app like any other message.

**macOS permissions you must grant**

| Permission | Where | Needed for |
|------------|-------|------------|
| Full Disk Access | System Settings > Privacy & Security > Full Disk Access | Reading `chat.db` and the AddressBook databases. Grant it to the terminal or app that launches the server. |
| Automation | Prompted on first use (System Settings > Privacy & Security > Automation) | `osascript` control of Messages for `send_message`, `send_message_batch` and `react_to_message`. Also System Events for the UI tools. |
| Accessibility | System Settings > Privacy & Security > Accessibility | Only for `edit_message`, `undo_send_message` and failed-bubble cleanup, which drive the Messages window through System Events. |

Reading needs only Full Disk Access. Sending needs Automation as well.

## How it works

### The attributedBody Fix

Apple's iMessage database (`~/Library/Messages/chat.db`) has two columns for message content:

- `text`: the legacy plain text column (used by older macOS versions)
- `attributedBody`: a serialized `NSAttributedString` blob (used by macOS 14+)

On modern macOS, Apple gradually migrated message storage to `attributedBody` to support rich text, mentions, and formatting. The `text` column is increasingly just a legacy fallback that's often `NULL`.

This server detects messages with `NULL` text and extracts the content from `attributedBody` by parsing the binary `NSTypedStream` format:

1. Finds the `NSString` marker in the binary blob
2. Reads past the type header bytes (`01 94 84 01 2b`)
3. Decodes the length prefix (single-byte for short messages, multi-byte for longer ones)
4. Extracts the UTF-8 text payload

### Attachment Handling

iMessage attachments are stored in `~/Library/Messages/Attachments/` with paths tracked in the `attachment` table. The `get_attachment` tool:

1. Queries the attachment metadata for a given message ID
2. Resolves the `~/Library/Messages/...` path to an absolute path
3. For JPEG/PNG/GIF/WebP: reads the file and returns base64 image content
4. For HEIC (iPhone default): converts to JPEG using macOS `sips` before returning
5. For other files: returns metadata and the file path

## FAQ

**Can an agent send a message without asking me?**
Not through the confirmation tools. `send_message`, `send_message_batch` and `react_to_message` refuse unless `confirm` is `true`, and batches also need the approval token from a preview. The agent still decides when to call them, so keep tool approval on in your client. `delete_messages` needs no confirmation when its flag is enabled.

**Does my data leave my Mac?**
The server itself sends nothing anywhere. Anything a tool returns goes to your MCP client and its model provider.

## Troubleshooting

**"Failed to open iMessage database"**
Grant Full Disk Access to the terminal or app that launches the server: System Settings > Privacy & Security > Full Disk Access.

**Contact lookup returns no results**
Contact resolution reads the macOS AddressBook SQLite databases directly (no Contacts app needed). Make sure Full Disk Access is granted. If a contact was just added, restart the MCP server to refresh the cache.

**Native module crash / "NODE_MODULE_VERSION mismatch"**
Rebuild native dependencies: `npm rebuild`. This happens when your Node.js version changes. Also make sure your Claude Desktop config uses the full path to node (see [Client configuration](#client-configuration)).

**Missing messages from a conversation**
This is exactly the bug this server fixes. Make sure you're running the latest version which includes the `attributedBody` extraction.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Security reports go through [SECURITY.md](SECURITY.md). This project follows the [Code of Conduct](CODE_OF_CONDUCT.md). Changes are listed in [CHANGELOG.md](CHANGELOG.md), and coding agents should read [AGENTS.md](AGENTS.md).

## License

[MIT](LICENSE)
