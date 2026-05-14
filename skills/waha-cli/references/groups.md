# Groups API Reference

All group endpoints use path: `/api/$WAHA_SESSION/groups/...`

Group ID format: `123456789012345678@g.us`

## Response Structure Warning

**Both the list and single-group endpoints return the same shape** — a chat-like object, NOT the flat `{id, subject, description, ...}` structure from the Swagger spec.

Real schema: `{groupMetadata{}, id{server,user,_serialized}, name, isGroup, isReadOnly, unreadCount, timestamp, archived, pinned, isLocked, isMuted, muteExpiration, lastMessage{_data{},id,body,...}}`

Critical issues:
- `id` is an **object** `{server, user, _serialized}` — NOT a string. Use `id._serialized` for the group ID string.
- `name` is at root. `subject` does NOT appear at root — it's inside `groupMetadata.subject`.
- `groupMetadata` is a ~35-field raw blob. `description`, `invite`, `membersCanSendMessages` live inside it, not at root.
- `lastMessage` contains a `_data` blob (same as chats overview).
- grep on `"id"` will also match inside `groupMetadata.id` and `lastMessage._data.id`.

## Get All Groups

```bash
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/groups?limit=100&sortBy=id&sortOrder=asc&exclude=participants" \
  > /tmp/waha.json
```

Optional params: `sortBy=id|name`, `sortOrder=asc|desc`, `limit=100`, `offset=0`, `exclude=participants`

```bash
# Clean: group IDs and names (id._serialized is the string form)
node -e "const d=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); d.forEach(g=>console.log(g.id._serialized, g.name))" /tmp/waha.json

# Clean: with unread count
node -e "const d=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); d.filter(g=>g.unreadCount>0).forEach(g=>console.log(g.id._serialized, g.name, 'unread:'+g.unreadCount))"

# Clean: admin-only message setting (inside groupMetadata)
node -e "const d=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); d.forEach(g=>console.log(g.id._serialized, g.name, 'announceOnly:'+g.groupMetadata.announce))"

# grep OK: @g.us suffix is unique to group IDs in _serialized
grep -o '"_serialized":"[^"]*@g\.us"' /tmp/waha.json

# grep OK: name appears only at root
grep -o '"name":"[^"]*"' /tmp/waha.json
```

## Get Group Count

```bash
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/groups/count" > /tmp/waha.json

grep -o '"count":[0-9]*' /tmp/waha.json
```

## Get Group Info

Same schema as the list — same blob structure, same `id` object.

```bash
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}" > /tmp/waha.json

# Get group name and description
node -e "const g=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); console.log('name:', g.name, '\ndesc:', g.groupMetadata.desc||'(none)')"

# Check admin-only settings
node -e "const g=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); console.log('announceOnly:', g.groupMetadata.announce, 'restrictInfo:', g.groupMetadata.restrict)"
```

## Create Group

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"name":"Group Name","participants":[{"id":"1234567890@c.us"},{"id":"0987654321@c.us"}]}' \
  "$WAHA_URL/api/$WAHA_SESSION/groups" | node -e "const g=JSON.parse(require('fs').readFileSync(0,'utf8')); console.log(g.id._serialized)"
```

## Update Group Subject (Name)

```bash
curl -s -X PUT -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"subject":"New Group Name"}' \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/subject"
```

## Update Group Description

```bash
curl -s -X PUT -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"description":"New description"}' \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/description"
```

## Leave Group

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/leave"
```

## Delete Group

```bash
curl -s -X DELETE -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}"
```

## Get Participants

Use `/v2` for a typed response:

```bash
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/participants/v2" > /tmp/waha.json
```

Response schema: `[{id, pn, role}]`
- `id`: `number@c.us` (plain string — grep safe)
- `pn`: phone number string (same value as id usually)
- `role`: `"participant"`, `"admin"`, or `"superadmin"`

```bash
# List all participant IDs
grep -o '"id":"[^"]*"' /tmp/waha.json

# List admins only
node -e "const d=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); d.filter(p=>p.role==='admin'||p.role==='superadmin').forEach(p=>console.log(p.id, p.role))"

# Check if specific number is a participant
grep "1234567890" /tmp/waha.json
```

## Add Participants

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"participants":[{"id":"1234567890@c.us"}]}' \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/participants/add"
```

## Remove Participants

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"participants":[{"id":"1234567890@c.us"}]}' \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/participants/remove"
```

## Promote to Admin

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"participants":[{"id":"1234567890@c.us"}]}' \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/admin/promote"
```

## Demote from Admin

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"participants":[{"id":"1234567890@c.us"}]}' \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/admin/demote"
```

## Get Invite Code

```bash
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/invite-code"
```

Response: raw string (not JSON object) — just the code, e.g. `"AbCdEfGhIj1234567890"`

## Revoke Invite Code

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/invite-code/revoke"
```

Response: new invite code string

## Join Group via Invite

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"code":"https://chat.whatsapp.com/AbCdEfGhIj"}' \
  "$WAHA_URL/api/$WAHA_SESSION/groups/join" | grep -o '"id":"[^"]*"'
```

## Group Picture

```bash
# Get picture URL  =>  Response: {url: "https://..."}
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/picture" | grep -o '"url":"[^"]*"'

# Set picture from URL  =>  Response: {success: true}
curl -s -X PUT -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"file":{"url":"https://example.com/image.jpg"}}' \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/picture"

# Delete picture  =>  Response: {success: true}
curl -s -X DELETE -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/picture"
```

## Group Settings

```bash
# Get/set who can send messages
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/settings/security/messages-admin-only" | grep -o '"adminsOnly":[a-z]*'

curl -s -X PUT -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"adminsOnly":true}' \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/settings/security/messages-admin-only"

# Get/set who can edit group info
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/settings/security/info-admin-only" | grep -o '"adminsOnly":[a-z]*'

curl -s -X PUT -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d '{"adminsOnly":true}' \
  "$WAHA_URL/api/$WAHA_SESSION/groups/{groupId}/settings/security/info-admin-only"
```
