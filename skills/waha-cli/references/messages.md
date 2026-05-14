# Messages API Reference

## Response Structure Warning

All WAMessage objects contain a `_data` field with 100+ raw WebJS internal fields. grep on `id`, `from`, `to` will return duplicates from `_data`. `body` is safe to grep (only at root). For any field that also appears in `_data`, use node.

All `/api/sendX` endpoints require `"session": "$WAHA_SESSION"` in the request body.

All send operations return a WAMessage: `{id, timestamp, from, fromMe, source, to, participant, body, hasMedia, mediaUrl, ack, ackName, media{url,mimetype,filename}, location{latitude,longitude,live,name,address}, vCards[], replyTo{id,participant,body}, _data{}}`

```bash
# Get sent message ID (safe — top-level id is a plain string)
... | node -e "const m=JSON.parse(require('fs').readFileSync(0,'utf8')); console.log(m.id)"

# grep OK for body (root only, _data.body matches too but same value)
... | grep -o '"body":"[^"]*"'
```

---

## Send Text Message

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d "{\"chatId\":\"1234567890@c.us\",\"text\":\"Hello!\",\"session\":\"$WAHA_SESSION\"}" \
  "$WAHA_URL/api/sendText"
```

Optional body fields: `"reply_to":"msgId"`, `"linkPreview":true`, `"linkPreviewHighQuality":false`

---

## Send Image

```bash
# From URL
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d "{\"chatId\":\"1234567890@c.us\",\"file\":{\"url\":\"https://...\",\"mimetype\":\"image/jpeg\"},\"caption\":\"optional\",\"session\":\"$WAHA_SESSION\"}" \
  "$WAHA_URL/api/sendImage"

# From base64
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d "{\"chatId\":\"1234567890@c.us\",\"file\":{\"data\":\"base64==\",\"mimetype\":\"image/jpeg\",\"filename\":\"photo.jpg\"},\"session\":\"$WAHA_SESSION\"}" \
  "$WAHA_URL/api/sendImage"
```

---

## Send File (Document)

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d "{\"chatId\":\"1234567890@c.us\",\"file\":{\"url\":\"https://...\",\"mimetype\":\"application/pdf\",\"filename\":\"doc.pdf\"},\"session\":\"$WAHA_SESSION\"}" \
  "$WAHA_URL/api/sendFile"
```

---

## Send Video

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d "{\"chatId\":\"1234567890@c.us\",\"file\":{\"url\":\"https://...\",\"mimetype\":\"video/mp4\"},\"session\":\"$WAHA_SESSION\"}" \
  "$WAHA_URL/api/sendVideo"
```

---

## Send Audio / Voice

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d "{\"chatId\":\"1234567890@c.us\",\"file\":{\"url\":\"https://...\",\"mimetype\":\"audio/ogg\"},\"convert\":true,\"session\":\"$WAHA_SESSION\"}" \
  "$WAHA_URL/api/sendVoice"
```

`"convert":true` re-encodes audio to WhatsApp-compatible format.

---

## Send Location

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d "{\"chatId\":\"1234567890@c.us\",\"latitude\":40.7128,\"longitude\":-74.006,\"title\":\"New York\",\"session\":\"$WAHA_SESSION\"}" \
  "$WAHA_URL/api/sendLocation"
```

---

## Send Contact (vCard)

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d "{\"chatId\":\"1234567890@c.us\",\"contacts\":[{\"vcard\":\"BEGIN:VCARD\\nVERSION:3.0\\nFN:Jane Doe\\nTEL:+1234567890\\nEND:VCARD\"}],\"session\":\"$WAHA_SESSION\"}" \
  "$WAHA_URL/api/sendContactVcard"
```

---

## Delete Message

```bash
curl -s -X DELETE -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/chats/{chatId}/messages/{messageId}"
```

Message ID format: `false_1234567890@c.us_AAAAAAAAAAAAAAAAAAAA`

---

## Edit Message

```bash
curl -s -X PUT -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"text":"Updated text","linkPreview":true}' \
  "$WAHA_URL/api/$WAHA_SESSION/chats/{chatId}/messages/{messageId}"
```

Body: `{text* (required), linkPreview, linkPreviewHighQuality}`

---

## Pin Message

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"duration":86400}' \
  "$WAHA_URL/api/$WAHA_SESSION/chats/{chatId}/messages/{messageId}/pin"
```

Body is only `{duration}` — chatId/messageId are path params. Default: 86400s (24h).

---

## Unpin Message

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/chats/{chatId}/messages/{messageId}/unpin"
```

---

## React to Message

```bash
curl -s -X PUT -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d "{\"messageId\":\"{messageId}\",\"reaction\":\"👍\",\"session\":\"$WAHA_SESSION\"}" \
  "$WAHA_URL/api/reaction"
```

Use `"reaction":""` to remove.

---

## Star / Unstar Message

```bash
curl -s -X PUT -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d "{\"messageId\":\"{messageId}\",\"chatId\":\"{chatId}\",\"star\":true,\"session\":\"$WAHA_SESSION\"}" \
  "$WAHA_URL/api/star"
```

---

## Fields safe to grep vs. use node

| Field | Safe to grep? | Note |
|-------|--------------|-------|
| `body` | Yes | Appears at root only |
| `hasMedia` | Yes | Root only |
| `ackName` | Yes | Root only |
| `id` | No | Also in `_data.id{}` (object) |
| `from` / `to` | No | Also in `_data.from{}` / `_data.to{}` |
| `timestamp` | No | Also in `_data.t` and `_data.serverStoreTimeMicros` |
