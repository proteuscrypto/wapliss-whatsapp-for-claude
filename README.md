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
5. **Scan the QR code** shown in the dashboard with WhatsApp on your phone:
   **Settings → Linked devices → Link a device**.
6. You are sent back to Claude, already connected. Done.

There is nothing to copy, paste, or configure. Your WhatsApp provider
credentials are stored encrypted on the Wapliss servers and are never sent to
Claude or to your computer.

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

- Send text, image, document, audio, video, location, and contact (vCard)
  messages.
- Read message history and chats.
- React to and delete messages.
- List and inspect groups and contacts, check whether a number has WhatsApp,
  and block or unblock contacts.
- Check your Wapliss account status, plan, and daily usage.

> **Editing messages:** WhatsApp does not support editing an already-sent
> message through the API. Claude can delete it (`delete_message`) and send a
> corrected replacement.

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

- You never handle provider tokens; Wapliss stores them encrypted.
- Claude confirms recipients and content before sending anything you did not
  explicitly ask for, and never sends bulk messages without confirming the full
  recipient list. Mass messaging can get a number banned by WhatsApp.

## Development

`dev/stdio-server/` contains a dependency-free local MCP server that talks to a
single, manually configured UltraMsg instance. It is **only** for testing the
tool surface during development and is not used by the published plugin.

```bash
cp dev/stdio-server/.env.example dev/stdio-server/.env   # add your test credentials
claude mcp add wapliss-dev -- node dev/stdio-server/index.js
```

Never commit `.env`. If a token is ever exposed, regenerate it in the UltraMsg
dashboard.

## Credits

Plugin developed and maintained by [Wapliss](https://wapliss.com/).
