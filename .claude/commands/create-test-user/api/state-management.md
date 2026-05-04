# Subscription State Management API

Modify subscription state: cancel, pause, or enroll in loyalty programs.

## Cancel Subscription

### Endpoint

```
POST https://gw.staging-k8s.hellofresh.io/api/plans/{planId}/cancel?country={MARKET}
Authorization: Bearer {access_token}
Content-Type: application/json
```

### Request Body

```json
{
  "reason": {
    "text": "Test user creation"
  }
}
```

**Reason Options:**
- "Test user creation"
- "Cost concerns"
- "Too much food"
- "Moving"
- "Other"

### Response

```json
{
  "status": "cancelled",
  "cancelledAt": "2026-05-04T08:30:00Z",
  "lastDeliveryDate": "2026-05-10"
}
```

### Implementation

```bash
CANCEL_RESPONSE=$(curl -s -X POST "https://gw.staging-k8s.hellofresh.io/api/plans/${PLAN_ID}/cancel?country=${MARKET}" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H 'content-type: application/json' \
  -d '{"reason":{"text":"Test user creation"}}')

# Check status
STATUS=$(echo "$CANCEL_RESPONSE" | jq -r '.status')

if [[ "$STATUS" == "cancelled" ]]; then
  echo "✅ Subscription cancelled"
else
  echo "⚠️ Cancellation response: $CANCEL_RESPONSE"
fi
```

---

## Pause Subscription

### Endpoint

```
POST https://gw.staging-k8s.hellofresh.io/api/plans/{planId}/pause?country={MARKET}
Authorization: Bearer {access_token}
Content-Type: application/json
```

### Request Body

```json
{
  "weeks": 4
}
```

**Pause Duration:**
- Minimum: 1 week
- Maximum: Varies by market (usually 12 weeks)
- Common values: 1, 2, 4, 8

### Response

```json
{
  "status": "paused",
  "pausedUntil": "2026-06-01",
  "resumeDate": "2026-06-08"
}
```

### Implementation

```bash
PAUSE_RESPONSE=$(curl -s -X POST "https://gw.staging-k8s.hellofresh.io/api/plans/${PLAN_ID}/pause?country=${MARKET}" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H 'content-type: application/json' \
  -d '{"weeks":4}')

# Check status
STATUS=$(echo "$PAUSE_RESPONSE" | jq -r '.status')

if [[ "$STATUS" == "paused" ]]; then
  RESUME_DATE=$(echo "$PAUSE_RESPONSE" | jq -r '.resumeDate')
  echo "✅ Subscription paused until ${RESUME_DATE}"
else
  echo "⚠️ Pause response: $PAUSE_RESPONSE"
fi
```

---

## Loyalty Enrollment

**Note:** Loyalty API availability varies by market. Not all markets support programmatic loyalty enrollment.

### Supported Markets

- US
- DE
- GB
- AU
- CA
- NL
- BE
- AT

### Endpoint (if available)

```
POST https://gw.staging-k8s.hellofresh.io/api/loyalty/enroll?country={MARKET}
Authorization: Bearer {access_token}
Content-Type: application/json
```

### Request Body

```json
{
  "programId": "loyalty-basic",
  "acceptTerms": true
}
```

### Response

```json
{
  "enrolled": true,
  "tier": "basic",
  "points": 0,
  "enrolledAt": "2026-05-04T08:30:00Z"
}
```

### Implementation

```bash
# Check if loyalty API is available for market
if supports_loyalty_api "$MARKET"; then
  LOYALTY_RESPONSE=$(curl -s -X POST "https://gw.staging-k8s.hellofresh.io/api/loyalty/enroll?country=${MARKET}" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    -H 'content-type: application/json' \
    -d '{"programId":"loyalty-basic","acceptTerms":true}')
  
  ENROLLED=$(echo "$LOYALTY_RESPONSE" | jq -r '.enrolled')
  
  if [[ "$ENROLLED" == "true" ]]; then
    TIER=$(echo "$LOYALTY_RESPONSE" | jq -r '.tier')
    echo "✅ Enrolled in loyalty (${TIER} tier)"
  else
    echo "⚠️ Loyalty enrollment response: $LOYALTY_RESPONSE"
  fi
else
  echo "ℹ️  Loyalty API not available for ${MARKET}"
  echo "   Manual enrollment may be required via UI"
fi
```

### Loyalty Tier Management

**Setting Points/Tier (Advanced):**

Some markets support direct tier assignment for testing:

```bash
# Set loyalty points (testing only)
curl -s -X POST "https://gw.staging-k8s.hellofresh.io/api/loyalty/points?country=${MARKET}" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H 'content-type: application/json' \
  -d '{"points":500,"reason":"test setup"}'
```

**Tier Thresholds (typical):**
- Basic: 0-299 points
- Gold: 300-999 points
- Platinum: 1000+ points

---

## Reactivate Subscription

### Endpoint

```
POST https://gw.staging-k8s.hellofresh.io/reactivation-subscription/reactivate?country={MARKET}
Authorization: Bearer {access_token}
Content-Type: application/json
```

### Request Body

```json
{
  "sku": "US-CBT25-2-2-0"
}
```

### Response

```json
{
  "subscriptionId": "new_sub_id",
  "status": "active",
  "firstDeliveryDate": "2026-05-10"
}
```

### Implementation

```bash
# Reactivate cancelled subscription
REACTIVATE_RESPONSE=$(curl -s -X POST "https://gw.staging-k8s.hellofresh.io/reactivation-subscription/reactivate?country=${MARKET}" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H 'content-type: application/json' \
  -d "{\"sku\":\"${SKU}\"}")

NEW_SUB_ID=$(echo "$REACTIVATE_RESPONSE" | jq -r '.subscriptionId')

if [[ -n "$NEW_SUB_ID" ]] && [[ "$NEW_SUB_ID" != "null" ]]; then
  echo "✅ Subscription reactivated (new ID: ${NEW_SUB_ID})"
else
  echo "❌ Reactivation failed: $REACTIVATE_RESPONSE"
fi
```

---

## Helper Functions

### Check Loyalty API Availability

```bash
supports_loyalty_api() {
  local market=$1
  local supported="US DE GB AU CA NL BE AT"
  echo "$supported" | grep -qw "$market"
}

# Usage
if supports_loyalty_api "US"; then
  echo "Loyalty API available"
fi
```

### Get Subscription Status

```bash
get_subscription_status() {
  local access_token=$1
  local market=$2
  
  curl -s -X GET "https://gw.staging-k8s.hellofresh.io/api/customers/me/subscriptions?country=${market}" \
    -H "Authorization: Bearer ${access_token}" \
    | jq -r '.items[0].status'
}

# Usage
STATUS=$(get_subscription_status "$ACCESS_TOKEN" "$MARKET")
echo "Current status: $STATUS"
```

---

## State Transition Rules

### Valid Transitions

```
new (no subscription)
  ↓ create subscription
active
  ↓ cancel
cancelled
  ↓ reactivate
active

active
  ↓ pause
paused
  ↓ auto-resume after duration
active
```

### Invalid Transitions

- Cannot cancel a subscription that doesn't exist
- Cannot pause a cancelled subscription
- Cannot unpause manually (auto-resumes after duration)

---

## Error Handling

### Common Errors

#### 404 Not Found - Invalid Plan ID

```json
{
  "error": "not_found",
  "message": "Plan not found"
}
```

**Fix:** Verify `planId` from subscription creation response.

#### 400 Bad Request - Already Cancelled

```json
{
  "error": "bad_request",
  "message": "Subscription already cancelled"
}
```

**Info:** Subscription is already in desired state.

#### 400 Bad Request - Cannot Pause

```json
{
  "error": "bad_request",
  "message": "Cannot pause cancelled subscription"
}
```

**Fix:** Check subscription status before attempting pause.

#### 401 Unauthorized

```json
{
  "error": "unauthorized",
  "message": "Invalid or expired token"
}
```

**Fix:** Token expired (1 hour limit). Create new user or refresh token.

---

## Market-Specific Considerations

### Most Markets
- Standard cancel/pause flow
- Loyalty API may not be available
- Immediate state changes

### Whitelabel Markets (MR, GN, YE)
- May use different endpoints
- Pause duration limits may vary
- Check market-specific documentation

### Special Cases

**Good Chop (MR):**
- Different cancellation flow
- May require additional reason codes

**Factor (FJ):**
- Pause may not be available
- Different reactivation process

---

## Testing Scenarios

### Scenario 1: Create Active User, Then Cancel

```bash
# Create user and subscription
/create-test-user US active

# Later, cancel
ACCESS_TOKEN="<from_fixture>"
PLAN_ID="<from_fixture>"
curl -X POST "https://gw.staging-k8s.hellofresh.io/api/plans/${PLAN_ID}/cancel?country=US" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -d '{"reason":{"text":"Test complete"}}'
```

### Scenario 2: Create Paused User

```bash
# Create active, then pause immediately
/create-test-user US paused
# Command handles: create → activate → pause
```

### Scenario 3: Loyalty Enrollment After Creation

```bash
# Create without loyalty
/create-test-user US active

# Enroll separately
ACCESS_TOKEN="<from_fixture>"
curl -X POST "https://gw.staging-k8s.hellofresh.io/api/loyalty/enroll?country=US" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -d '{"programId":"loyalty-basic","acceptTerms":true}'
```

---

## Related Documentation

- `api/signup.md` - User creation
- `api/subscription.md` - Subscription creation
- `config/markets.md` - Market-specific features
- `output/credential-manager.md` - Saving state to fixtures
