# User Account Creation API

Create a new user account via the signup API endpoint.

## Endpoint

```
POST https://gw.staging-k8s.hellofresh.io/api/signup?country={MARKET}
Content-Type: application/json
```

## Request Body

```json
{
  "email": "test-user-1735564800@hellofresh.com",
  "password": "qwerty123",
  "firstName": "Test",
  "lastName": "User"
}
```

## Email Generation Strategy

Generate unique email addresses using Unix timestamp:

```bash
TIMESTAMP=$(date +%s)
EMAIL="test-user-${TIMESTAMP}@hellofresh.com"
```

**Why this works:**
- Timestamp ensures uniqueness
- Uses hellofresh.com domain (accepted by staging)
- No special characters that could cause issues
- Easy to identify as test data

**Examples:**
- `test-user-1735564800@hellofresh.com`
- `test-user-1735564956@hellofresh.com`

## Password Policy

Use consistent test password: `qwerty123`

**Requirements (staging):**
- Minimum 6 characters
- No special complexity requirements in staging

## Response Format

### Success (200 OK)

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "refresh_token_here...",
  "expires_in": 3600,
  "token_type": "Bearer",
  "customerId": "12345678",
  "customer_id": "12345678",
  "email": "test-user-1735564800@hellofresh.com"
}
```

**Key Fields:**
- `access_token` - Use for subsequent API calls (Authorization: Bearer {token})
- `customerId` or `customer_id` - User identifier (may vary by market)
- `expires_in` - Token validity in seconds (usually 1 hour)

### Error Responses

#### 409 Conflict - Email Already Exists

```json
{
  "error": "conflict",
  "message": "User with this email already exists"
}
```

**Fix:** Timestamp-based emails should make this extremely rare. If it happens, retry.

#### 400 Bad Request - Invalid Input

```json
{
  "error": "bad_request",
  "message": "Invalid email format",
  "details": {
    "email": ["Must be a valid email address"]
  }
}
```

**Common causes:**
- Malformed email
- Missing required fields
- Password too short

#### 503 Service Unavailable

```json
{
  "error": "service_unavailable",
  "message": "Service temporarily unavailable"
}
```

**Causes:**
- VPN not connected
- Staging gateway down
- Network timeout

**Fix:** Check VPN connection first.

## Implementation Example

### Using curl + jq

```bash
RESPONSE=$(curl -s -X POST "https://gw.staging-k8s.hellofresh.io/api/signup?country=US" \
  -H 'content-type: application/json' \
  -d "{
    \"email\": \"${EMAIL}\",
    \"password\": \"${PASSWORD}\",
    \"firstName\": \"${FIRST_NAME}\",
    \"lastName\": \"${LAST_NAME}\"
  }")

# Extract fields
ACCESS_TOKEN=$(echo "$RESPONSE" | jq -r '.access_token')
CUSTOMER_ID=$(echo "$RESPONSE" | jq -r '.customerId // .customer_id')

# Validate
if [[ -z "$ACCESS_TOKEN" ]] || [[ "$ACCESS_TOKEN" == "null" ]]; then
  echo "❌ Signup failed: $RESPONSE"
  exit 1
fi
```

### Using curl + python3

```bash
RESPONSE=$(curl -s -X POST "https://gw.staging-k8s.hellofresh.io/api/signup?country=US" \
  -H 'content-type: application/json' \
  -d "{
    \"email\": \"${EMAIL}\",
    \"password\": \"${PASSWORD}\",
    \"firstName\": \"${FIRST_NAME}\",
    \"lastName\": \"${LAST_NAME}\"
  }")

# Extract fields
ACCESS_TOKEN=$(echo "$RESPONSE" | python3 -c "import sys, json; d=json.load(sys.stdin); print(d.get('access_token', ''))")
CUSTOMER_ID=$(echo "$RESPONSE" | python3 -c "import sys, json; d=json.load(sys.stdin); print(d.get('customerId') or d.get('customer_id', ''))")

# Validate
if [[ -z "$ACCESS_TOKEN" ]]; then
  echo "❌ Signup failed: $RESPONSE"
  exit 1
fi
```

## Market-Specific Considerations

### Most Markets (US, DE, GB, AU, etc.)
- Standard signup flow
- No additional fields required
- `customerId` returned as string

### Whitelabel Markets (MR, GN, YE, etc.)
- May use `customer_id` instead of `customerId`
- Response format mostly consistent
- Check both field names when parsing

### Special Cases

**Factor (FJ):**
- Same endpoint, different brand context
- May require `brandId` parameter

**Good Chop (MR):**
- Uses standard flow
- Returns whitelabel-specific customer data

## Security Notes

1. **Staging Only:** These credentials are for staging environment only
2. **Test Domain:** Always use `@hellofresh.com` for test emails
3. **Token Expiry:** Access tokens expire in 1 hour, use promptly
4. **No PII:** Never use real user data in test accounts
5. **Cleanup:** Consider periodic cleanup of test users (not automated)

## Debugging Tips

### Test API Manually

```bash
curl -v -X POST 'https://gw.staging-k8s.hellofresh.io/api/signup?country=US' \
  -H 'content-type: application/json' \
  -d '{
    "email": "manual-test@hellofresh.com",
    "password": "qwerty123",
    "firstName": "Manual",
    "lastName": "Test"
  }'
```

Look for:
- HTTP status code (should be 200)
- Response body with access_token
- Network errors (VPN issues)

### Common Issues

1. **Empty Response:**
   - Check VPN connection
   - Verify staging gateway URL
   - Test with `curl -v` for verbose output

2. **401 Unauthorized:**
   - Should not happen on signup
   - If it does, check API endpoint URL

3. **Timeout:**
   - VPN not connected
   - Network issues
   - Add `--max-time 30` to curl

## Related Documentation

- `api/subscription.md` - Next step after signup
- `config/markets.md` - Valid market codes
- `output/credential-manager.md` - Saving credentials
