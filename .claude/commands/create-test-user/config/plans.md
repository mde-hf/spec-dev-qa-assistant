# Plan SKU Patterns

SKU (Stock Keeping Unit) format patterns for different markets and products.

## Standard Meal Kit SKU Format

```
{COUNTRY}-CBT25-{meals}-{people}-{recurrency}
```

**Components:**
- `{COUNTRY}` - Market code (US, DE, GB, etc.)
- `CBT25` - Product type identifier (meal kit)
- `{meals}` - Number of meals per week
- `{people}` - Number of people per meal
- `{recurrency}` - Recurrency flag (0 = standard subscription)

## Common Plan Configurations

### 2 Meals, 2 People

**Markets:** US, DE, GB, AU, CA, NL, BE, AT, CH, FR, IT, ES, SE, DK, NO, IE, NZ

```
US-CBT25-2-2-0
DE-CBT25-2-2-0
GB-CBT25-2-2-0
AU-CBT25-2-2-0
CA-CBT25-2-2-0
...
```

### 3 Meals, 2 People

```
US-CBT25-3-2-0
DE-CBT25-3-2-0
GB-CBT25-3-2-0
```

### 3 Meals, 4 People

```
US-CBT25-3-4-0
DE-CBT25-3-4-0
GB-CBT25-3-4-0
```

### 4 Meals, 2 People

```
US-CBT25-4-2-0
DE-CBT25-4-2-0
GB-CBT25-4-2-0
```

### 4 Meals, 4 People

```
US-CBT25-4-4-0
DE-CBT25-4-4-0
GB-CBT25-4-4-0
```

## Valid Configurations Per Market

### United States (US)

| Meals | People | SKU                |
|-------|--------|--------------------|
| 2     | 2      | US-CBT25-2-2-0     |
| 3     | 2      | US-CBT25-3-2-0     |
| 4     | 2      | US-CBT25-4-2-0     |
| 2     | 4      | US-CBT25-2-4-0     |
| 3     | 4      | US-CBT25-3-4-0     |
| 4     | 4      | US-CBT25-4-4-0     |
| 5     | 2      | US-CBT25-5-2-0     |
| 5     | 4      | US-CBT25-5-4-0     |

### Germany (DE)

| Meals | People | SKU                |
|-------|--------|--------------------|
| 2     | 2      | DE-CBT25-2-2-0     |
| 3     | 2      | DE-CBT25-3-2-0     |
| 4     | 2      | DE-CBT25-4-2-0     |
| 5     | 2      | DE-CBT25-5-2-0     |
| 2     | 4      | DE-CBT25-2-4-0     |
| 3     | 4      | DE-CBT25-3-4-0     |
| 4     | 4      | DE-CBT25-4-4-0     |
| 5     | 4      | DE-CBT25-5-4-0     |

### United Kingdom (GB)

| Meals | People | SKU                |
|-------|--------|--------------------|
| 2     | 2      | GB-CBT25-2-2-0     |
| 3     | 2      | GB-CBT25-3-2-0     |
| 4     | 2      | GB-CBT25-4-2-0     |
| 2     | 4      | GB-CBT25-2-4-0     |
| 3     | 4      | GB-CBT25-3-4-0     |
| 4     | 4      | GB-CBT25-4-4-0     |

### Australia (AU)

| Meals | People | SKU                |
|-------|--------|--------------------|
| 2     | 2      | AU-CBT25-2-2-0     |
| 3     | 2      | AU-CBT25-3-2-0     |
| 4     | 2      | AU-CBT25-4-2-0     |
| 2     | 4      | AU-CBT25-2-4-0     |
| 3     | 4      | AU-CBT25-3-4-0     |
| 4     | 4      | AU-CBT25-4-4-0     |

## Whitelabel SKU Patterns

### Good Chop (MR)

**Format:** `MR-{product}-{meals}-{people}-{recurrency}`

**Product Codes:**
- `BEEF` - Beef boxes
- `PORK` - Pork boxes
- `CHICKEN` - Chicken boxes
- `MIX` - Mixed meat boxes

**Examples:**
```
MR-BEEF-2-2-0    # 2 beef meals, 2 people
MR-MIX-3-4-0     # 3 mixed meals, 4 people
MR-CHICKEN-4-2-0 # 4 chicken meals, 2 people
```

### Green Chef (GN)

**Format:** `GN-CBT25-{meals}-{people}-{recurrency}`

**Examples:**
```
GN-CBT25-2-2-0
GN-CBT25-3-4-0
```

### Factor (FJ)

**Format:** `FJ-RTE-{meals}-{portions}-{recurrency}`

**Notes:**
- RTE = Ready To Eat
- Portions are individual (usually 1)
- More meals available (6, 8, 10, 12, 18)

**Examples:**
```
FJ-RTE-6-1-0     # 6 meals, 1 portion each
FJ-RTE-8-1-0     # 8 meals, 1 portion each
FJ-RTE-10-1-0    # 10 meals, 1 portion each
FJ-RTE-12-1-0    # 12 meals, 1 portion each
FJ-RTE-18-1-0    # 18 meals, 1 portion each
```

### Every Plate (ER)

**Format:** `ER-CBT25-{meals}-{people}-{recurrency}`

**Examples:**
```
ER-CBT25-2-2-0
ER-CBT25-3-4-0
```

### Chefs Plate (CK)

**Format:** `CK-CBT25-{meals}-{people}-{recurrency}`

**Examples:**
```
CK-CBT25-2-2-0
CK-CBT25-3-4-0
```

### Youfoodz (YE)

**Format:** `YE-RTE-{meals}-{portions}-{recurrency}`

**Examples:**
```
YE-RTE-6-1-0
YE-RTE-10-1-0
```

## Helper Functions

### Generate SKU

```bash
get_plan_sku() {
  local market=$1
  local meals=$2
  local people=$3
  
  # Whitelabel-specific logic
  case "$market" in
    MR)
      # Good Chop - default to MIX product
      echo "${market}-MIX-${meals}-${people}-0"
      ;;
    FJ|YE)
      # Ready-to-eat - use portions instead of people
      echo "${market}-RTE-${meals}-1-0"
      ;;
    *)
      # Standard meal kit format
      echo "${market}-CBT25-${meals}-${people}-0"
      ;;
  esac
}

# Usage
SKU=$(get_plan_sku "US" "2" "2")  # US-CBT25-2-2-0
SKU=$(get_plan_sku "MR" "3" "4")  # MR-MIX-3-4-0
SKU=$(get_plan_sku "FJ" "6" "1")  # FJ-RTE-6-1-0
```

### Parse Plan Option

```bash
parse_plan_option() {
  local plan=$1  # Format: "2-meals-2-people" or "3-meals-4-people"
  
  # Extract meals and people
  MEALS=$(echo "$plan" | cut -d'-' -f1)
  PEOPLE=$(echo "$plan" | cut -d'-' -f3)
  
  echo "meals=$MEALS people=$PEOPLE"
}

# Usage
parse_plan_option "3-meals-4-people"  # meals=3 people=4
```

### Validate Plan Configuration

```bash
validate_plan() {
  local market=$1
  local meals=$2
  local people=$3
  
  # Standard validation for most markets
  if [[ ! "$meals" =~ ^[2-5]$ ]]; then
    echo "❌ Invalid meals count: $meals (valid: 2-5)"
    return 1
  fi
  
  if [[ ! "$people" =~ ^[2-6]$ ]]; then
    echo "❌ Invalid people count: $people (valid: 2, 4, 6)"
    return 1
  fi
  
  # Market-specific validation
  case "$market" in
    FJ|YE)
      # Ready-to-eat: meals 6-18, portions always 1
      if [[ ! "$meals" =~ ^(6|8|10|12|18)$ ]]; then
        echo "❌ Invalid meals for $market: $meals (valid: 6, 8, 10, 12, 18)"
        return 1
      fi
      ;;
  esac
  
  return 0
}

# Usage
if validate_plan "US" "3" "4"; then
  echo "Valid plan configuration"
fi
```

## Default Plans by Market

```bash
get_default_plan() {
  local market=$1
  
  case "$market" in
    FJ|YE)
      # Ready-to-eat default: 6 meals
      echo "6-meals-1-person"
      ;;
    *)
      # Standard default: 2 meals, 2 people
      echo "2-meals-2-people"
      ;;
  esac
}

# Usage
DEFAULT=$(get_default_plan "US")  # 2-meals-2-people
DEFAULT=$(get_default_plan "FJ")  # 6-meals-1-person
```

## Plan Naming Conventions

### User-Friendly Format

For display and user input:
```
2-meals-2-people
3-meals-4-people
4-meals-2-people
```

### API Format (SKU)

For API calls:
```
US-CBT25-2-2-0
DE-CBT25-3-4-0
GB-CBT25-4-2-0
```

### Conversion

```bash
# User format → API format
plan_to_sku() {
  local plan=$1      # "2-meals-2-people"
  local market=$2    # "US"
  
  local meals=$(echo "$plan" | cut -d'-' -f1)
  local people=$(echo "$plan" | cut -d'-' -f3)
  
  get_plan_sku "$market" "$meals" "$people"
}

# Usage
SKU=$(plan_to_sku "3-meals-4-people" "US")  # US-CBT25-3-4-0
```

## Error Messages

### Invalid Plan Format

```
❌ Invalid plan format: 3-meals

Valid format: X-meals-Y-people
  Examples:
    - 2-meals-2-people
    - 3-meals-4-people
    - 4-meals-2-people
    - 5-meals-4-people
```

### Invalid Meals Count

```
❌ Invalid meals count: 7

Valid meals: 2, 3, 4, 5
(Factor/Youfoodz: 6, 8, 10, 12, 18)
```

### Invalid People Count

```
❌ Invalid people count: 3

Valid people: 2, 4, 6
(Factor/Youfoodz: always 1 portion)
```

## Related Documentation

- `config/markets.md` - Market codes and configurations
- `config/addresses.md` - Test addresses
- `api/subscription.md` - Using SKUs in subscription creation
