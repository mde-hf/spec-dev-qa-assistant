---
name: /create-test-user
argument-hint: '<market> <state> [--plan=X-meals-Y-people] [--loyalty-tier=basic|gold|platinum] [--save-to=path]'
description: Create test user in staging environment via backend APIs
mode: single-agent
dependencies:
  - curl (required)
  - jq or python3 (required for JSON parsing)
  - VPN connection to staging (required)
---

# Create Test User

## Purpose
Create test users in staging environment using direct backend API calls. No browser automation required. Perfect for E2E test preparation and manual testing scenarios.

## Prerequisites Check

**CRITICAL: Check VPN Connection**

```bash
# Test staging gateway connectivity
if curl -s -o /dev/null -w "%{http_code}" --max-time 5 "https://gw.staging-k8s.hellofresh.io/health" 2>/dev/null | grep -q "200"; then
  echo "✅ VPN connected - staging gateway reachable"
  VPN_CONNECTED=true
else
  echo "❌ VPN not connected or staging gateway unreachable"
  echo ""
  echo "Please connect to staging VPN before proceeding:"
  echo "  1. Connect to HelloFresh staging VPN"
  echo "  2. Verify connection: curl https://gw.staging-k8s.hellofresh.io/health"
  echo "  3. Try again"
  echo ""
  read -p "Connected to VPN? (y/N) " -n 1 -r
  if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo ""
    echo "Aborting. Please connect VPN and try again."
    exit 1
  fi
  VPN_CONNECTED=true
fi
```

**Check JSON Parser:**

```bash
# Check for jq or python3
if command -v jq >/dev/null 2>&1; then
  JSON_PARSER="jq"
  echo "✅ jq available for JSON parsing"
elif command -v python3 >/dev/null 2>&1; then
  JSON_PARSER="python3"
  echo "✅ python3 available for JSON parsing"
else
  echo "❌ No JSON parser available (need jq or python3)"
  echo "   Install: brew install jq"
  exit 1
fi
```

## Workflow

### Step 1: Parse Input & Validate

**Required Arguments:**
- `<market>`: Market code (US, DE, GB, AU, CA, MR, GN, etc.)
- `<state>`: User state (new, active, cancelled, paused)

**Optional Arguments:**
- `--plan=X-meals-Y-people`: Plan configuration (default: 2-meals-2-people)
- `--loyalty-tier=basic|gold|platinum`: Enroll in loyalty program
- `--save-to=path`: Custom fixture path (default: app/fixtures/generated/test-users-{market}.json)
- `--reuse-if-exists`: Check fixture for existing user with matching criteria

**Validation:**

See `config/markets.md` for valid market codes and configuration.

```bash
# Validate market
if ! validate_market "$MARKET"; then
  echo "❌ Invalid market: $MARKET"
  echo "Supported markets: US, DE, GB, AU, CA, NL, BE, AT, CH, FR, IT, ES, SE, DK, NO, IE, NZ, MR, GN, YE, CK, FJ, ER, AO, CG, CF, KN"
  exit 1
fi

# Validate state
if [[ ! "$STATE" =~ ^(new|active|cancelled|paused)$ ]]; then
  echo "❌ Invalid state: $STATE"
  echo "Valid states: new, active, cancelled, paused"
  exit 1
fi

# Parse plan option
PLAN="${PLAN:-2-meals-2-people}"
MEALS=$(echo "$PLAN" | cut -d'-' -f1)
PEOPLE=$(echo "$PLAN" | cut -d'-' -f3)

# Set fixture path
FIXTURE_PATH="${SAVE_TO:-app/fixtures/generated/test-users-${MARKET,,}.json}"
```

### Step 2: Generate User Credentials

```bash
# Generate unique email with timestamp
TIMESTAMP=$(date +%s)
EMAIL="test-user-${TIMESTAMP}@hellofresh.com"
PASSWORD="qwerty123"
FIRST_NAME="Test"
LAST_NAME="User"

echo ""
echo "🎯 Creating test user:"
echo "   Market: $MARKET"
echo "   State: $STATE"
echo "   Email: $EMAIL"
if [[ "$STATE" != "new" ]]; then
  echo "   Plan: $MEALS meals for $PEOPLE people"
fi
if [[ -n "$LOYALTY_TIER" ]]; then
  echo "   Loyalty: $LOYALTY_TIER tier"
fi
echo ""
```

### Step 3: Create User Account

See `api/signup.md` for API details.

```bash
echo "📝 Step 1/4: Creating user account..."

# Call signup API
SIGNUP_RESPONSE=$(curl -s -X POST "https://gw.staging-k8s.hellofresh.io/api/signup?country=${MARKET}" \
  -H 'content-type: application/json' \
  -d "{
    \"email\": \"${EMAIL}\",
    \"password\": \"${PASSWORD}\",
    \"firstName\": \"${FIRST_NAME}\",
    \"lastName\": \"${LAST_NAME}\"
  }")

# Parse response
if [[ "$JSON_PARSER" == "jq" ]]; then
  ACCESS_TOKEN=$(echo "$SIGNUP_RESPONSE" | jq -r '.access_token')
  CUSTOMER_ID=$(echo "$SIGNUP_RESPONSE" | jq -r '.customerId // .customer_id')
else
  ACCESS_TOKEN=$(echo "$SIGNUP_RESPONSE" | python3 -c "import sys, json; d=json.load(sys.stdin); print(d.get('access_token', ''))")
  CUSTOMER_ID=$(echo "$SIGNUP_RESPONSE" | python3 -c "import sys, json; d=json.load(sys.stdin); print(d.get('customerId') or d.get('customer_id', ''))")
fi

# Validate
if [[ -z "$ACCESS_TOKEN" ]] || [[ "$ACCESS_TOKEN" == "null" ]]; then
  echo "❌ Failed to create user account"
  echo "Response: $SIGNUP_RESPONSE"
  exit 1
fi

echo "✅ User account created (ID: ${CUSTOMER_ID})"
```

### Step 4: Create Subscription (if needed)

See `api/subscription.md` for API details.

Skip if state is "new":

```bash
if [[ "$STATE" == "new" ]]; then
  echo "⏭️  Skipping subscription (state: new)"
  SUBSCRIPTION_ID=""
  PLAN_ID=""
else
  echo "📦 Step 2/4: Creating subscription..."
  
  # Get SKU, address, and payment method from config
  SKU=$(get_plan_sku "$MARKET" "$MEALS" "$PEOPLE")
  ADDRESS=$(get_test_address "$MARKET")
  PAYMENT_METHOD=$(get_test_payment_method "$MARKET")
  
  # Create subscription
  SUB_RESPONSE=$(curl -s -X POST "https://gw.staging-k8s.hellofresh.io/api/customers/me/subscriptions?country=${MARKET}" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    -H 'content-type: application/json' \
    -d "{
      \"sku\": \"${SKU}\",
      \"address\": ${ADDRESS},
      \"paymentMethod\": ${PAYMENT_METHOD}
    }")
  
  # Parse response
  if [[ "$JSON_PARSER" == "jq" ]]; then
    SUBSCRIPTION_ID=$(echo "$SUB_RESPONSE" | jq -r '.subscriptionId // .id')
    PLAN_ID=$(echo "$SUB_RESPONSE" | jq -r '.planId // .customer_plan_id')
  else
    SUBSCRIPTION_ID=$(echo "$SUB_RESPONSE" | python3 -c "import sys, json; d=json.load(sys.stdin); print(d.get('subscriptionId') or d.get('id', ''))")
    PLAN_ID=$(echo "$SUB_RESPONSE" | python3 -c "import sys, json; d=json.load(sys.stdin); print(d.get('planId') or d.get('customer_plan_id', ''))")
  fi
  
  if [[ -z "$SUBSCRIPTION_ID" ]] || [[ "$SUBSCRIPTION_ID" == "null" ]]; then
    echo "❌ Failed to create subscription"
    echo "Response: $SUB_RESPONSE"
    exit 1
  fi
  
  echo "✅ Subscription created (ID: ${SUBSCRIPTION_ID})"
fi
```

### Step 5: Manipulate State

See `api/state-management.md` for API details.

**Cancel subscription:**

```bash
if [[ "$STATE" == "cancelled" ]] && [[ -n "$PLAN_ID" ]]; then
  echo "🚫 Step 3/4: Cancelling subscription..."
  
  CANCEL_RESPONSE=$(curl -s -X POST "https://gw.staging-k8s.hellofresh.io/api/plans/${PLAN_ID}/cancel?country=${MARKET}" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    -H 'content-type: application/json' \
    -d '{"reason":{"text":"Test user creation"}}')
  
  echo "✅ Subscription cancelled"
fi
```

**Pause subscription:**

```bash
if [[ "$STATE" == "paused" ]] && [[ -n "$PLAN_ID" ]]; then
  echo "⏸️  Step 3/4: Pausing subscription..."
  
  PAUSE_RESPONSE=$(curl -s -X POST "https://gw.staging-k8s.hellofresh.io/api/plans/${PLAN_ID}/pause?country=${MARKET}" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    -H 'content-type: application/json' \
    -d '{"weeks":4}')
  
  echo "✅ Subscription paused"
fi
```

**Enroll in loyalty:**

```bash
if [[ -n "$LOYALTY_TIER" ]]; then
  echo "🎁 Enrolling in loyalty program (${LOYALTY_TIER} tier)..."
  
  # API call for loyalty enrollment (if available in market)
  # Note: Not all markets support loyalty API
  
  echo "✅ Loyalty enrollment attempted"
fi
```

### Step 6: Save Credentials to Fixture

See `output/credential-manager.md` for save logic.

```bash
echo "💾 Step 4/4: Saving credentials to fixture..."

# Create fixture directory if needed
mkdir -p "$(dirname "$FIXTURE_PATH")"

# Build user data object
USER_DATA=$(cat <<EOF
{
  "email": "${EMAIL}",
  "password": "${PASSWORD}",
  "customerId": "${CUSTOMER_ID}",
  "subscriptionId": "${SUBSCRIPTION_ID}",
  "planId": "${PLAN_ID}",
  "market": "${MARKET}",
  "state": "${STATE}",
  "plan": "${PLAN}",
  "loyaltyTier": "${LOYALTY_TIER}",
  "createdAt": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "accessToken": "${ACCESS_TOKEN}"
}
EOF
)

# Append to fixture or create new
if [[ -f "$FIXTURE_PATH" ]]; then
  # Read existing, append new user
  if [[ "$JSON_PARSER" == "jq" ]]; then
    EXISTING=$(cat "$FIXTURE_PATH")
    echo "$EXISTING" | jq ". + [$USER_DATA]" > "$FIXTURE_PATH"
  else
    python3 <<PYEOF
import json
with open('$FIXTURE_PATH', 'r') as f:
    existing = json.load(f)
existing.append($USER_DATA)
with open('$FIXTURE_PATH', 'w') as f:
    json.dump(existing, f, indent=2)
PYEOF
  fi
else
  # Create new fixture with single user
  echo "[$USER_DATA]" | jq '.' > "$FIXTURE_PATH"
fi

echo "✅ Saved to: $FIXTURE_PATH"
```

### Step 7: Display Summary

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "✅ Test user created successfully!"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "📧 Email:          ${EMAIL}"
echo "🔑 Password:       ${PASSWORD}"
echo "🆔 Customer ID:    ${CUSTOMER_ID}"
if [[ -n "$SUBSCRIPTION_ID" ]]; then
  echo "🎫 Subscription ID: ${SUBSCRIPTION_ID}"
  echo "📦 Plan:           ${MEALS} meals for ${PEOPLE} people"
fi
echo "🌍 Market:         ${MARKET}"
echo "📊 State:          ${STATE}"
if [[ -n "$LOYALTY_TIER" ]]; then
  echo "🎁 Loyalty:        ${LOYALTY_TIER} tier"
fi
echo ""
echo "📝 Credentials saved to:"
echo "   ${FIXTURE_PATH}"
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

## Helper Functions

Load these from config modules:

```bash
# Source helper functions
source "$(dirname "$0")/config/markets.md"
source "$(dirname "$0")/config/addresses.md"
source "$(dirname "$0")/config/plans.md"

get_plan_sku() {
  local market=$1
  local meals=$2
  local people=$3
  echo "${market}-CBT25-${meals}-${people}-0"
}

validate_market() {
  local market=$1
  local valid_markets="US DE GB AU CA NL BE AT CH FR IT ES SE DK NO IE NZ MR GN YE CK FJ ER AO CG CF KN"
  echo "$valid_markets" | grep -qw "$market"
}
```

## Error Handling

**Common Errors:**

1. **VPN Not Connected:**
   ```
   ❌ VPN not connected or staging gateway unreachable
   → Connect to staging VPN and try again
   ```

2. **Invalid Market:**
   ```
   ❌ Invalid market: XX
   → Use valid market code (see config/markets.md)
   ```

3. **API Error:**
   ```
   ❌ Failed to create user account
   Response: {"error": "..."}
   → Check API response for details
   ```

4. **Subscription Creation Failed:**
   ```
   ❌ Failed to create subscription
   → May need to update test payment method or address
   ```

## Usage Examples

### Example 1: Basic Active User
```bash
/create-test-user US active
```
Creates active US user with default 2-meals-2-people plan.

### Example 2: Cancelled User with Custom Plan
```bash
/create-test-user DE cancelled --plan=3-meals-4-people
```
Creates German user with 3 meals for 4 people, then cancels.

### Example 3: Loyalty-Enrolled User
```bash
/create-test-user US active --loyalty-tier=gold
```
Creates active US user and attempts loyalty enrollment.

### Example 4: New User (No Subscription)
```bash
/create-test-user GB new
```
Creates UK user account without subscription.

### Example 5: Custom Fixture Path
```bash
/create-test-user AU active --save-to=fixtures/my-tests/users.json
```
Saves credentials to custom fixture location.

## Output Format

The fixture JSON contains:

```json
{
  "email": "test-user-1735564800@hellofresh.com",
  "password": "qwerty123",
  "customerId": "12345678",
  "subscriptionId": "87654321",
  "planId": "plan_abc123",
  "market": "US",
  "state": "active",
  "plan": "2-meals-2-people",
  "loyaltyTier": "gold",
  "createdAt": "2026-05-04T08:30:00Z",
  "accessToken": "eyJhbGc..."
}
```

## Important Notes

1. **VPN Required:** Must be connected to staging VPN
2. **Unique Emails:** Uses timestamp to ensure unique emails
3. **Test Cards:** Uses test payment methods (see config/addresses.md)
4. **State Changes:** Cancel/pause operations are destructive
5. **Fixture Management:** Add `fixtures/generated/` to `.gitignore`
6. **Access Tokens:** Tokens expire, use promptly for API testing

## Supported Markets

See `config/markets.md` for complete list:
- **Meal Kits:** US, DE, GB, AU, CA, NL, BE, AT, CH, FR, IT, ES, SE, DK, NO, IE, NZ
- **Whitelabels:** MR (Good Chop), GN (Green Chef), YE (Youfoodz), CK (Chefs Plate), FJ (Factor), ER (Every Plate)

## Related Commands

- `/setup-qa-assistant` - Setup prerequisites and dependencies
- `/generate-e2e-tests` - Generate tests that can use created users
- `/verify-ac` - Verify ACs using test users
