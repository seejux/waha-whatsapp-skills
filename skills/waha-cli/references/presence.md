# Presence API Reference

> **WEBJS ENGINE: NOT SUPPORTED**
> All presence endpoints return HTTP 501 when using the WEBJS engine (the default).
> Response: `{"message":"The method is not implemented by 'WEBJS' engine...","statusCode":501}`
> Presence requires the NOWEB engine. Check `GET /api/sessions` to confirm your engine type before attempting these calls.

Presence endpoints use `/api/$WAHA_SESSION/presence/...`

## Get Presence for Chat

Auto-subscribes if not already subscribed.

```bash
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/presence/{chatId}"
```

Response schema:
```json
{
  "id": "1234567890@c.us",
  "presences": [
    {
      "participant": "1234567890@c.us",
      "lastKnownPresence": "online|offline|typing|recording|paused",
      "lastSeen": 1686568773
    }
  ]
}
```

```bash
# Get current presence status
... | grep -o '"lastKnownPresence":"[^"]*"'

# Get lastSeen timestamp
... | grep -o '"lastSeen":[0-9]*'
```

## Subscribe to Presence

Subscribe to receive presence updates (without fetching current status).

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"chatId":"{chatId}"}' \
  "$WAHA_URL/api/$WAHA_SESSION/presence/subscribe"
```

Response: HTTP 200 (empty body)

## Get All Subscribed Presence

```bash
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/presence"
```

Response schema: `[{id, presences: [{participant, lastKnownPresence, lastSeen}]}]`

```bash
# Get all chat IDs being tracked
... | grep -o '"id":"[^"]*"'

# Find anyone currently online
... | grep '"lastKnownPresence":"online"'

# Find anyone typing
... | grep '"lastKnownPresence":"typing"'
```

## Set Your Own Presence

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"chatId":"{chatId}","presence":"typing"}' \
  "$WAHA_URL/api/$WAHA_SESSION/presence"
```

Valid presence values: `online`, `offline`, `typing`, `recording`, `paused`

Response: HTTP 200 (empty body)

## Presence Values Explained

| Value | Meaning |
|-------|---------|
| `online` | User is online |
| `offline` | User is offline |
| `typing` | User is typing a message |
| `recording` | User is recording audio |
| `paused` | User stopped typing (but hasn't sent) |
