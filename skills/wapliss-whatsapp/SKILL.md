---
name: wapliss-whatsapp
description: >
  Use this skill when the user wants to send a WhatsApp text, image, document,
  audio, video, location, contact, poll, or forwarded message through Wapliss,
  read their WhatsApp chats, group messages or group members, check their
  Wapliss account status, or asks about WhatsApp capabilities.
  Example requests: "Send a WhatsApp message to Alex", "Read my last messages
  from Laura", "What did they say in the family group?", "Check my Wapliss
  account", "Connect Wapliss to Claude".
metadata:
  version: "0.9.0"
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

Read-only tools (chats and groups of the authenticated account only):

- `get_chats`: optional `type` (`all`, `individual`, `group`), `unread_only`,
  `query` (name, number or last-message text), `limit` (max 50), `cursor`.
  Returns `chat_id`, name, type, last message, date and unread count.
- `get_chat_history`: `chat_id` (a `chat_id` from `get_chats`, an international
  number, or the exact chat name), optional `limit` (max 50) and `before`
  (ISO date or the `before_cursor` of the previous page). Messages come in
  chronological order; in groups each incoming message has `sender_name` and
  `sender_phone`.
- `get_message`: `chat_id`, `message_id`.
- `get_last_incoming_messages`: optional `limit`, `since`, `include_groups`.
- `get_last_outgoing_messages`: optional `limit`, `since`.
- `get_group`: `chat_id` of a group (or its exact name), optional `refresh`.
  Returns name, description, size, admins and participants.

Media URLs must be publicly reachable by Wapliss. Use only the parameters in
the tool schema; do not add unsupported fields such as `quoted_message_id`.

## Important limitations

Searching the contact book, checking whether a number has WhatsApp, managing
groups (create, add or remove participants), editing or deleting messages,
marking chats read, archiving, and checking the instance status are not
available yet; the backend answers `no disponible hasta la fase 3` if called.
If asked for those, explain they are not available through the connector yet.
Do not invent results.

History is what Wapliss has synchronized from the linked phone. Very old chats
may be missing; when a tool returns `chat_not_found` or an empty history, say
so plainly instead of guessing. Media files are not downloaded: describe them
by type and caption.

## Sending safely

- Send only when the user specified the recipient and content clearly. If the
  recipient is ambiguous or the content was not requested, ask for confirmation
  of the exact recipient and content first.
- Never send a bulk message until the user confirms the complete recipient list
  and content. Do not turn a one-to-one request into a broadcast.
- Do not invent phone numbers or chat IDs. To find a recipient by name, use
  `get_chats` with `query`; if several chats match, ask which one.
- Chat IDs may be international numbers, IDs ending in `@c.us` or `@g.us`, or
  the `chat_id` values returned by `get_chats` (`pn:...`, `grp:...`).
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
  panel; use the supplied action URL (the panel lives at
  https://wapliss.com/app/connect) and do not request tokens or QR data in
  chat.
- `quota_exceeded` means the backend rejected the operation at its quota limit.
  Report the returned message and action URL; do not promise a reset time unless
  the response actually includes one.
- HTTP 429 / JSON-RPC rate-limit errors include a retry duration in the error
  message. Wait for that duration if the user retries; never retry in a loop.
- For other tool or provider errors, report the returned error plainly and do
  not claim the message was queued or sent if the tool failed.

## Reading safely

- Read only the chats the user asked about; do not browse unrelated
  conversations. Start with a small `limit` and page with `before` only if the
  user needs more.
- Summarize what you read; quote messages only when the user asks for the
  exact text. Group members' phone numbers are personal data: show them only
  when the user needs them.
- Never forward, send or share content you read unless the user explicitly
  asks for that action and confirms the recipient.

## Language and privacy

Summarize account results and messages in natural language; do not dump raw
JSON unless requested. Never reveal credentials or ask the user to paste them
into Claude.
