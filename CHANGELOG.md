# Changelog

All notable changes to iMessage MCP are listed here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project uses [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- Documentation: rewritten README with a tools table, client configuration for Claude Code, Codex and Claude Desktop, and a permissions and privacy section. Added CONTRIBUTING, SECURITY, Code of Conduct, AGENTS, issue and pull request templates, and a CI workflow.

## [1.5.1] - 2026-06-05

### Added
- Gated read receipt metadata (`read_receipts` release flag).

### Changed
- Node support is capped to versions the native SQLite dependency supports (18 up to, but not including, 26).

## [1.5.0] - 2026-06-01

### Added
- Message mutation tools: `delete_messages` and `delete_threads` (`message_mutation_tools` flag), with a backup of the Messages database first.
- Experimental `edit_message` and `undo_send_message` (`experimental_message_ui_actions` flag).

## [1.4.2] - 2026-05-27

### Added
- `cleanup_failed_imessage_after_sms_fallback` release flag.

## [1.4.1] - 2026-05-26

### Added
- `auto_sms_fallback` release flag: verified SMS/RCS retry after a failed iMessage send.

## [1.4.0] - 2026-05-21

### Added
- SMS outreach tools: `find_outreach_followups`, `send_message_batch` with approval tokens, `detect_message_service`, `list_delivery_failures`, `list_chats_structured`, `get_conversation_by_chat_id`.

## [1.3.0] - 2026-03-17

### Added
- SMS (green bubble) support with automatic fallback.

## [1.1.0]

Initial release as Multimodal iMessage MCP: conversation reading with `attributedBody` decoding, search, image attachments, sending, contact lookup and reactions. Contact lookup reads the AddressBook SQLite databases directly.

[Unreleased]: https://github.com/tszaks/imessage-mcp/compare/v1.5.1...HEAD
[1.5.1]: https://github.com/tszaks/imessage-mcp/compare/v1.5.0...v1.5.1
[1.5.0]: https://github.com/tszaks/imessage-mcp/compare/v1.4.2...v1.5.0
[1.4.2]: https://github.com/tszaks/imessage-mcp/compare/v1.4.1...v1.4.2
[1.4.1]: https://github.com/tszaks/imessage-mcp/compare/v1.4.0...v1.4.1
[1.4.0]: https://github.com/tszaks/imessage-mcp/compare/v1.3.0...v1.4.0
[1.3.0]: https://github.com/tszaks/imessage-mcp/releases/tag/v1.3.0
