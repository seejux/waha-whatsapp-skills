---
name: waha-cli
description: >
  Make direct WAHA WhatsApp HTTP API calls via curl instead of using MCP tools.
  Use this skill when interacting with WhatsApp through WAHA to minimize token usage -
  curl+grep returns only the specific fields needed rather than full MCP tool responses.
  Triggers: any WhatsApp task (send message, get chats, manage groups, check contacts,
  get presence) when token efficiency matters or MCP tools are unavailable.
---

# WAHA CLI Skill

Make direct API calls to WAHA instead of using MCP tools. Each curl command returns raw JSON - pipe through grep or node to extract only what you need.

## Environment Setup

```bash
export WAHA_URL="http://localhost:3000"   # WAHA_BASE_URL from .env
export WAHA_KEY="your-api-key"            # WAHA_API_KEY from .env
export WAHA_SESSION="default"             # WAHA_SESSION from .env
```

Set these at the start of any session. All commands below use these three variables.

## Auth Header

Every request needs: `-H "X-Api-Key: $WAHA_KEY"`

## Standard Pattern: Save First, Then Extract

Always save to a temp file first. This avoids re-hitting the API and lets you re-grep or inspect the raw JSON freely.

```bash
# Step 1: save response
curl -s -H "X-Api-Key: $WAHA_KEY" "$WAHA_URL/api/..." > /tmp/waha.json

# Step 2: extract what you need
grep -o '"fieldname":"[^"]*"' /tmp/waha.json       # string field
grep -o '"fieldname":[0-9]*' /tmp/waha.json         # numeric field
grep -o '"fieldname":[a-z]*' /tmp/waha.json         # boolean field
node -e "const d=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); d.forEach(x=>console.log(x.id))"
```

For POST/PUT/DELETE, still pipe (no need to save write-op responses):
```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"key":"value"}' "$WAHA_URL/api/..." | grep -o '"id":"[^"]*"'
```

## Reference Files

Load the relevant file for the task at hand:

- **[references/chats.md](references/chats.md)** - get chats, archive, delete, mark read, chat picture
- **[references/messages.md](references/messages.md)** - send text/media/audio/location/contact, edit, delete, pin, react, star
- **[references/groups.md](references/groups.md)** - get/create/manage groups, participants, invite codes, settings
- **[references/contacts.md](references/contacts.md)** - get contact info, check exists, block/unblock, profile picture
- **[references/presence.md](references/presence.md)** - get/set/subscribe presence

Load only the file relevant to the current task.
