---
name: /smart-selector-scan
argument-hint: '[path]'
description: Scan codebase for selectors and generate smart selector mapping
mode: single-agent
---

# Smart Selector Scanner

## Purpose
Automatically scan your codebase to find all test selectors (data-testid, data-cy, aria-labels, etc.) and create a selector map for accurate test generation.

## Workflow

### Step 1: Scan Strategy

Ask user: "What to scan?"

Options:
1. **Full codebase** - Scan entire src/
2. **Specific directory** - Choose directory
3. **Recent changes** - Only modified files (git diff)
4. **Smart scan** - AI-powered relevant files only

### Step 2: Find All Selectors

Scan for multiple selector patterns:

**A. data-testid (React/Web)**
```typescript
// Find patterns like:
<button data-testid="submit-button">
<div data-testid="user-profile">
data-testid="email-input"
getByTestId('submit-button')
screen.getByTestId('email-input')
```

**B. data-cy (Cypress)**
```typescript
<button data-cy="submit-btn">
cy.get('[data-cy=submit-btn]')
```

**C. ARIA labels**
```typescript
aria-label="Close modal"
getByRole('button', { name: 'Submit' })
getByLabelText('Email address')
```

**D. Test IDs (React Native)**
```typescript
testID="submit-button"
accessibilityLabel="Close"
```

**E. CSS Classes (as fallback)**
```typescript
className="submit-button"
```

**F. Component names**
```typescript
// Scan component exports
export const SubmitButton = () => ...
export function UserProfile() { ...}
```

### Step 3: Build Selector Map

Create comprehensive map:

```typescript
interface SelectorMap {
  components: {
    [componentName: string]: {
      file: string;
      selectors: {
        type: 'data-testid' | 'data-cy' | 'aria-label' | 'testID' | 'class';
        value: string;
        line: number;
        context: string; // surrounding code
      }[];
      props: string[];
      routes?: string[]; // if it's a page component
    };
  };
  
  pages: {
    [route: string]: {
      file: string;
      components: string[];
      selectors: string[];
    };
  };
  
  forms: {
    [formName: string]: {
      file: string;
      fields: {
        name: string;
        type: string;
        selector: string;
        validation: string[];
      }[];
    };
  };
}
```

### Step 4: Intelligent Parsing

**Parse React components:**
```typescript
// Example component analysis
function analyzeComponent(fileContent: string, fileName: string) {
  const ast = parse(fileContent);
  
  return {
    name: extractComponentName(ast),
    selectors: findAllSelectors(ast),
    props: extractProps(ast),
    state: extractStateVariables(ast),
    hooks: extractHooks(ast),
    apiCalls: extractAPIEndpoints(ast),
    routes: extractRoutes(ast)
  };
}
```

**Example output:**
```json
{
  "VerificationForm": {
    "file": "src/components/VerificationForm.tsx",
    "selectors": [
      {
        "type": "data-testid",
        "value": "email-input",
        "line": 45,
        "element": "input",
        "context": "<input data-testid=\"email-input\" type=\"email\" />"
      },
      {
        "type": "data-testid",
        "value": "submit-button",
        "line": 52,
        "element": "button",
        "context": "<button data-testid=\"submit-button\">Send</button>"
      }
    ],
    "props": ["onSuccess", "onError", "defaultEmail"],
    "state": ["email", "loading", "error"],
    "apiCalls": ["/api/auth/send-verification"],
    "routes": null
  }
}
```

### Step 5: Detect Patterns

**A. Naming conventions:**
```typescript
// Detect patterns
const patterns = {
  buttons: /-(btn|button|submit|action)$/,
  inputs: /-(input|field|textbox)$/,
  forms: /-(form|modal|dialog)$/,
  containers: /-(container|wrapper|box)$/,
  lists: /-(list|grid|table|items)$/
};

// Examples detected:
"submit-button" → type: button
"email-input" → type: input
"user-form" → type: form
```

**B. Component hierarchy:**
```typescript
// Build component tree
{
  "SignupPage": {
    "children": [
      "VerificationForm",
      "SuccessNotification"
    ]
  },
  "VerificationForm": {
    "children": [
      "EmailInput",
      "SubmitButton"
    ]
  }
}
```

**C. Route mapping:**
```typescript
// Map routes to components
{
  "/signup": "SignupPage",
  "/verify": "VerificationPage",
  "/success": "SuccessPage"
}
```

### Step 6: Generate Selector Library

Create: `.ac-verification/selectors.json`

```json
{
  "version": "1.0.0",
  "generated": "2026-03-22T10:30:00Z",
  "scannedFiles": 156,
  "selectorsFound": 342,
  
  "selectors": {
    "buttons": {
      "submit-button": {
        "file": "src/components/VerificationForm.tsx",
        "line": 52,
        "type": "data-testid",
        "element": "button",
        "text": "Send Verification",
        "usage": ["VerificationForm", "SignupPage"]
      },
      "verify-button": {
        "file": "src/components/CodeInput.tsx",
        "line": 34,
        "type": "data-testid",
        "element": "button"
      }
    },
    
    "inputs": {
      "email-input": {
        "file": "src/components/VerificationForm.tsx",
        "line": 45,
        "type": "data-testid",
        "element": "input",
        "inputType": "email",
        "validation": ["required", "email"],
        "placeholder": "Enter your email"
      },
      "code-input": {
        "file": "src/components/CodeInput.tsx",
        "line": 28,
        "type": "data-testid",
        "element": "input",
        "inputType": "text",
        "pattern": "[0-9]{6}"
      }
    },
    
    "notifications": {
      "success-message": {
        "file": "src/components/Notification.tsx",
        "line": 18,
        "type": "data-testid",
        "variants": ["success", "error", "warning"]
      }
    },
    
    "pages": {
      "/signup": {
        "component": "SignupPage",
        "file": "src/pages/signup.tsx",
        "selectors": ["email-input", "submit-button", "success-message"]
      },
      "/verify": {
        "component": "VerificationPage",
        "file": "src/pages/verify.tsx",
        "selectors": ["code-input", "verify-button"]
      }
    },
    
    "apis": {
      "/api/auth/send-verification": {
        "method": "POST",
        "usedIn": ["VerificationForm"],
        "body": { "email": "string" },
        "responses": {
          "200": { "success": true },
          "400": { "error": "Invalid email" }
        }
      }
    }
  },
  
  "recommendations": {
    "missingSelectors": [
      {
        "file": "src/components/ProfileForm.tsx",
        "line": 67,
        "element": "button",
        "suggestion": "Add data-testid=\"save-profile-button\""
      }
    ],
    "inconsistentNaming": [
      {
        "issue": "Mixed naming conventions",
        "examples": ["submit-btn", "submit-button", "submitButton"],
        "recommendation": "Standardize to: submit-button"
      }
    ]
  }
}
```

### Step 7: Visual Selector Map

Generate visual HTML report:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Selector Map - EPS-1234</title>
  <style>
    /* Interactive selector explorer */
  </style>
</head>
<body>
  <h1>Smart Selector Map</h1>
  
  <div class="stats">
    <div>📊 342 selectors found</div>
    <div>📁 156 files scanned</div>
    <div>⚠️ 12 missing selectors</div>
  </div>
  
  <div class="search">
    <input type="text" placeholder="Search selectors...">
  </div>
  
  <div class="categories">
    <div class="category">
      <h2>Buttons (45)</h2>
      <ul>
        <li>
          <code>submit-button</code>
          <span class="file">VerificationForm.tsx:52</span>
          <button>Copy</button>
          <button>View</button>
        </li>
      </ul>
    </div>
    
    <div class="category">
      <h2>Inputs (78)</h2>
      <!-- ... -->
    </div>
    
    <div class="category">
      <h2>Pages (12)</h2>
      <!-- ... -->
    </div>
  </div>
  
  <div class="component-tree">
    <h2>Component Hierarchy</h2>
    <ul class="tree">
      <li>SignupPage
        <ul>
          <li>VerificationForm
            <ul>
              <li>EmailInput</li>
              <li>SubmitButton</li>
            </ul>
          </li>
        </ul>
      </li>
    </ul>
  </div>
</body>
</html>
```

### Step 8: Integration with Test Generation

Update `/generate-e2e-tests` to use selector map:

```typescript
// When generating tests, use real selectors
const selectorMap = loadSelectorMap();

// Instead of generic:
await page.click('[data-testid="button"]');

// Use actual selector:
await page.click('[data-testid="submit-button"]');

// With context awareness:
const selector = selectorMap.selectors.buttons['submit-button'];
if (selector) {
  test.comment(`Using selector: ${selector.type}="${selector.value}"`);
  test.comment(`Location: ${selector.file}:${selector.line}`);
}
```

### Step 9: Selector Quality Report

Generate quality report:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Selector Quality Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Coverage: 87% (342/394 elements)

✅ Good Practices:
- 95% use data-testid (recommended)
- Consistent naming: kebab-case
- Clear semantic names

⚠️ Issues Found:
1. Missing Selectors (52 elements)
   - 12 buttons without data-testid
   - 8 forms without test IDs
   - 32 interactive elements untestable
   
2. Inconsistent Naming (8 cases)
   - Mixed: submit-btn vs submit-button
   - Suggest: Standardize to -button suffix
   
3. Duplicate Selectors (3 cases)
   - "submit-button" used in 3 different contexts
   - Risk: Tests may target wrong element
   
4. Fragile Selectors (15 cases)
   - Using CSS classes for tests
   - Recommend: Switch to data-testid

📋 Recommendations:
1. Add selectors to 52 untestable elements
2. Rename 8 inconsistent selectors
3. Make 3 duplicate selectors unique
4. Replace 15 CSS-based selectors

Auto-fix available for 45 issues.
Run: /fix-selectors --auto
```

### Step 10: Auto-fix Selectors

```markdown
### New command: /fix-selectors

Automatically fix selector issues:

1. Add missing data-testid attributes
2. Rename inconsistent selectors
3. Make duplicates unique
4. Convert CSS selectors to data-testid

Example changes:
- <button>Submit</button>
+ <button data-testid="submit-button">Submit</button>

- cy.get('.submit-btn')
+ cy.get('[data-testid="submit-button"]')
```

## Output Files

- `.ac-verification/selectors.json` - Machine-readable selector map
- `.ac-verification/selector-map.html` - Visual explorer
- `.ac-verification/selector-quality-report.md` - Quality report
- `.ac-verification/missing-selectors.txt` - Elements needing selectors

## Usage

```bash
# Scan entire codebase
/smart-selector-scan

# Scan specific directory
/smart-selector-scan src/components

# Scan recent changes only
/smart-selector-scan --recent

# Update existing map
/smart-selector-scan --update
```

## Integration

This selector map is automatically used by:
- `/generate-e2e-tests` - Uses real selectors in generated tests
- `/verify-ac` - Can verify selector coverage per AC
- IDE extensions - Autocomplete for test selectors

## Benefits

✅ Accurate test generation (no generic selectors)
✅ Test maintainability (find selector usage)
✅ Consistency enforcement
✅ Coverage tracking
✅ Refactoring safety (detect broken selectors)
