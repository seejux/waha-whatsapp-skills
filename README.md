# WAHA WhatsApp Skills for Claude Code

Skills for interacting with the [WAHA WhatsApp HTTP API](https://waha.devlike.pro/) via direct `curl` calls instead of MCP tools. Minimizes token usage by returning only the specific fields you need.

## Skills Included

- **waha-cli** — Send messages, manage chats, groups, contacts, and more via curl + grep/node

## Environment Variables

Three variables are required:

| Variable | Description | Example |
|----------|-------------|---------|
| `WAHA_URL` | Your WAHA server URL | `https://wa.yourdomain.com` |
| `WAHA_KEY` | API key for authentication | `c7312cbfce744cbe...` |
| `WAHA_SESSION` | WhatsApp session name | `default` |

## How to Set Environment Variables in Claude Code

### Option 1: Project `.env` file (recommended)

Create a `.env` file in your project root:

```env
WAHA_URL=https://wa.yourdomain.com
WAHA_KEY=your-api-key-here
WAHA_SESSION=default
```

Claude Code automatically loads `.env` files from the project directory.

### Option 2: Claude Code `settings.json`

Add the variables to your Claude Code settings so they're always available globally.

Open or create `~/.claude/settings.json` (Windows: `%USERPROFILE%\.claude\settings.json`):

```json
{
  "env": {
    "WAHA_URL": "https://wa.yourdomain.com",
    "WAHA_KEY": "your-api-key-here",
    "WAHA_SESSION": "default"
  }
}
```

### Option 3: System environment variables

Add to your shell profile (`.bashrc`, `.zshrc`, or Windows System Environment Variables):

```bash
export WAHA_URL="https://wa.yourdomain.com"
export WAHA_KEY="your-api-key-here"
export WAHA_SESSION="default"
```

## Installation

In Claude Code, run:

```
/plugin marketplace add seejux/waha-whatsapp-skills
```

Then select `waha-skills` → **Install now**.

## Usage

Once installed and env variables are set, just ask Claude naturally:

- "Get my last 10 WhatsApp chats"
- "Send a message to 1234567890 saying Hello"
- "List all groups I'm in"
- "Check if +1234567890 is on WhatsApp"
