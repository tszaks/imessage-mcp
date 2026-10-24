# Security policy

## Supported versions

Only the latest release of iMessage MCP gets security fixes.

| Version | Supported |
|---|---|
| 1.x (latest) | Yes |
| Older | No |

## Reporting a vulnerability

Please report security problems privately, not in a public issue.

Use GitHub private vulnerability reporting: open the **Security** tab of this repository and choose **Report a vulnerability**. If that is not available to you, email tyler@szakacsmedia.com.

Please include:

- What you found and what an attacker could do with it.
- Steps to reproduce.
- Your macOS version, Node version and server version.

Do not include real messages or contact details in a report.

You can expect a first reply within a few days. Once a fix is released we will credit you in the changelog unless you prefer to stay anonymous.

## Sensitive surface

This server can read all of a user's messages, attachments and contacts, and can send messages and tapbacks as them. With release flags enabled it can also delete local message data and drive the Messages window. These areas matter most:

- Any way to send, react or delete without `confirm: true` or the approval token.
- Any way to run a mutation tool without its release flag.
- Shell or AppleScript injection through a recipient, message text or snippet (the server builds `osascript` and `sips` commands).
- Reading or returning files outside `~/Library/Messages` through `get_attachment`.
- Any network request. The server makes none.

See the [Permissions and privacy](README.md#permissions-and-privacy) section of the README.
