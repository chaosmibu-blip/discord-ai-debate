# Discord AI Debate

Run structured debates between multiple Claude Code instances in a Discord channel. Turn vague questions into actionable plans.

## What It Looks Like

5 AIs each play a different role, debating in a Discord group:
- Pro team researches and drafts proposals
- Con team challenges with data and counterexamples
- Formal debate (Pro-1 opening → Con-1 cross-exam → Con-2 opening → Pro-2 cross-exam → Closing)
- Converge on an action plan both sides agree on

## Prerequisites

- **Claude Code** v2.1+ (requires Claude Max or API key)
- **Discord account** with permission to create Bots
- **5 terminals** (Replit multi-shell, tmux, or multiple terminal tabs)

## Setup

### 1. Create 5 Discord Bots

Go to [Discord Developer Portal](https://discord.com/developers/applications) and create 5 Applications.

For each Application:
1. Bot → Reset Token → save the token
2. Enable **Message Content Intent** (under Privileged Gateway Intents)
3. OAuth2 → URL Generator → check `bot` → check View Channels, Send Messages, Read Message History, Add Reactions
4. Use the generated link to invite the bot to your Discord server

### 2. Install Discord Plugin

```bash
# In Claude Code
/plugin install discord@claude-plugins-official
```

### 3. Create Bot Directories and Startup Scripts

```bash
mkdir -p ~/.claude/channels/discord-bot-{1,2,3,4,5}

echo 'DISCORD_BOT_TOKEN=your_bot1_token' > ~/.claude/channels/discord-bot-1/.env
echo 'DISCORD_BOT_TOKEN=your_bot2_token' > ~/.claude/channels/discord-bot-2/.env
echo 'DISCORD_BOT_TOKEN=your_bot3_token' > ~/.claude/channels/discord-bot-3/.env
echo 'DISCORD_BOT_TOKEN=your_bot4_token' > ~/.claude/channels/discord-bot-4/.env
echo 'DISCORD_BOT_TOKEN=your_bot5_token' > ~/.claude/channels/discord-bot-5/.env
```

Create startup scripts (see full examples in the Chinese README or below):

```bash
# ~/bin/debate1 - Pro-1
#!/bin/bash
export DISCORD_STATE_DIR=~/.claude/channels/discord-bot-1
exec claude --channels plugin:discord@claude-plugins-official \
  --append-system-prompt "$(cat ~/.claude/skills/discord-ai-debate-en.md)

Your debate role is 'Pro-1'."
```

Repeat for debate2 (Con-1), debate3 (Con-2), debate4 (Pro-2), debate5 (Moderator). See README.md for full script templates.

Remember `chmod +x ~/bin/debate*`.

### 4. Enable Bot-to-Bot Communication

Edit `~/.claude/plugins/marketplaces/claude-plugins-official/external_plugins/discord/server.ts`:

Find `if (msg.author.bot) return`, change to `if (msg.author.id === client.user?.id) return`.

### 5. Set Up access.json

Create access.json in each bot directory with all bot and user IDs in allowFrom:

```json
{
  "dmPolicy": "pairing",
  "allowFrom": ["yourUserID", "bot1ID", "bot2ID", "bot3ID", "bot4ID", "bot5ID"],
  "groups": {
    "yourChannelID": {
      "requireMention": false,
      "allowFrom": ["yourUserID", "bot1ID", "bot2ID", "bot3ID", "bot4ID", "bot5ID"]
    }
  }
}
```

### 6. Start the Debate

```
Terminal 1: debate1
Terminal 2: debate2
Terminal 3: debate3
Terminal 4: debate4
Terminal 5: debate5
```

Then @ the Moderator in your Discord channel:

> @Moderator Topic: Should our product prioritize AI features or improve existing UX? Please start the debate.

## Files

- `discord-ai-debate-en.md` — Core skill (English). Copy to `~/.claude/skills/`
- `discord-ai-debate.md` — Core skill (Chinese)
- `README-en.md` — Setup guide (English)
- `README.md` — Setup guide (Chinese)

## License

MIT