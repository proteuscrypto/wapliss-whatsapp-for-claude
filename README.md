<p align="center">
  <img src="https://4144c25c59954fdd6f735c7a0e030ed2.cdn.bubble.io/f1794163595576x849555288109176200/Gemini_Generated_Image_un7sseun7sseun7s-Photoroom.png" alt="Wapliss logo" width="120" />
</p>

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
4. In the Wapliss panel ([wapliss.com/app/connect](https://wapliss.com/app/connect)),
   create or select your WhatsApp instance and link your phone using the QR
   code or pairing flow shown there.
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

The web application and backend are maintained separately from this plugin and
deployed by Wapliss; this repository contains only the Claude plugin. The
plugin's remote HTTP MCP endpoint is configured in [.mcp.json](.mcp.json).

Authorization uses OAuth 2.1 with PKCE (`S256`) and dynamic client
registration. The service publishes its metadata at
`https://wapliss.com/.well-known/oauth-protected-resource` (resource
`https://wapliss.com/mcp`) and
`https://wapliss.com/.well-known/oauth-authorization-server`. Scopes:
`wapliss:mcp`, `whatsapp:read`, `whatsapp:write`.

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
an unavailable-until-a-later-phase error (`no disponible hasta la fase 3`) for
reading chats/messages, looking
up contacts, checking WhatsApp availability, group operations,
editing/deleting messages, and checking instance status. This plugin does not
present those operations as available. Linking the phone and managing the
WhatsApp instance are done in the Wapliss panel.

## Account errors and quotas

Use the status and `action_url` returned by Wapliss. The service may ask you to
finish registration or billing, wait for provisioning, link/reconnect the
phone, or reactivate a canceled account. Quota errors include the service's
message and panel link. HTTP 429 rate-limit errors include a retry duration;
wait that long and do not retry repeatedly. Plan limits and usage are managed
by Wapliss and can be checked with `get_account_status`.

## Project and release status

This plugin is version **0.8.0**. It was re-checked on 2026-11-09 against the
deployed Wapliss MCP server (server version 1.0.0): the same 10 operations are
implemented (`get_account_status` plus the 9 send tools), and the 17 read and
management tools still return the phase-3 unavailable error. Tool names and
parameters are unchanged from 0.7.0.

On 2026-11-09 the OAuth discovery endpoints and the unauthenticated `401` +
`WWW-Authenticate` response from `/mcp` were verified against production. That
does not prove a full authenticated Claude OAuth flow or WhatsApp delivery;
test OAuth and representative MCP calls before treating a release as fully
production-verified, and re-check the backend contract before each release.

## Changelog

- **0.8.0** (2026-11-09): Wapliss logo and branding in the README; documented
  OAuth metadata, scopes and the panel URL; documented the exact
  `no disponible hasta la fase 3` error so Claude recognizes unavailable tools;
  removed the reference to the old backend repository. Backend changes in the
  same window (no effect on the tool contract): Wapliss web sessions no longer
  expire after inactivity, and the WhatsApp engine now runs continuously so
  linked phones stay connected.
- **0.7.0** (2026-10-12): Send queue and account-status tool.

## Credits

Plugin developed and maintained by [Wapliss](https://wapliss.com/).
