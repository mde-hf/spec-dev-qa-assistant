# Test Addresses Per Market

Valid test addresses for each market to use in subscription creation.

## Format Notes

- Use these exact addresses for API testing
- Addresses are validated by backend services
- Format varies by country (zip vs postcode, state names, etc.)
- JSON format ready for API calls

## United States (US)

### New York
```json
{
  "line1": "123 Test Street",
  "line2": "",
  "city": "New York",
  "state": "NY",
  "zip": "10001",
  "country": "US"
}
```

### California
```json
{
  "line1": "456 Test Avenue",
  "city": "Los Angeles",
  "state": "CA",
  "zip": "90001",
  "country": "US"
}
```

### Texas
```json
{
  "line1": "789 Test Boulevard",
  "city": "Austin",
  "state": "TX",
  "zip": "78701",
  "country": "US"
}
```

## Germany (DE)

### Berlin
```json
{
  "line1": "Teststraße 123",
  "line2": "",
  "city": "Berlin",
  "zip": "10115",
  "country": "DE"
}
```

### Munich
```json
{
  "line1": "Testweg 456",
  "city": "München",
  "zip": "80331",
  "country": "DE"
}
```

## United Kingdom (GB)

### London
```json
{
  "line1": "123 Test Street",
  "line2": "",
  "city": "London",
  "postcode": "SW1A 1AA",
  "country": "GB"
}
```

### Manchester
```json
{
  "line1": "456 Test Road",
  "city": "Manchester",
  "postcode": "M1 1AE",
  "country": "GB"
}
```

**Note:** UK uses "postcode" instead of "zip"

## Australia (AU, YE, AO)

### Sydney
```json
{
  "line1": "123 Test Street",
  "city": "Sydney",
  "state": "NSW",
  "postcode": "2000",
  "country": "AU"
}
```

### Melbourne
```json
{
  "line1": "456 Test Avenue",
  "city": "Melbourne",
  "state": "VIC",
  "postcode": "3000",
  "country": "AU"
}
```

**Valid Australian States:**
- NSW (New South Wales)
- VIC (Victoria)
- QLD (Queensland)
- WA (Western Australia)
- SA (South Australia)
- TAS (Tasmania)
- ACT (Australian Capital Territory)
- NT (Northern Territory)

## Canada (CA, CK, CG, CF)

### Toronto
```json
{
  "line1": "123 Test Street",
  "city": "Toronto",
  "province": "ON",
  "postalCode": "M5H 2N2",
  "country": "CA"
}
```

### Vancouver
```json
{
  "line1": "456 Test Avenue",
  "city": "Vancouver",
  "province": "BC",
  "postalCode": "V6B 2W9",
  "country": "CA"
}
```

**Valid Canadian Provinces:**
- ON (Ontario)
- BC (British Columbia)
- QC (Quebec)
- AB (Alberta)
- MB (Manitoba)
- SK (Saskatchewan)
- NS (Nova Scotia)
- NB (New Brunswick)
- NL (Newfoundland and Labrador)
- PE (Prince Edward Island)
- YT (Yukon)
- NT (Northwest Territories)
- NU (Nunavut)

## Netherlands (NL)

### Amsterdam
```json
{
  "line1": "Teststraat 123",
  "city": "Amsterdam",
  "zip": "1012 AB",
  "country": "NL"
}
```

## Belgium (BE)

### Brussels
```json
{
  "line1": "Teststraat 123",
  "city": "Brussels",
  "zip": "1000",
  "country": "BE"
}
```

## Austria (AT)

### Vienna
```json
{
  "line1": "Teststraße 123",
  "city": "Wien",
  "zip": "1010",
  "country": "AT"
}
```

## Switzerland (CH)

### Zurich
```json
{
  "line1": "Teststrasse 123",
  "city": "Zürich",
  "zip": "8001",
  "country": "CH"
}
```

## France (FR)

### Paris
```json
{
  "line1": "123 Rue de Test",
  "city": "Paris",
  "zip": "75001",
  "country": "FR"
}
```

## Italy (IT)

### Rome
```json
{
  "line1": "Via del Test 123",
  "city": "Roma",
  "zip": "00100",
  "country": "IT"
}
```

## Spain (ES)

### Madrid
```json
{
  "line1": "Calle de Test 123",
  "city": "Madrid",
  "zip": "28001",
  "country": "ES"
}
```

## Sweden (SE)

### Stockholm
```json
{
  "line1": "Testgatan 123",
  "city": "Stockholm",
  "zip": "111 20",
  "country": "SE"
}
```

## Denmark (DK)

### Copenhagen
```json
{
  "line1": "Testvej 123",
  "city": "København",
  "zip": "1000",
  "country": "DK"
}
```

## Norway (NO)

### Oslo
```json
{
  "line1": "Testveien 123",
  "city": "Oslo",
  "zip": "0010",
  "country": "NO"
}
```

## Ireland (IE)

### Dublin
```json
{
  "line1": "123 Test Street",
  "city": "Dublin",
  "eircode": "D01 F5P2",
  "country": "IE"
}
```

**Note:** Ireland uses "eircode" for postal codes

## New Zealand (NZ)

### Auckland
```json
{
  "line1": "123 Test Street",
  "city": "Auckland",
  "postcode": "1010",
  "country": "NZ"
}
```

## Helper Functions

### Get Test Address

```bash
get_test_address() {
  local market=$1
  
  case "$market" in
    US)
      cat <<'EOF'
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
      cat <<'EOF'
{
  "line1": "Teststraße 123",
  "city": "Berlin",
  "zip": "10115",
  "country": "DE"
}
EOF
      ;;
    
    GB)
      cat <<'EOF'
{
  "line1": "123 Test Street",
  "city": "London",
  "postcode": "SW1A 1AA",
  "country": "GB"
}
EOF
      ;;
    
    AU|YE|AO)
      cat <<'EOF'
{
  "line1": "123 Test Street",
  "city": "Sydney",
  "state": "NSW",
  "postcode": "2000",
  "country": "AU"
}
EOF
      ;;
    
    CA|CK|CG|CF)
      cat <<'EOF'
{
  "line1": "123 Test Street",
  "city": "Toronto",
  "province": "ON",
  "postalCode": "M5H 2N2",
  "country": "CA"
}
EOF
      ;;
    
    NL)
      cat <<'EOF'
{
  "line1": "Teststraat 123",
  "city": "Amsterdam",
  "zip": "1012 AB",
  "country": "NL"
}
EOF
      ;;
    
    BE)
      cat <<'EOF'
{
  "line1": "Teststraat 123",
  "city": "Brussels",
  "zip": "1000",
  "country": "BE"
}
EOF
      ;;
    
    AT)
      cat <<'EOF'
{
  "line1": "Teststraße 123",
  "city": "Wien",
  "zip": "1010",
  "country": "AT"
}
EOF
      ;;
    
    CH)
      cat <<'EOF'
{
  "line1": "Teststrasse 123",
  "city": "Zürich",
  "zip": "8001",
  "country": "CH"
}
EOF
      ;;
    
    FR)
      cat <<'EOF'
{
  "line1": "123 Rue de Test",
  "city": "Paris",
  "zip": "75001",
  "country": "FR"
}
EOF
      ;;
    
    IT)
      cat <<'EOF'
{
  "line1": "Via del Test 123",
  "city": "Roma",
  "zip": "00100",
  "country": "IT"
}
EOF
      ;;
    
    ES)
      cat <<'EOF'
{
  "line1": "Calle de Test 123",
  "city": "Madrid",
  "zip": "28001",
  "country": "ES"
}
EOF
      ;;
    
    SE)
      cat <<'EOF'
{
  "line1": "Testgatan 123",
  "city": "Stockholm",
  "zip": "111 20",
  "country": "SE"
}
EOF
      ;;
    
    DK)
      cat <<'EOF'
{
  "line1": "Testvej 123",
  "city": "København",
  "zip": "1000",
  "country": "DK"
}
EOF
      ;;
    
    NO)
      cat <<'EOF'
{
  "line1": "Testveien 123",
  "city": "Oslo",
  "zip": "0010",
  "country": "NO"
}
EOF
      ;;
    
    IE)
      cat <<'EOF'
{
  "line1": "123 Test Street",
  "city": "Dublin",
  "eircode": "D01 F5P2",
  "country": "IE"
}
EOF
      ;;
    
    NZ)
      cat <<'EOF'
{
  "line1": "123 Test Street",
  "city": "Auckland",
  "postcode": "1010",
  "country": "NZ"
}
EOF
      ;;
    
    # Whitelabels use same address as parent market
    MR|GN|FJ|ER)
      # US-based whitelabels
      get_test_address "US"
      ;;
    
    *)
      echo "❌ No test address configured for market: $market"
      return 1
      ;;
  esac
}

# Usage
ADDRESS=$(get_test_address "US")
echo "$ADDRESS"
```

## Address Validation Notes

### Common Validation Rules

1. **Required Fields:**
   - line1 (street address)
   - city
   - zip/postcode (format varies by country)
   - country

2. **Optional Fields:**
   - line2 (apartment, suite, etc.)
   - state/province (required for some countries)

3. **Format Variations:**
   - US/CA: "zip" or "postalCode"
   - GB/AU/NZ: "postcode"
   - IE: "eircode"
   - CA: "province" instead of "state"

### Testing Tips

1. **Always use exact format** from this file
2. **Don't modify postal codes** - they're validated by backend
3. **Check field names** - zip vs postcode vs postalCode
4. **Use appropriate state/province codes**

## Related Documentation

- `config/markets.md` - Market codes and configurations
- `config/plans.md` - SKU patterns
- `api/subscription.md` - How addresses are used in API calls
