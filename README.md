# Wapliss WhatsApp for Claude

Use [WhatsApp](https://www.whatsapp.com/) from Claude through
[Wapliss](https://wapliss.com/). Wapliss handles your account, subscription,
WhatsApp connection, and message limits — you never touch an API token.

**Provided by [Wapliss](https://wapliss.com/).**

## Setup (about 2 minutes)

1. **Install** the Wapliss WhatsApp plugin in Claude.
2. Click **Connect** on the Wapliss connector (or ask Claude anything about
   WhatsApp — it will show you the link).
3. **Sign up** at [wapliss.com](https://wapliss.com/) with Google or email.
   You land directly in your Wapliss dashboard.
4. **Activate your subscription.** As soon as the payment is approved, Wapliss
   creates your WhatsApp instance automatically.
5. **Link your WhatsApp** from the dashboard: scan the QR code with your phone
   (**Settings → Linked devices → Link a device**) or enter the 8-digit
   pairing code instead.
6. You are sent back to Claude, already connected. Done.

There is nothing to copy, paste, or configure. Your WhatsApp provider
credentials are stored encrypted on the Wapliss servers and are never sent to
Claude or to your computer. Under the hood, Wapliss creates a dedicated
[Green-API](https://green-api.com/) WhatsApp instance for each subscriber.

If something is missing (no subscription yet, phone disconnected, payment
failed), Claude tells you what happened and gives you the exact link to fix it.

## How it works

```
Claude ──(OAuth sign-in)──▶ wapliss.com ──▶ account · billing · QR
   │
   └──(MCP over HTTPS, bearer token)──▶ wapliss.com/mcp ──▶ WhatsApp provider
```

The plugin only contains:

| Component | Purpose |
|---|---|
| `.mcp.json` | Points Claude to the hosted Wapliss MCP server `https://wapliss.com/mcp` |
| `skills/wapliss-whatsapp` | Guides Claude on phone numbers, chat IDs, confirmations, and account states |

Authentication uses the standard MCP authorization flow (OAuth 2.1 with PKCE).
You can revoke Claude's access at any time from your Wapliss dashboard.

## Features

- Send text, images, documents, audio, video, locations, contacts, and polls;
  forward messages.
- Edit and delete messages you sent.
- Read recent messages, chats, and full chat history.
- List contacts, inspect and create groups, add or remove group participants,
  and check whether a number has WhatsApp.
- Check your Wapliss account status, plan, and daily usage.

## Usage examples

- "Send a WhatsApp message to +54 9 11 2233-4455 saying I will be late."
- "Check my latest WhatsApp messages."
- "What did the family group say today?"
- "Check whether +1 405 555 0100 has WhatsApp."
- "Send this image to Juan on WhatsApp." (The image needs a public URL.)
- "What's my Wapliss plan and how many messages have I sent today?"

## Plans and limits

Plans, prices, and daily limits are shown at [wapliss.com](https://wapliss.com/).
Limits are enforced by Wapliss on the server, with a daily window that resets
at 00:00 UTC. You manage or cancel your subscription from the dashboard.

## Security and responsible use

- You never handle Green-API tokens; Wapliss stores them encrypted.
- Claude confirms recipients and content before sending anything you did not
  explicitly ask for, and never sends bulk messages without confirming the full
  recipient list. Mass messaging can get a number banned by WhatsApp.

## Development

This repository contains only the Claude plugin: its MCP connection settings
and the skill that guides Claude. The Wapliss web application and backend are
developed separately in
[proteuscrypto/Wapliss-Web-y-Backend](https://github.com/proteuscrypto/Wapliss-Web-y-Backend).

That backend project is still under development and is not complete. It owns
the Wapliss website and account panel, sign-in and Claude OAuth, subscriptions
and billing, WhatsApp instance provisioning and connection, the remote MCP
server, quota enforcement, and account administration. These capabilities are
not implemented in this plugin repository. To test against a staging server,
point `.mcp.json` to its `/mcp` URL.

Do not merge this plugin configuration into the published branch until the
backend is complete and the production MCP endpoint and its authorization flow
are deployed and ready for users.

## Credits

Plugin developed and maintained by [Wapliss](https://wapliss.com/).
