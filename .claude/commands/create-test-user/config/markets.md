# Market Configuration

Supported markets with their codes, brands, and features.

## Market Codes

### HelloFresh Meal Kits

| Code | Country         | Currency | Brand       | Loyalty API |
|------|-----------------|----------|-------------|-------------|
| US   | United States   | USD      | HelloFresh  | ✅          |
| DE   | Germany         | EUR      | HelloFresh  | ✅          |
| GB   | United Kingdom  | GBP      | HelloFresh  | ✅          |
| AU   | Australia       | AUD      | HelloFresh  | ✅          |
| CA   | Canada          | CAD      | HelloFresh  | ✅          |
| NL   | Netherlands     | EUR      | HelloFresh  | ✅          |
| BE   | Belgium         | EUR      | HelloFresh  | ✅          |
| AT   | Austria         | EUR      | HelloFresh  | ✅          |
| CH   | Switzerland     | CHF      | HelloFresh  | ❌          |
| FR   | France          | EUR      | HelloFresh  | ❌          |
| IT   | Italy           | EUR      | HelloFresh  | ❌          |
| ES   | Spain           | EUR      | HelloFresh  | ❌          |
| SE   | Sweden          | SEK      | HelloFresh  | ❌          |
| DK   | Denmark         | DKK      | HelloFresh  | ❌          |
| NO   | Norway          | NOK      | HelloFresh  | ❌          |
| IE   | Ireland         | EUR      | HelloFresh  | ❌          |
| NZ   | New Zealand     | NZD      | HelloFresh  | ❌          |

### Whitelabel Brands

| Code | Country         | Currency | Brand         | Loyalty API |
|------|-----------------|----------|---------------|-------------|
| MR   | United States   | USD      | Good Chop     | ❌          |
| GN   | United States   | USD      | Green Chef    | ❌          |
| YE   | Australia       | AUD      | Youfoodz      | ❌          |
| CK   | Canada          | CAD      | Chefs Plate   | ❌          |
| FJ   | United States   | USD      | Factor        | ❌          |
| ER   | United States   | USD      | Every Plate   | ❌          |
| AO   | Australia       | AUD      | HelloFresh    | ❌          |
| CG   | Canada          | CAD      | HelloFresh    | ❌          |
| CF   | Canada          | CAD      | Chefs Fresh   | ❌          |
| KN   | United Kingdom  | GBP      | Pet's Table   | ❌          |

## Default Plans Per Market

### Meal Kits (2 meals, 2 people)

- `US-CBT25-2-2-0`
- `DE-CBT25-2-2-0`
- `GB-CBT25-2-2-0`
- `AU-CBT25-2-2-0`
- `CA-CBT25-2-2-0`
- etc.

### Whitelabels

**Good Chop (MR):**
- Format: `MR-{product_code}-{meals}-{people}-0`
- Example: `MR-BEEF-2-2-0`

**Factor (FJ):**
- Ready-to-eat meals
- Format: `FJ-RTE-{meals}-{portions}-0`
- Example: `FJ-RTE-6-1-0` (6 meals, 1 portion each)

## Market-Specific Features

### 3DS Required Markets

Markets that require 3D Secure authentication for payments:
- GN (Green Chef)
- YE (Youfoodz)
- CK (Chefs Plate)

**Note:** API-based subscription creation may fail in these markets without completing 3DS flow.

### Charge Flow Markets

Markets using "charge at reactivation" model:
- GN, YE, CK (3DS markets)

### VMS (Variable Menu Selection) Markets

Markets with variable menu selection:
- FJ (Factor)
- MR (Good Chop)

## Validation Functions

### Validate Market Code

```bash
validate_market() {
  local market=$1
  local valid_markets="US DE GB AU CA NL BE AT CH FR IT ES SE DK NO IE NZ MR GN YE CK FJ ER AO CG CF KN"
  
  if echo "$valid_markets" | grep -qw "$market"; then
    return 0
  else
    return 1
  fi
}

# Usage
if validate_market "US"; then
  echo "Valid market"
else
  echo "Invalid market"
fi
```

### Get Market Currency

```bash
get_market_currency() {
  local market=$1
  
  case "$market" in
    US|MR|GN|FJ|ER) echo "USD" ;;
    DE|NL|BE|AT|FR|IT|ES|IE) echo "EUR" ;;
    GB|KN) echo "GBP" ;;
    AU|YE|AO) echo "AUD" ;;
    CA|CK|CG|CF) echo "CAD" ;;
    CH) echo "CHF" ;;
    SE) echo "SEK" ;;
    DK) echo "DKK" ;;
    NO) echo "NOK" ;;
    NZ) echo "NZD" ;;
    *) echo "UNKNOWN" ;;
  esac
}

# Usage
CURRENCY=$(get_market_currency "US")  # USD
```

### Check Loyalty API Support

```bash
supports_loyalty_api() {
  local market=$1
  local supported="US DE GB AU CA NL BE AT"
  echo "$supported" | grep -qw "$market"
}

# Usage
if supports_loyalty_api "US"; then
  echo "Loyalty API available"
else
  echo "Loyalty API not available (use UI for enrollment)"
fi
```

### Get Brand Name

```bash
get_brand_name() {
  local market=$1
  
  case "$market" in
    MR) echo "Good Chop" ;;
    GN) echo "Green Chef" ;;
    YE) echo "Youfoodz" ;;
    CK) echo "Chefs Plate" ;;
    FJ) echo "Factor" ;;
    ER) echo "Every Plate" ;;
    CF) echo "Chefs Fresh" ;;
    KN) echo "Pet's Table" ;;
    *) echo "HelloFresh" ;;
  esac
}

# Usage
BRAND=$(get_brand_name "MR")  # Good Chop
```

## Market-Specific Considerations

### United States (US)
- Largest market
- Full loyalty program
- Standard meal kit offerings
- Multiple whitelabel brands (MR, GN, FJ, ER)

### Germany (DE)
- Second largest market
- Full loyalty program
- European address format
- VAT included in pricing

### United Kingdom (GB)
- Uses "postcode" instead of "zip"
- Full loyalty program
- Brexit-specific considerations

### Australia (AU, YE, AO)
- Uses states (NSW, VIC, QLD, etc.)
- Full loyalty for HelloFresh (AU)
- Youfoodz whitelabel (YE)

### Canada (CA, CK, CG, CF)
- Bilingual (English/French)
- Multiple brands
- Province-based addressing

### Whitelabels

**Good Chop (MR):**
- Meat-focused subscription
- Different product codes
- Specialized SKU format

**Green Chef (GN):**
- Organic/dietary focus
- 3DS payment required
- Different meal options

**Factor (FJ):**
- Ready-to-eat meals
- No cooking required
- Different portion model

**Every Plate (ER):**
- Budget-friendly option
- Simplified meal kits

## Staging Gateway URL

All markets use the same staging gateway:
```
https://gw.staging-k8s.hellofresh.io
```

**Endpoints:**
- Signup: `POST /api/signup?country={MARKET}`
- Subscriptions: `POST /api/customers/me/subscriptions?country={MARKET}`
- Cancel: `POST /api/plans/{planId}/cancel?country={MARKET}`
- Pause: `POST /api/plans/{planId}/pause?country={MARKET}`
- Loyalty: `POST /api/loyalty/enroll?country={MARKET}`

## Error Messages

### Invalid Market

```
❌ Invalid market: XX

Supported markets:
  Meal Kits: US, DE, GB, AU, CA, NL, BE, AT, CH, FR, IT, ES, SE, DK, NO, IE, NZ
  Whitelabels: MR (Good Chop), GN (Green Chef), YE (Youfoodz), CK (Chefs Plate), 
               FJ (Factor), ER (Every Plate), AO, CG, CF (Chefs Fresh), KN (Pet's Table)

See config/markets.md for full details.
```

## Related Documentation

- `config/addresses.md` - Test addresses per market
- `config/plans.md` - SKU patterns and plan options
- `api/signup.md` - User creation API
- `api/subscription.md` - Subscription creation API
