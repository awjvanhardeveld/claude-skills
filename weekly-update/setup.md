^# Slack App Setup Guide

Each PM needs their own Slack app to authenticate the MCP server. Follow these steps once.

## 1. Create a Slack App

1. Go to https://api.slack.com/apps
2. Click **Create New App** → **From scratch**
3. Name it something like `Weekly Update MCP` and select your workspace
4. Click **Create App**

## 2. Configure OAuth Scopes

1. In the left sidebar, go to **OAuth & Permissions**
2. Under **Bot Token Scopes**, add the following:

| Scope | Why |
|---|---|
| `channels:history` | Read messages from public channels |
| `channels:read` | List and resolve channel names |
| `chat:write` | Post the weekly update to a channel |
| `search:read` | Search messages by author and date |
| `users:read` | Resolve your own user ID for author filtering |

## 3. Install the App to Your Workspace

1. Still under **OAuth & Permissions**, scroll up and click **Install to Workspace**
2. Authorize the app
3. Copy the **Bot User OAuth Token** — it starts with `xoxb-`

## 4. Find Your Team ID

Your Slack Team ID can be found in one of two ways:
- From your workspace URL: `https://app.slack.com/client/T01234567` — the `T01234567` part is your Team ID
- Or in **Settings & administration** → **Workspace settings** → scroll to the bottom

## 5. Set Environment Variables

Add these to your shell profile (`~/.zshrc` or `~/.bash_profile`):

```bash
export SLACK_BOT_TOKEN="xoxb-your-token-here"
export SLACK_TEAM_ID="T01234567"
export LINEAR_API_KEY="lin_api_your-key-here"
```

Then reload your shell:

```bash
source ~/.zshrc
```

## 6. Get Your Linear API Key

1. Go to Linear → **Settings** → **Security & access**
2. Scroll to **Personal API keys**
3. Click **New API key**, give it a name (e.g. `Claude MCP`)
4. Copy the key — it starts with `lin_api_`

## Notes

- After adding `chat:write`, you must **reinvite the bot** to any channel you want it to post in (e.g. `/invite @your-bot-name` in `#product-standup`). The bot cannot post to channels it hasn't joined.
- Slack bot tokens expire every **90 days**. Set a reminder to rotate yours.
- The bot token is personal — do not share it or commit it to version control.
- The `.mcp.json` file references these env vars automatically; no further config needed.
