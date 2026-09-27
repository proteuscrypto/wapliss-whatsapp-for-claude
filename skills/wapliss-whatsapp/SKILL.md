---
name: wapliss-whatsapp
description: >
  This skill should be used whenever the user asks to send a WhatsApp message,
  check WhatsApp messages, read WhatsApp chats, send an image/document/video/
  audio/location by WhatsApp, manage WhatsApp groups or contacts, connect
  WhatsApp to Claude, or check their Wapliss account. Trigger phrases include
  "send a WhatsApp message to", "mandale un WhatsApp a", "check my WhatsApp
  messages", "revisa mis mensajes de WhatsApp", "what does the WhatsApp group
  say", "send this image on WhatsApp", "check whether this number has
  WhatsApp", "connect my WhatsApp".
metadata:
  version: "0.6.0"
---

# WhatsApp via Wapliss

The Wapliss MCP tools send and read WhatsApp messages through the user's
Wapliss account. Wapliss handles sign-in, subscription, the WhatsApp
connection (a Green-API instance created automatically for each user), and the
daily message quota on its servers.

## Connecting

- If the Wapliss tools are not available or Claude reports that the connector
  needs authentication, tell the user to click **Connect** on the Wapliss
  connector, or to open https://wapliss.com/ to sign up. Setup is: sign up →
  activate subscription → scan QR code → done.
- **Never** ask the user for Green-API (or any provider) instance IDs, tokens,
  or API keys, and never suggest editing `.env` files. Users do not have or
  need them.

## Account states

When unsure whether the account is ready, call `get_account_status` first.
Tool errors from Wapliss include a state and a link. Relay the message in
plain language and give the link exactly as returned; do not retry in a loop.

| State | What to tell the user |
|---|---|
| `registered` | They need to activate the subscription (billing link). |
| `payment_pending` | Payment is being processed; try again in a moment. |
| `provisioning` | Their WhatsApp is being prepared; try again in a minute. |
| `awaiting_qr` / `disconnected` | Scan the QR in the dashboard: WhatsApp → Settings → Linked devices → Link a device. |
| `past_due` | Messages still work, but they should update the payment method. |
| `canceled` | The subscription ended; they can reactivate from the dashboard. |
| `active` | Everything is ready. |

Linking the phone (QR code or 8-digit pairing code) happens only in the
Wapliss dashboard; send the user to the link Wapliss returns.

## Before sending anything

1. Before sending a message that isn't explicitly and clearly requested by the
   user (e.g. a new recipient not previously discussed, or a large group),
   briefly confirm the recipient and content first.
2. Never send bulk/broadcast messages to multiple contacts without explicit
   confirmation of the full recipient list — this can get the number banned
   by WhatsApp and may violate WhatsApp's terms.

## Phone numbers and chat IDs

- Individual chats: chat ids are the international number with digits only
  plus `@c.us` (e.g. `5491122334455@c.us`). Tools also accept a plain
  international number (`+5491122334455`) and Wapliss normalizes it.
- Group chats: the chatId ends in `@g.us`. If the user only knows the group
  name, look it up in `get_chats` (or `get_contacts`, type `group`) before
  asking for the id; `get_group` returns its participants and details.
- Before messaging a number for the first time, `check_whatsapp` tells you
  whether it has WhatsApp.
- Never invent a phone number or chatId. If the user names a contact by name
  only, look them up with `get_contacts` first; if not found, ask for the
  number.

## Sending messages

- Plain text: `send_text_message`.
- Media (`send_image`, `send_document`, `send_audio`, `send_video`): need a
  **publicly reachable URL** and a file name with extension. A local file path
  alone will not work.
- Location: `send_location` with latitude/longitude as numbers.
- Contact card: `send_contact` (phone number plus name).
- Poll: `send_poll` (question plus 2-12 options).
- Forward: `forward_messages` (source chat plus message ids).

## Editing and deleting

- `edit_message` changes the text of a message the user sent (WhatsApp only
  allows this for a limited time after sending; if it fails, offer a follow-up
  correction instead).
- `delete_message` deletes a message (for everyone when WhatsApp allows it).
- Both are hard to undo: confirm with the user first.

## Reading messages and chats

- `get_last_incoming_messages` / `get_last_outgoing_messages`: recent messages
  across all chats (optionally limited to the last N minutes).
- `get_chats`: list of conversations.
- `get_chat_history`: messages in one chat (needs `chatId`, optional count).
- `get_message`: one message by id.
- `mark_chat_read` / `archive_chat`: housekeeping on a chat.
- Summarize messages in plain language (sender, time, content) rather than
  dumping raw JSON, unless the user asked for raw data.

## Quotas and reliability

If Wapliss reports `quota_exceeded`, tell the user the limit, when it resets
(00:00 UTC), and the upgrade link returned. If a send fails or stays queued,
check `get_account_status` / `get_instance_status` and explain the likely
cause instead of retrying silently.
