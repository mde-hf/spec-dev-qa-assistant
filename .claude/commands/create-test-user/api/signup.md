# User Account Creation API

Create a new user account via the customer management service register endpoint.

## Endpoint

```
POST https://gw.staging-k8s.hellofresh.io/v1/register
Content-Type: application/json
```

**Note:** This endpoint uses the customer-management-service API (not the legacy `/api/signup` endpoint).

## Request Body

```json
{
  "business_division": {
    "brand": "BRAND_HELLOFRESH",
    "region_code": "US"
  },
  "email": "test-user-1735564800@hellofresh.com",
  "password": "qwerty123",
  "first_name": "Test",
  "last_name": "User"
}
```

**Required Fields:**
- `business_division` - Object containing brand and region_code
- `email` - User email address
- `password` - User password (minimum 6 characters in staging)
- `first_name` - User's first name (can be empty string)
- `last_name` - User's last name (can be empty string)

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

## Business Division Format

Each market requires a business_division object:

```json
{
  "brand": "BRAND_HELLOFRESH",
  "region_code": "US"
}
```

**Brand values:**
- `BRAND_HELLOFRESH` - Most HelloFresh markets (US, DE, GB, AU, CA, etc.)
- `BRAND_GREENCHEF` - Green Chef (GN)
- `BRAND_EVERYPLATE` - Every Plate (ER)
- `BRAND_FACTOR` - Factor (FJ)
- `BRAND_CHEFS_PLATE` - Chefs Plate (CK)
- `BRAND_YOUFOODZ` - Youfoodz (YE)
- `BRAND_GOODCHOP` - Good Chop (MR)

**Region code:** Same as market code (US, DE, GB, etc.)

### Helper Function

```bash
get_business_division() {
  local market=$1
  local brand=""
  
  case "$market" in
    MR) brand="BRAND_GOODCHOP" ;;
    GN) brand="BRAND_GREENCHEF" ;;
    YE) brand="BRAND_YOUFOODZ" ;;
    CK) brand="BRAND_CHEFS_PLATE" ;;
    FJ) brand="BRAND_FACTOR" ;;
    ER) brand="BRAND_EVERYPLATE" ;;
    *) brand="BRAND_HELLOFRESH" ;;
  esac
  
  cat <<EOF
{
  "brand": "${brand}",
  "region_code": "${market}"
}
EOF
}

# Usage
BUSINESS_DIVISION=$(get_business_division "US")
```

## Response Format

### Success (200 OK)

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "refresh_token_here...",
  "expires_in": 3600,
  "token_type": "Bearer",
  "user_data": {
    "id": "12345678",
    "email": "test-user-1735564800@hellofresh.com",
    "blocked": false,
    "country": "US",
    "username": "test-user-1735564800@hellofresh.com",
    "user_id": "12345678"
  }
}
```

**Key Fields:**
- `access_token` - Use for subsequent API calls (Authorization: Bearer {token})
- `user_data.id` or `user_data.user_id` - Customer identifier
- `expires_in` - Token validity in seconds (usually 3600 = 1 hour)

**Note:** Customer ID can be in `user_data.id` or `user_data.user_id` depending on response format.

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
# Get business division for market
BUSINESS_DIVISION=$(get_business_division "$MARKET")

# Call register endpoint
RESPONSE=$(curl -s -X POST "https://gw.staging-k8s.hellofresh.io/v1/register" \
  -H 'content-type: application/json' \
  -d "{
    \"business_division\": ${BUSINESS_DIVISION},
    \"email\": \"${EMAIL}\",
    \"password\": \"${PASSWORD}\",
    \"first_name\": \"${FIRST_NAME}\",
    \"last_name\": \"${LAST_NAME}\"
  }")

# Extract fields
ACCESS_TOKEN=$(echo "$RESPONSE" | jq -r '.access_token')
CUSTOMER_ID=$(echo "$RESPONSE" | jq -r '.user_data.id // .user_data.user_id')

# Validate
if [[ -z "$ACCESS_TOKEN" ]] || [[ "$ACCESS_TOKEN" == "null" ]]; then
  echo "❌ Registration failed: $RESPONSE"
  exit 1
fi
```

### Using curl + python3

```bash
# Get business division for market
get_business_division() {
  local market=$1
  local brand=""
  
  case "$market" in
    MR) brand="BRAND_GOODCHOP" ;;
    GN) brand="BRAND_GREENCHEF" ;;
    YE) brand="BRAND_YOUFOODZ" ;;
    CK) brand="BRAND_CHEFS_PLATE" ;;
    FJ) brand="BRAND_FACTOR" ;;
    ER) brand="BRAND_EVERYPLATE" ;;
    *) brand="BRAND_HELLOFRESH" ;;
  esac
  
  echo "{\"brand\":\"${brand}\",\"region_code\":\"${market}\"}"
}

BUSINESS_DIVISION=$(get_business_division "$MARKET")

# Call register endpoint
RESPONSE=$(curl -s -X POST "https://gw.staging-k8s.hellofresh.io/v1/register" \
  -H 'content-type: application/json' \
  -d "{
    \"business_division\": ${BUSINESS_DIVISION},
    \"email\": \"${EMAIL}\",
    \"password\": \"${PASSWORD}\",
    \"first_name\": \"${FIRST_NAME}\",
    \"last_name\": \"${LAST_NAME}\"
  }")

# Extract fields
ACCESS_TOKEN=$(echo "$RESPONSE" | python3 -c "import sys, json; d=json.load(sys.stdin); print(d.get('access_token', ''))")
CUSTOMER_ID=$(echo "$RESPONSE" | python3 -c "import sys, json; d=json.load(sys.stdin); user=d.get('user_data', {}); print(user.get('id') or user.get('user_id', ''))")

# Validate
if [[ -z "$ACCESS_TOKEN" ]]; then
  echo "❌ Registration failed: $RESPONSE"
  exit 1
fi
```

## Market-Specific Considerations

### Most Markets (US, DE, GB, AU, etc.)
- Brand: `BRAND_HELLOFRESH`
- Standard registration flow
- No additional fields required
- Customer ID in `user_data.id` or `user_data.user_id`

### Whitelabel Markets (MR, GN, YE, etc.)
- Use specific brand identifier (see business_division helper)
- Response format same as HelloFresh
- Region code matches market code

### Special Cases

**Factor (FJ):**
- Brand: `BRAND_FACTOR`
- Same endpoint and flow
- Region code: `FJ`

**Good Chop (MR):**
- Brand: `BRAND_GOODCHOP`
- Region code: `MR`
- Uses standard customer-management-service

## Security Notes

1. **Staging Only:** These credentials are for staging environment only
2. **Test Domain:** Always use `@hellofresh.com` for test emails
3. **Token Expiry:** Access tokens expire in 1 hour, use promptly
4. **No PII:** Never use real user data in test accounts
5. **Cleanup:** Consider periodic cleanup of test users (not automated)

## Debugging Tips

### Test API Manually

```bash
curl -v -X POST 'https://gw.staging-k8s.hellofresh.io/v1/register' \
  -H 'content-type: application/json' \
  -d '{
    "business_division": {
      "brand": "BRAND_HELLOFRESH",
      "region_code": "US"
    },
    "email": "manual-test@hellofresh.com",
    "password": "qwerty123",
    "first_name": "Manual",
    "last_name": "Test"
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
