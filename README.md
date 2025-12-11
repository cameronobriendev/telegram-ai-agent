# Telegram AI Lead Qualification Bot

An n8n workflow that powers an AI chatbot on Telegram for lead qualification. The bot has natural conversations with users, qualifies their needs, captures their email, and notifies your team with an AI-generated summary.

## Features

- **Telegram Integration** - Responds to messages in your Telegram bot
- **Conversational AI** - Claude-powered chat that qualifies leads naturally
- **Conversation Memory** - PostgreSQL stores full chat history per user
- **Email Capture** - Detects when users provide email and saves lead
- **AI Summarization** - Generates bullet-point summary for sales team
- **Team Notifications** - Slack/webhook notification when lead is captured

## Architecture

```
Telegram User sends message
    ↓
n8n Workflow:
    → Telegram Trigger (receives message)
    → Upsert Conversation (PostgreSQL)
    → Save User Message
    → Load Conversation History
    → Format Messages for Claude
    → Claude AI (HTTP Request)
    → Save Assistant Message
    → Extract Email (Code node)
    → IF has email:
        → Save Lead
        → Mark Completed
        → Summarize Chat (Claude)
        → Notify Team (Slack)
    → Send Telegram Message (reply to user)
```

## Setup

### 1. Create Telegram Bot

1. Open Telegram and search for `@BotFather`
2. Send `/newbot` and follow the prompts
3. Save the **API token** (looks like `123456789:ABCdefGHIjklMNOpqrsTUVwxyz`)

### 2. Database (PostgreSQL/Neon)

Run `schema.sql` to create the required tables:
- `conversations` - Tracks chat sessions (uses Telegram chat_id)
- `messages` - Stores all messages with role (user/assistant)
- `leads` - Captured leads with email and chat summary

### 3. n8n Credentials

Create these credentials in n8n:

**Telegram API:**
- Go to Credentials → New → Telegram API
- Paste your bot token from BotFather

**PostgreSQL:**
- Your database connection details

**HTTP Header Auth (for Anthropic):**
- Header name: `x-api-key`
- Header value: Your Anthropic API key

### 4. Import Workflow

1. Open n8n
2. Import `workflow.json`
3. Update credential references in each node
4. Activate the workflow

### 5. Test Your Bot

1. Open Telegram and find your bot (search by username)
2. Send `/start` then a message
3. The bot should respond!

## Configuration

### System Prompt
Edit the prompt in the "Format Messages" Code node to customize:
- Bot personality
- Qualification questions
- Conversation flow

### Notifications
Replace Slack webhook URL or swap for your preferred notification service.

## Files

| File | Description |
|------|-------------|
| `workflow.json` | n8n workflow (import this) |
| `schema.sql` | PostgreSQL table definitions |
| `system-prompt.md` | Full system prompt reference |

## Key Differences from Web Version

| Aspect | Web Chat | Telegram |
|--------|----------|----------|
| Trigger | Webhook (HTTP POST) | Telegram Trigger |
| Session ID | localStorage generated | Telegram `chat.id` |
| User message | `$json.body.message` | `$json.message.text` |
| Response | HTTP JSON response | Telegram Send Message |

## Gotchas

### SQL Injection
User messages with apostrophes break queries. Always escape:
```javascript
$json.message.text.replace(/'/g, "''")
```

### Telegram API Limits
- Messages max 4096 characters
- 30 messages/second to same chat
- Bot must be started by user first (`/start`)

### Paired Item Data
After HTTP Request nodes, use `$execution.customData` to pass data:
```javascript
// Store early
$execution.customData.set('chat_id', String(value));

// Retrieve later
$execution.customData.get('chat_id');
```

## License

MIT - Use freely for your own projects.

## Credits

Built with [n8n](https://n8n.io), [Telegram Bot API](https://core.telegram.org/bots/api), and [Claude](https://anthropic.com).
