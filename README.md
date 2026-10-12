# Wapliss WhatsApp for Claude

Connect Claude to WhatsApp through Wapliss. This plugin connects to the hosted
Wapliss MCP service using OAuth; it does not connect Claude to the backend's
GitHub repository and does not require provider tokens or local configuration.

**Provided by [Wapliss](https://wapliss.com/).**

## Setup

1. Install the Wapliss WhatsApp plugin in Claude.
2. Click **Connect** on the Wapliss connector and sign in or create your
   account at [wapliss.com](https://wapliss.com/).
3. Approve Claude's access to your Wapliss account.
4. In the Wapliss panel, create or select your WhatsApp instance and link your
   phone using the QR code or pairing flow shown there.
5. Return to Claude and ask it to check your Wapliss account status.

If authorization expires, use **Connect** again. Never paste Wapliss API keys,
WhatsApp credentials, or session data into Claude.

## How it works

```text
Claude plugin --OAuth 2.1--> https://wapliss.com/mcp
                                   |
                        Wapliss WhatsApp engine
                                   |
                               WhatsApp
```

The code for the web application and backend lives in
[proteuscrypto/Wapliss-Web-y-Backend](https://github.com/proteuscrypto/Wapliss-Web-y-Backend).
That GitHub URL is the source repository, not the MCP connection URL. The
plugin's remote HTTP MCP endpoint is configured in [.mcp.json](.mcp.json).

The current backend uses its own WhatsApp engine. The plugin stores no account
data or WhatsApp credentials; authorization and account management are handled
by Wapliss.

## Available from Claude

The backend currently implements account status and these asynchronous send
operations:

- Text: `send_text_message` with `chat_id` and `text`.
- Image: `send_image` with `chat_id`, a public `url`, and optional `caption`.
- Document: `send_document` with `chat_id`, a public `url`, `file_name`, and
  optional `caption`.
- Audio and video: `send_audio` or `send_video` with `chat_id` and public `url`;
  video also accepts an optional `caption`.
- Location: `send_location` with `chat_id`, `latitude`, `longitude`, and
  optional `name` and `address`.
- Contact card: `send_contact` with `chat_id`, `contact_phone`, and
  `contact_name`.
- Poll: `send_poll` with `chat_id`, `question`, and `options`.
- Forward: `forward_messages` with destination `chat_id` and `message_ids`.
- Account: `get_account_status` reports account state, plan, and usage.

Sending is queued asynchronously. A successful response means Wapliss accepted
the request into its queue; it does not confirm WhatsApp delivery. Claude will
report the queue ID/status returned by the service and must not say a message
was delivered without a later delivery confirmation.

Before sending, Claude confirms when the recipient or content is ambiguous or
was not explicitly requested. Bulk messages require confirmation of the full
recipient list and content. The current MCP cannot look up names, contacts, or
chat history, so provide the recipient's international number or exact chat ID
when it is not already clear from context. Files must have a public URL that
Wapliss can fetch.

## Not available yet

The backend advertises some additional tools, but currently returns
`no disponible hasta la fase 3` for reading chats/messages, looking up
contacts, checking WhatsApp availability, group operations, editing/deleting
messages, and checking instance status. This plugin does not present those
operations as available. Linking the phone and managing the WhatsApp instance
are done in the Wapliss panel.

## Account errors and quotas

Use the status and `action_url` returned by Wapliss. The service may ask you to
finish registration or billing, wait for provisioning, link/reconnect the
phone, or reactivate a canceled account. Quota errors include the service's
message and panel link. HTTP 429 rate-limit errors include a retry duration;
wait that long and do not retry repeatedly. Plan limits and usage are managed
by Wapliss and can be checked with `get_account_status`.

## Project and release status

This plugin is version **0.7.0**. It was aligned with backend source commit
`28468a5` (`refactor: rebrand from Green-API to autonomous motor`, 2026-10-12),
which implements the send queue and account-status tool while leaving several
advertised read/management tools disabled until a later phase.

The public MCP discovery endpoints have been observed, but that alone does not
prove an authenticated Claude OAuth flow or successful WhatsApp delivery. Do
not treat this plugin/backend combination as fully production-verified until
OAuth and representative MCP calls have been tested against the deployed
service. Re-check the backend contract before each release.

## Credits

Plugin developed and maintained by [Wapliss](https://wapliss.com/).