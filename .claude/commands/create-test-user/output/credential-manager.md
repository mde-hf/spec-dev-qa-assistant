# Credential Manager

Save user credentials to fixture files for test reuse.

## Purpose

Store generated test user credentials in JSON fixture files that can be:
- Used by E2E tests (Playwright, Cypress)
- Referenced in test scenarios
- Shared across test suites
- Tracked for cleanup

## Fixture File Format

### Default Path

```
app/fixtures/generated/test-users-{market}.json
```

**Examples:**
- `app/fixtures/generated/test-users-us.json`
- `app/fixtures/generated/test-users-de.json`
- `app/fixtures/generated/test-users-mr.json`

### JSON Structure

**Array of User Objects:**

```json
[
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
  },
  {
    "email": "test-user-1735564956@hellofresh.com",
    "password": "qwerty123",
    "customerId": "12345679",
    "subscriptionId": null,
    "planId": null,
    "market": "US",
    "state": "new",
    "plan": null,
    "loyaltyTier": null,
    "createdAt": "2026-05-04T08:35:00Z",
    "accessToken": "eyJhbGc..."
  }
]
```

## Field Definitions

| Field          | Type   | Description                                    | Required |
|----------------|--------|------------------------------------------------|----------|
| email          | string | User email address                             | ✅       |
| password       | string | User password                                  | ✅       |
| customerId     | string | Customer identifier                            | ✅       |
| subscriptionId | string | Subscription ID (null if state=new)            | ⚠️       |
| planId         | string | Plan ID (null if state=new)                    | ⚠️       |
| market         | string | Market code (US, DE, GB, etc.)                 | ✅       |
| state          | string | User state (new, active, cancelled, paused)    | ✅       |
| plan           | string | Plan config (2-meals-2-people, etc.)           | ⚠️       |
| loyaltyTier    | string | Loyalty tier (basic, gold, platinum, null)     | ❌       |
| createdAt      | string | ISO 8601 timestamp                             | ✅       |
| accessToken    | string | Bearer token (expires in 1 hour)              | ✅       |

**Legend:** ✅ Required | ⚠️ Conditional | ❌ Optional

## Save Logic

### First User (New Fixture)

```bash
mkdir -p "$(dirname "$FIXTURE_PATH")"

cat > "$FIXTURE_PATH" <<EOF
[
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
]
EOF
```

### Append to Existing Fixture

**Using jq:**

```bash
# Build new user object
NEW_USER=$(cat <<EOF
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

# Append to existing array
jq ". + [$NEW_USER]" "$FIXTURE_PATH" > "${FIXTURE_PATH}.tmp"
mv "${FIXTURE_PATH}.tmp" "$FIXTURE_PATH"
```

**Using python3:**

```bash
python3 <<PYEOF
import json
from datetime import datetime

# Read existing fixture
try:
    with open('${FIXTURE_PATH}', 'r') as f:
        users = json.load(f)
except FileNotFoundError:
    users = []

# Build new user
new_user = {
    "email": "${EMAIL}",
    "password": "${PASSWORD}",
    "customerId": "${CUSTOMER_ID}",
    "subscriptionId": "${SUBSCRIPTION_ID}" if "${SUBSCRIPTION_ID}" else None,
    "planId": "${PLAN_ID}" if "${PLAN_ID}" else None,
    "market": "${MARKET}",
    "state": "${STATE}",
    "plan": "${PLAN}" if "${PLAN}" else None,
    "loyaltyTier": "${LOYALTY_TIER}" if "${LOYALTY_TIER}" else None,
    "createdAt": datetime.utcnow().isoformat() + "Z",
    "accessToken": "${ACCESS_TOKEN}"
}

# Append
users.append(new_user)

# Write back
with open('${FIXTURE_PATH}', 'w') as f:
    json.dump(users, f, indent=2)

print(f"✅ Saved to: ${FIXTURE_PATH}")
PYEOF
```

## Usage in Tests

### Playwright

```typescript
// Load fixture
import usersFixture from '../fixtures/generated/test-users-us.json';

test('Login with test user', async ({ page }) => {
  const user = usersFixture[0]; // First user
  
  await page.goto('/login');
  await page.fill('[data-testid="email-input"]', user.email);
  await page.fill('[data-testid="password-input"]', user.password);
  await page.click('[data-testid="login-button"]');
  
  await expect(page).toHaveURL('/my-deliveries');
});

// Filter by state
const activeUser = usersFixture.find(u => u.state === 'active');
const cancelledUser = usersFixture.find(u => u.state === 'cancelled');
```

### Cypress

```javascript
// Load fixture
import usersFixture from '../fixtures/generated/test-users-us.json';

describe('User login', () => {
  it('should login with test user', () => {
    const user = usersFixture[0];
    
    cy.visit('/login');
    cy.get('[data-testid="email-input"]').type(user.email);
    cy.get('[data-testid="password-input"]').type(user.password);
    cy.get('[data-testid="login-button"]').click();
    
    cy.url().should('include', '/my-deliveries');
  });
});
```

### Jest/Node.js

```javascript
const usersFixture = require('../fixtures/generated/test-users-us.json');

describe('API Tests', () => {
  test('should fetch user data', async () => {
    const user = usersFixture[0];
    
    const response = await fetch('https://api.staging.hellofresh.io/customers/me', {
      headers: {
        'Authorization': `Bearer ${user.accessToken}`
      }
    });
    
    const data = await response.json();
    expect(data.customerId).toBe(user.customerId);
  });
});
```

## Query Helpers

### Find User by State

```bash
get_user_by_state() {
  local fixture=$1
  local state=$2
  
  if command -v jq >/dev/null 2>&1; then
    jq -r ".[] | select(.state == \"${state}\") | @json" "$fixture" | head -1
  else
    python3 -c "import json; users=json.load(open('$fixture')); print(json.dumps(next((u for u in users if u['state']=='$state'), None)))"
  fi
}

# Usage
ACTIVE_USER=$(get_user_by_state "app/fixtures/generated/test-users-us.json" "active")
```

### Get Latest User

```bash
get_latest_user() {
  local fixture=$1
  
  if command -v jq >/dev/null 2>&1; then
    jq -r '.[-1] | @json' "$fixture"
  else
    python3 -c "import json; users=json.load(open('$fixture')); print(json.dumps(users[-1]))"
  fi
}

# Usage
LATEST=$(get_latest_user "app/fixtures/generated/test-users-us.json")
```

### Count Users by State

```bash
count_users_by_state() {
  local fixture=$1
  local state=$2
  
  if command -v jq >/dev/null 2>&1; then
    jq -r "[.[] | select(.state == \"${state}\")] | length" "$fixture"
  else
    python3 -c "import json; users=json.load(open('$fixture')); print(len([u for u in users if u['state']=='$state']))"
  fi
}

# Usage
ACTIVE_COUNT=$(count_users_by_state "app/fixtures/generated/test-users-us.json" "active")
echo "Active users: $ACTIVE_COUNT"
```

## Display Summary

### Console Output

```bash
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "✅ Test user created successfully!"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "📧 Email:          ${EMAIL}"
echo "🔑 Password:       ${PASSWORD}"
echo "🆔 Customer ID:    ${CUSTOMER_ID}"

if [[ -n "$SUBSCRIPTION_ID" ]] && [[ "$SUBSCRIPTION_ID" != "null" ]]; then
  echo "🎫 Subscription ID: ${SUBSCRIPTION_ID}"
  echo "📦 Plan:           ${MEALS} meals for ${PEOPLE} people"
fi

echo "🌍 Market:         ${MARKET}"
echo "📊 State:          ${STATE}"

if [[ -n "$LOYALTY_TIER" ]] && [[ "$LOYALTY_TIER" != "null" ]]; then
  echo "🎁 Loyalty:        ${LOYALTY_TIER} tier"
fi

echo ""
echo "📝 Credentials saved to:"
echo "   ${FIXTURE_PATH}"
echo ""

# Show fixture summary
TOTAL_USERS=$(jq 'length' "$FIXTURE_PATH")
echo "📊 Fixture summary:"
echo "   Total users: ${TOTAL_USERS}"
echo "   Latest: ${EMAIL}"
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

## Cleanup Recommendations

### Add to .gitignore

```gitignore
# Generated test fixtures (contains credentials)
app/fixtures/generated/
fixtures/generated/
*.test-users.json
```

### Periodic Cleanup Script

```bash
# cleanup-old-test-users.sh

#!/bin/bash

FIXTURE_DIR="app/fixtures/generated"
DAYS_OLD=7

echo "🧹 Cleaning up test users older than ${DAYS_OLD} days..."

for fixture in "$FIXTURE_DIR"/*.json; do
  if [[ -f "$fixture" ]]; then
    # Remove users older than DAYS_OLD
    jq --arg cutoff_date "$(date -u -d "${DAYS_OLD} days ago" +%Y-%m-%dT%H:%M:%SZ)" \
      '[.[] | select(.createdAt >= $cutoff_date)]' \
      "$fixture" > "${fixture}.tmp"
    
    mv "${fixture}.tmp" "$fixture"
    
    REMAINING=$(jq 'length' "$fixture")
    echo "   $fixture: $REMAINING users remaining"
  fi
done

echo "✅ Cleanup complete"
```

## Security Notes

1. **Never commit fixtures to git** - Add to `.gitignore`
2. **Access tokens expire in 1 hour** - Don't rely on them long-term
3. **Staging only** - These are test credentials for staging environment
4. **No PII** - Test emails only, no real user data
5. **Rotate periodically** - Clean up old users regularly

## Troubleshooting

### Permission Issues

```bash
# Ensure directory is writable
mkdir -p "$(dirname "$FIXTURE_PATH")"
chmod 755 "$(dirname "$FIXTURE_PATH")"
```

### Invalid JSON

```bash
# Validate fixture JSON
if jq empty "$FIXTURE_PATH" 2>/dev/null; then
  echo "✅ Valid JSON"
else
  echo "❌ Invalid JSON in fixture"
  # Backup and recreate
  cp "$FIXTURE_PATH" "${FIXTURE_PATH}.backup"
  echo "[]" > "$FIXTURE_PATH"
fi
```

### Duplicate Users

```bash
# Remove duplicate emails
jq 'unique_by(.email)' "$FIXTURE_PATH" > "${FIXTURE_PATH}.tmp"
mv "${FIXTURE_PATH}.tmp" "$FIXTURE_PATH"
```

## Related Documentation

- `api/signup.md` - User creation API
- `api/subscription.md` - Subscription creation API
- `api/state-management.md` - State manipulation API
- `config/markets.md` - Market configurations
