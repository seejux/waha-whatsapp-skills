# Chats API Reference

## Response Structure Warning

Chat objects contain two large raw WebJS blobs — `_data` inside `lastMessage` (~100 fields) and `_chat` (~50 fields including a nested `lastMessage` copy). **grep will match fields inside these blobs too**, producing duplicate results. Use node for clean top-level-only extraction.

Also: `unreadCount` lives at `_chat.unreadCount`, not at the chat root. `_chat.id` is an object `{server, user, _serialized}`, not a string.

---

## Get Chats Overview

```bash
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/chats/overview?limit={limit}&offset={offset}" \
  > /tmp/waha.json
```

Params: `limit` (how many chats to fetch), `offset` (for pagination, default 0).
Choose `limit` based on the task — 20 for a quick overview, 50+ to find a specific chat, 100+ for full inbox analysis.

Response schema: `[{id, name, picture, lastMessage{id,timestamp,from,fromMe,body,hasMedia,ack,ackName,_data}, _chat{id{},name,isGroup,unreadCount,timestamp,archived,pinned,isMuted}}]`

```bash
# Clean: top-level chat IDs and names only (no blob noise)
node -e "const d=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); d.forEach(c=>console.log(c.id, c.name))"

# Clean: unread chats only (unreadCount is in _chat, not root)
node -e "const d=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); d.filter(c=>c._chat&&c._chat.unreadCount>0).forEach(c=>console.log(c.id, c.name, 'unread:'+c._chat.unreadCount))"

# Clean: last message preview per chat
node -e "const d=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); d.forEach(c=>console.log(c.id,'|',c.lastMessage&&c.lastMessage.body?c.lastMessage.body.substring(0,60):'(no msg)'))"

# Clean: group chats only
node -e "const d=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); d.filter(c=>c._chat&&c._chat.isGroup).forEach(c=>console.log(c.id, c.name))"

# grep OK for picture URL (only appears once at root)
grep -o '"picture":"[^"]*"' /tmp/waha.json
```

---

## Get Messages in Chat

```bash
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/chats/{chatId}/messages?limit={limit}&offset={offset}" \
  > /tmp/waha.json
```

Params: `limit` (how many messages), `offset` (pagination), `downloadMedia=false`.
Choose `limit` based on the task — 20 for recent context, 50 for a conversation summary, 100+ for full history analysis.

Response schema: `[{id, timestamp, from, fromMe, source, to, participant, body, hasMedia, mediaUrl, ack, ackName, media{url,mimetype,filename}, location{latitude,longitude,live,name,address}, vCards[], replyTo{id,participant,body}, _data{}}]`

Note: `_data` contains raw WebJS internals — grep on `body` will only match once per message (body is at root level), but `id` and `from` appear in `_data` too.

```bash
# Clean: message list with sender direction and body
node -e "const d=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); d.forEach(m=>console.log(m.fromMe?'[ME]':'[IN]', m.timestamp, m.body||'(media)'))"

# Clean: media messages only
node -e "const d=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); d.filter(m=>m.hasMedia).forEach(m=>console.log(m.id, m.mediaUrl))"

# Clean: messages with their IDs (for delete/react operations)
node -e "const d=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); d.forEach(m=>console.log(m.id, '|', (m.body||'').substring(0,40)))"

# grep OK: body appears only at root, safe to use
grep -o '"body":"[^"]*"' /tmp/waha.json

# grep NOISY: id appears in _data too — use node instead
# AVOID: grep -o '"id":"[^"]*"'  (returns duplicates from _data)
```

---

## Get Single Message

```bash
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/chats/{chatId}/messages/{messageId}" > /tmp/waha.json
```

Same schema as WAMessage above (single object, not array).

```bash
node -e "const m=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); console.log('from:', m.from, 'body:', m.body, 'ack:', m.ackName)"
```

---

## Mark Chat as Read

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"chatId":"{chatId}","messages":{count},"days":{days}}' \
  "$WAHA_URL/api/$WAHA_SESSION/chats/{chatId}/messages/read"
```

`messages` and `days` control how far back to mark as read. Use judgment — `messages:50,days:7` for a week of backlog, `messages:200,days:30` for a full cleanup.

Response: `{ids: ["msg_id_1", "msg_id_2"]}`

---

## Mark Chat as Unread

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/chats/{chatId}/unread"
```

---

## Get Chat Picture

```bash
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/chats/{chatId}/picture?refresh=false" | grep -o '"url":"[^"]*"'
```

Response: `{url: "https://..."}` — grep is safe here (flat object, no nesting).

---

## Archive / Unarchive Chat

```bash
# Archive
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/chats/{chatId}/archive"

# Unarchive
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/chats/{chatId}/unarchive"
```

---

## Clear Chat Messages

```bash
curl -s -X DELETE -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/chats/{chatId}/messages"
```

WARNING: irreversible.

---

## Chat ID Formats

- Individual: `1234567890@c.us` (may also appear as `@lid` for linked devices)
- Group: `123456789012345678@g.us`
