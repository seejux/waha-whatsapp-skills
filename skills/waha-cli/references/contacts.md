# Contacts API Reference

Two URL patterns exist — legacy flat path and new session-scoped path:
- Legacy: `/api/contacts?session=$WAHA_SESSION` (most endpoints)
- Session-scoped: `/api/$WAHA_SESSION/contacts/{id}` (GET/PUT single contact)

## Response Structure Note

Contact objects are mostly flat with no `_data` blob. `id` is a plain string (`number@c.us`) — grep is safe.

**Watch out**: for business accounts, a `businessProfile` blob is present with ~25 fields (location, hours, description, categories, etc). This won't cause grep duplicates for standard fields but adds significant size.

Field name casing: `pushname` is **lowercase** (not `pushName`).

Real schema: `{businessProfile{} (if business), id, number, isBusiness, isEnterprise, labels[], name, pushname, shortName, statusMute, type, verifiedLevel, verifiedName, isMe, isUser, isGroup, isWAContact, isMyContact, isBlocked}`

## Get Contact Info

```bash
# Legacy (most common)
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/contacts?contactId=1234567890@c.us&session=$WAHA_SESSION" > /tmp/waha.json

# Session-scoped (also works)
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/$WAHA_SESSION/contacts/1234567890@c.us" > /tmp/waha.json
```

```bash
# Get contact name
grep -o '"name":"[^"]*"' /tmp/waha.json | head -1

# Get display name (note lowercase n)
grep -o '"pushname":"[^"]*"' /tmp/waha.json

# Check if blocked
grep -o '"isBlocked":[a-z]*' /tmp/waha.json

# Check if business account
grep -o '"isBusiness":[a-z]*' /tmp/waha.json
```

## Get All Contacts

```bash
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/contacts/all?session=$WAHA_SESSION&limit=100&sortBy=name&sortOrder=asc" > /tmp/waha.json
```

Optional params: `sortBy=id|name`, `sortOrder=asc|desc`, `limit=100`, `offset=0`

```bash
# Clean: list all contacts with ID and display name
node -e "const d=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); d.forEach(c=>console.log(c.id, c.name||c.pushname||'(no name)'))"

# Clean: find contact by partial number
node -e "const d=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); d.filter(c=>c.id.includes('1234567890')).forEach(c=>console.log(c.id, c.name, c.pushname))"

# Clean: business contacts only
node -e "const d=JSON.parse(require('fs').readFileSync('/tmp/waha.json','utf8')); d.filter(c=>c.isBusiness).forEach(c=>console.log(c.id, c.name))"

# grep OK: id is plain string, @c.us suffix is safe
grep -o '"id":"[^"]*@c\.us"' /tmp/waha.json

# Find specific contact by number
grep "1234567890" /tmp/waha.json
```

## Check If Number Exists on WhatsApp

```bash
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/contacts/check-exists?phone=1234567890&session=$WAHA_SESSION" | grep -o '"numberExists":[a-z]*'

# Get chat ID if exists
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/contacts/check-exists?phone=1234567890&session=$WAHA_SESSION" | grep -o '"chatId":"[^"]*"'
```

Response schema: `{numberExists: true|false, chatId: "1234567890@c.us"}`

## Get Contact About (Status Text)

```bash
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/contacts/about?contactId=1234567890@c.us&session=$WAHA_SESSION" | grep -o '"about":"[^"]*"'
```

## Get Contact Profile Picture

```bash
curl -s -H "X-Api-Key: $WAHA_KEY" \
  "$WAHA_URL/api/contacts/profile-picture?contactId=1234567890@c.us&session=$WAHA_SESSION&refresh=false" | grep -o '"url":"[^"]*"'
```

## Block Contact

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d "{\"contactId\":\"1234567890@c.us\",\"session\":\"$WAHA_SESSION\"}" \
  "$WAHA_URL/api/contacts/block"
```

Response: HTTP 200 (empty body)

## Unblock Contact

```bash
curl -s -X POST -H "X-Api-Key: $WAHA_KEY" -H "Content-Type: application/json" \
  -d "{\"contactId\":\"1234567890@c.us\",\"session\":\"$WAHA_SESSION\"}" \
  "$WAHA_URL/api/contacts/unblock"
```

Response: HTTP 200 (empty body)
