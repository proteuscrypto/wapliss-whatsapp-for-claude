---
name: wapliss-whatsapp
description: >
  Use this skill when the user wants to send a WhatsApp text, image, document,
  audio, video, location, contact, poll, or forwarded message through Wapliss,
  check their Wapliss account status, or asks about WhatsApp capabilities.
  Example requests: "Send a WhatsApp message to Alex", "Send this PDF on
  WhatsApp", "Check my Wapliss account", "Connect Wapliss to Claude".
metadata:
  version: "0.7.0"
---

# Wapliss WhatsApp

Use the Wapliss MCP connector. Reply in the user's language. Authenticate with
the connector's **Connect** flow; never ask for API keys, WhatsApp credentials,
or local configuration. The connector targets Wapliss's MCP service, not its
GitHub source repository.

## Available tools

The current backend implements these MCP operations:

- `get_account_status`: read account state, plan, current usage, and quota.
- `send_text_message`: `chat_id`, `text`.
- `send_image`: `chat_id`, `url`, optional `caption`.
- `send_document`: `chat_id`, `url`, `file_name`, optional `caption`.
- `send_audio`: `chat_id`, `url`.
- `send_video`: `chat_id`, `url`, optional `caption`.
- `send_location`: `chat_id`, `latitude`, `longitude`, optional `name` and
  `address`.
- `send_contact`: `chat_id`, `contact_phone`, `contact_name`.
- `send_poll`: `chat_id`, `question`, `options`.
- `forward_messages`: `chat_id`, `message_ids`.

Media URLs must be publicly reachable by Wapliss. Use only the parameters in
the tool schema; do not add unsupported fields such as `quoted_message_id`.

## Important limitations

The backend currently advertises additional tools for reading chats, contacts,
groups, editing/deleting messages, and checking instance status, but returns
an unavailable-until-a-later-phase error when they are called. Do not call
those tools or claim those capabilities work. If asked to read messages, search contacts,
inspect/manage groups, edit/delete messages, check the WhatsApp connection, or
check whether a number has WhatsApp, explain that this operation is not yet
available through the current Claude connector. Do not invent results.

## Sending safely

- Send only when the user specified the recipient and content clearly. If the
  recipient is ambiguous or the content was not requested, ask for confirmation
  of the exact recipient and content first.
- Never send a bulk message until the user confirms the complete recipient list
  and content. Do not turn a one-to-one request into a broadcast.
- Do not invent phone numbers or chat IDs. If a name cannot be resolved from
  context, ask the user for the international number or exact chat ID; the
  current MCP cannot look up contacts or chats for you.
- Chat IDs may be international numbers or IDs ending in `@c.us` or `@g.us`.
- Confirm the recipient and content before `forward_messages`; forwarding is a
  send and the current schema accepts only `chat_id` and `message_ids`.
- The MCP enqueues messages asynchronously. A successful tool response means
  the request was queued, not that WhatsApp delivered it. Tell the user it was
  queued and include its queue ID/status when returned. Never claim delivery
  unless a later verified status says it was delivered.

## Account and errors

- Use `get_account_status` for the account state and quota. If OAuth is missing,
  expired, or revoked, ask the user to click **Connect** again.
- In MCP tool errors, follow `result.wapliss_error.action_url` exactly when it
  is present. Account errors use codes such as `account_registered`,
  `account_payment_pending`, `account_provisioning`, `account_awaiting_qr`,
  `account_disconnected`, `account_past_due`, and `account_canceled`; explain
  the accompanying message and direct the user to the supplied action URL.
- `instance_not_connected` means the user must connect WhatsApp in the Wapliss
  panel; use the supplied action URL and do not request tokens or QR data in
  chat.
- `quota_exceeded` means the backend rejected the operation at its quota limit.
  Report the returned message and action URL; do not promise a reset time unless
  the response actually includes one.
- HTTP 429 / JSON-RPC rate-limit errors include a retry duration in the error
  message. Wait for that duration if the user retries; never retry in a loop.
- For other tool or provider errors, report the returned error plainly and do
  not claim the message was queued or sent if the tool failed.

## Language and privacy

Summarize account results in natural language; do not dump raw JSON unless
requested. Never reveal credentials or ask the user to paste them into Claude.