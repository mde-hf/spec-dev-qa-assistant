# Subscription Creation API

Create a subscription for an authenticated user.

## Endpoint

```
POST https://gw.staging-k8s.hellofresh.io/api/customers/me/subscriptions?country={MARKET}
Authorization: Bearer {access_token}
Content-Type: application/json
```

## Authentication

Requires valid access token from signup response:

```bash
-H "Authorization: Bearer ${ACCESS_TOKEN}"
```

## Plan SKU Format

```
{COUNTRY}-CBT25-{meals}-{people}-{recurrency}
```

**Components:**
- `{COUNTRY}` - Market code (US, DE, GB, etc.)
- `CBT25` - Product type identifier
- `{meals}` - Number of meals per week (2, 3, 4, 5)
- `{people}` - Number of people (2, 4, 6)
- `{recurrency}` - Recurring (0 = standard subscription)

**Examples:**
- `US-CBT25-2-2-0` - US, 2 meals, 2 people
- `DE-CBT25-3-4-0` - Germany, 3 meals, 4 people
- `GB-CBT25-4-2-0` - UK, 4 meals, 2 people
- `AU-CBT25-2-4-0` - Australia, 2 meals, 4 people

## Request Body

```json
{
  "sku": "US-CBT25-2-2-0",
  "address": {
    "line1": "123 Test Street",
    "line2": "",
    "city": "New York",
    "state": "NY",
    "zip": "10001",
    "country": "US"
  },
  "paymentMethod": {
    "type": "credit_card",
    "number": "4111111111111111",
    "cvv": "123",
    "expiryMonth": "12",
    "expiryYear": "2030"
  }
}
```

## Test Addresses

Use these test addresses per market (see `config/addresses.md` for full list):

### United States
```json
{
  "line1": "123 Test Street",
  "city": "New York",
  "state": "NY",
  "zip": "10001",
  "country": "US"
}
```

### Germany
```json
{
  "line1": "Teststraße 123",
  "city": "Berlin",
  "zip": "10115",
  "country": "DE"
}
```

### United Kingdom
```json
{
  "line1": "123 Test Street",
  "city": "London",
  "postcode": "SW1A 1AA",
  "country": "GB"
}
```

### Australia
```json
{
  "line1": "123 Test Street",
  "city": "Sydney",
  "state": "NSW",
  "postcode": "2000",
  "country": "AU"
}
```

## Test Payment Methods

### Most Markets (US, DE, GB, AU, etc.)

**Visa Test Card:**
```json
{
  "type": "credit_card",
  "number": "4111111111111111",
  "cvv": "123",
  "expiryMonth": "12",
  "expiryYear": "2030"
}
```

### 3DS Markets (GN, YE, CK)

May require additional authentication steps. Use standard test card but be aware subscription creation might fail without 3DS flow.

## Response Format

### Success (200 OK)

```json
{
  "subscriptionId": "87654321",
  "id": "87654321",
  "planId": "plan_abc123",
  "customer_plan_id": "plan_abc123",
  "status": "active",
  "sku": "US-CBT25-2-2-0",
  "deliveries": [
    {
      "id": "delivery_123",
      "date": "2026-05-10"
    }
  ]
}
```

**Key Fields:**
- `subscriptionId` or `id` - Subscription identifier
- `planId` or `customer_plan_id` - Plan identifier (needed for cancel/pause)
- `status` - Should be "active"

### Error Responses

#### 400 Bad Request - Invalid SKU

```json
{
  "error": "bad_request",
  "message": "Invalid SKU format"
}
```

**Fix:** Check SKU format, ensure market code matches.

#### 400 Bad Request - Invalid Address

```json
{
  "error": "bad_request",
  "message": "Address validation failed",
  "details": {
    "zip": ["Invalid postal code for region"]
  }
}
```

**Fix:** Use test address from `config/addresses.md`.

#### 400 Bad Request - Invalid Payment

```json
{
  "error": "bad_request",
  "message": "Payment method validation failed"
}
```

**Fix:** Check test card number, expiry date.

#### 401 Unauthorized

```json
{
  "error": "unauthorized",
  "message": "Invalid or expired token"
}
```

**Fix:** Token expired (1 hour limit). Create new user or refresh token.

## Implementation Example

### Using curl + jq

```bash
# Generate SKU
SKU="${MARKET}-CBT25-${MEALS}-${PEOPLE}-0"

# Load test address and payment method
ADDRESS=$(get_test_address "$MARKET")
PAYMENT=$(get_test_payment_method "$MARKET")

# Create subscription
SUB_RESPONSE=$(curl -s -X POST "https://gw.staging-k8s.hellofresh.io/api/customers/me/subscriptions?country=${MARKET}" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H 'content-type: application/json' \
  -d "{
    \"sku\": \"${SKU}\",
    \"address\": ${ADDRESS},
    \"paymentMethod\": ${PAYMENT}
  }")

# Extract fields
SUBSCRIPTION_ID=$(echo "$SUB_RESPONSE" | jq -r '.subscriptionId // .id')
PLAN_ID=$(echo "$SUB_RESPONSE" | jq -r '.planId // .customer_plan_id')

# Validate
if [[ -z "$SUBSCRIPTION_ID" ]] || [[ "$SUBSCRIPTION_ID" == "null" ]]; then
  echo "❌ Subscription creation failed: $SUB_RESPONSE"
  exit 1
fi
```

## Helper Functions

### Get Plan SKU

```bash
get_plan_sku() {
  local market=$1
  local meals=$2
  local people=$3
  echo "${market}-CBT25-${meals}-${people}-0"
}

# Usage
SKU=$(get_plan_sku "US" "2" "2")  # US-CBT25-2-2-0
```

### Get Test Address

```bash
get_test_address() {
  local market=$1
  
  case "$market" in
    US)
      cat <<EOF
{
  "line1": "123 Test Street",
  "city": "New York",
  "state": "NY",
  "zip": "10001",
  "country": "US"
}
EOF
      ;;
    DE)
      cat <<EOF
{
  "line1": "Teststraße 123",
  "city": "Berlin",
  "zip": "10115",
  "country": "DE"
}
EOF
      ;;
    # ... more markets
  esac
}
```

### Get Test Payment Method

```bash
get_test_payment_method() {
  local market=$1
  
  # Most markets use same test card
  cat <<EOF
{
  "type": "credit_card",
  "number": "4111111111111111",
  "cvv": "123",
  "expiryMonth": "12",
  "expiryYear": "2030"
}
EOF
}
```

## Market-Specific Variations

### Meal Kit Markets (US, DE, GB, etc.)
- Standard SKU format
- Standard address/payment flow
- Immediate activation

### Whitelabel Markets (MR, GN, YE, etc.)
- May use different SKU patterns
- Check market-specific requirements
- Response field names may vary

### Special Cases

**Good Chop (MR):**
- Uses different product codes
- SKU format: `MR-{product_code}-{meals}-{people}-0`

**Factor (FJ):**
- Ready-to-eat meals
- Different portion sizes
- SKU format may differ

## Validation Checklist

Before calling API:
- ✅ Valid access token (not expired)
- ✅ Correct SKU format for market
- ✅ Valid address for market region
- ✅ Test payment method configured
- ✅ Market code matches country parameter

## Debugging Tips

### Test Subscription Creation Manually

```bash
# Get token first
curl -X POST 'https://gw.staging-k8s.hellofresh.io/api/signup?country=US' \
  -H 'content-type: application/json' \
  -d '{"email":"test@hellofresh.com","password":"qwerty123","firstName":"Test","lastName":"User"}' \
  | jq -r '.access_token'

# Use token to create subscription
TOKEN="<token_from_above>"
curl -v -X POST 'https://gw.staging-k8s.hellofresh.io/api/customers/me/subscriptions?country=US' \
  -H "Authorization: Bearer $TOKEN" \
  -H 'content-type: application/json' \
  -d '{
    "sku": "US-CBT25-2-2-0",
    "address": {"line1":"123 Test St","city":"New York","state":"NY","zip":"10001","country":"US"},
    "paymentMethod": {"type":"credit_card","number":"4111111111111111","cvv":"123","expiryMonth":"12","expiryYear":"2030"}
  }'
```

### Common Issues

1. **Token Expired:**
   - Create fresh user account
   - Token valid for 1 hour only

2. **Invalid SKU:**
   - Check market code matches country parameter
   - Verify meals/people values are valid

3. **Address Validation Failed:**
   - Use exact format from `config/addresses.md`
   - Check required fields for market

## Related Documentation

- `api/signup.md` - User creation (prerequisite)
- `api/state-management.md` - Cancel/pause subscription
- `config/markets.md` - Market codes and configurations
- `config/addresses.md` - Test addresses per market
- `config/plans.md` - SKU patterns and plan options
