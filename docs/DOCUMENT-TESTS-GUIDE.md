# Test Documentation Generator Guide

## Overview

The `/document-tests` command generates an **interactive HTML dashboard** that visualizes your test coverage, identifies gaps, and automatically updates when tests change.

---

## Quick Start

```bash
/document-tests EPS-1234          # Analyze JIRA ticket
/document-tests shopping-cart     # Analyze feature
/document-tests CartItem.tsx      # Analyze component
/document-tests src/features/checkout  # Analyze directory
```

Opens an interactive dashboard in your browser showing coverage heatmaps, gap analysis, and test quality scores.

---

## What It Generates

### 1. Interactive HTML Dashboard

```
.test-docs/
├── index.html              # Central hub (all features)
├── shopping-cart.html      # Feature-specific page
└── data/
    └── shopping-cart.json  # Coverage data
```

### 2. Visual Elements

#### Coverage Heatmap
Color-coded grid showing test coverage by component:
- 🟢 Green: >80% coverage
- 🟡 Yellow: 50-80% coverage
- 🔴 Red: <50% coverage
- ⚪ Gray: No tests

#### Test Pyramid Visualization
Shows distribution across test layers:
```
        /\
       /E2E\      ← 2 tests (100%)
      /------\
     /  INT   \   ← 5 tests (80%)
    /----------\
   /   UNIT     \ ← 25 tests (90%)
  /--------------\
```

#### Gap Priority Matrix
Risk vs. Impact assessment:
```
        Low Impact    Medium Impact    High Impact
      ┌──────────────┬────────────────┬─────────────┐
High  │              │  🟡 Loading    │ 🔴 Payment  │
Risk  │              │     States     │   Validation│
      ├──────────────┼────────────────┼─────────────┤
Med   │              │  🟡 Error      │ 🟡 Cart     │
Risk  │              │     Handling   │   Logic     │
      ├──────────────┼────────────────┼─────────────┤
Low   │ 🟢 Tooltips  │  🟢 Animations │             │
Risk  │              │                │             │
      └──────────────┴────────────────┴─────────────┘
```

#### Component Detail Cards
For each component:
- Test status (has tests / no tests)
- Coverage % (lines, branches, functions)
- What's tested (list of scenarios)
- Gaps (what's missing)
- Quality score (0-100)
- Risk level (Critical / High / Medium / Low)

#### Component Diagrams
Mermaid diagrams showing:
- Component structure
- Props/state
- Child components
- Test coverage overlay (green = tested, red = not tested)

#### Screenshots with Hotspots
Visual mapping of tested UI elements (optional, requires Playwright)

---

## Input Modes

### 1. JIRA Ticket
```bash
/document-tests EPS-1234
```

**What it does:**
1. Checks if PR is linked to ticket
2. If PR found: Analyzes PR changed files
3. If no PR: Searches git history for ticket mentions
4. Generates docs for all related files

**Best for:** Post-PR documentation, feature verification

### 2. Feature Name
```bash
/document-tests shopping-cart
```

**What it does:**
1. Searches for folders matching "shopping-cart"
2. Prioritizes folders in `features/` directories
3. Analyzes all source files in matching folders

**Best for:** Feature-level documentation, architecture reviews

### 3. Component Name
```bash
/document-tests CartItem.tsx
```

**What it does:**
1. Finds the component file
2. Analyzes the component and its tests
3. Generates focused documentation

**Best for:** Component-level reviews, refactoring

### 4. Directory Path
```bash
/document-tests src/features/checkout
```

**What it does:**
1. Analyzes all source files in directory
2. Finds corresponding test files
3. Generates directory-level documentation

**Best for:** Module documentation, team handoffs

---

## Test Coverage Analysis

### What Counts as "Tested"?

The command uses the **Test Pyramid** approach:

#### 1. E2E Tests (Top Layer)
- User journeys
- Full workflows
- Cross-component interactions
- **Frameworks:** Playwright, Cypress, Maestro, Detox

#### 2. Integration Tests (Middle Layer)
- Feature-level flows
- Multiple components working together
- API + UI integration
- **Frameworks:** Jest + React Testing Library, Vitest

#### 3. Unit Tests (Base Layer)
- Individual components
- Business logic
- Utility functions
- **Frameworks:** Jest, Vitest, Mocha

### Quality Scoring

Each component gets a quality score (0-100) based on:

| Factor | Weight | Description |
|--------|--------|-------------|
| **Test File Exists** | 30% | Has any tests at all |
| **Coverage %** | 25% | Lines/branches/functions covered |
| **Scenarios Tested** | 20% | Props, interactions, states, errors |
| **Edge Cases** | 15% | Loading, error, empty states |
| **Accessibility** | 10% | ARIA, keyboard navigation |

**Scoring:**
- 90-100: Excellent (green)
- 75-89: Good (light green)
- 60-74: Fair (yellow)
- 40-59: Poor (orange)
- 0-39: Critical (red)

---

## Gap Analysis

### Risk Assessment

Each untested or under-tested component is classified:

#### 🔴 Critical Risk
- Payment processing
- Authentication
- Data persistence
- Security-sensitive operations
- **Action:** Test immediately

#### 🟠 High Risk
- Core features
- Complex business logic
- User-facing workflows
- Error-prone areas
- **Action:** Test soon

#### 🟡 Medium Risk
- UI state management
- Error handling
- Form validation
- **Action:** Test when possible

#### 🟢 Low Risk
- Cosmetic components
- Rarely used features
- Simple presentational components
- **Action:** Test if time permits

### Gap Categories

1. **No Tests** - Component has zero test coverage
2. **Insufficient Coverage** - < 50% code coverage
3. **Missing Scenarios** - Core behaviors not tested
4. **Missing Edge Cases** - Error/loading states not tested
5. **No Accessibility Tests** - ARIA/keyboard not tested

---

## Auto-Update System

### Git Hook (Automatic)

After generating docs, a git hook is installed:

```bash
.git/hooks/post-commit
```

**Triggers on:**
- Any commit that changes `*.spec.*` or `*.test.*` files

**What it does:**
1. Detects which components were updated
2. Re-runs `/document-tests` for those components (silent mode)
3. Updates the dashboard automatically
4. Stages updated `.test-docs/` for next commit

### Manual Update

```bash
/document-tests shopping-cart  # Re-generates docs
```

---

## Dashboard Features

### 1. Interactive Filtering

- Filter by risk level
- Filter by coverage %
- Search by component name
- Group by feature/directory

### 2. Sorting

- Sort by coverage (lowest first)
- Sort by risk (highest first)
- Sort by quality score
- Sort by last updated

### 3. Drill-Down

- Click component → See detailed breakdown
- Click gap → See suggested tests
- Click test file → Open in IDE

### 4. Export Options

- Download as PDF
- Export JSON data
- Share permalink
- Generate Markdown report

---

## Example Output

### Console Output

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 Test Documentation Generator
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔍 Discovering test scope...

✅ Found feature: shopping-cart
   Files to analyze: 15
   Test files found: 12

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Scope Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Name: shopping-cart
Files to analyze: 15

🧪 Analyzing test coverage...

✅ CartItem.tsx          85% | Quality: 75 | Medium Risk
✅ CartList.tsx          92% | Quality: 85 | Low Risk
❌ PriceCalculator.ts     0% | Quality:  0 | CRITICAL RISK
✅ CartActions.tsx       78% | Quality: 70 | Medium Risk
⚠️  CartHeader.tsx       45% | Quality: 40 | High Risk

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Test Documentation Generated!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 Coverage Summary:
   Overall: 72.5%
   Files with tests: 12/15
   Critical gaps: 1

📂 Generated Files:
   Dashboard:  .test-docs/index.html
   Feature:    .test-docs/shopping-cart.html
   Data:       .test-docs/data/shopping-cart.json

🌐 Opening in browser...

🔄 Auto-update enabled via git hook
```

### HTML Dashboard

![Test Coverage Dashboard](https://via.placeholder.com/800x500?text=Interactive+Coverage+Dashboard)

**Features:**
- Real-time coverage stats
- Color-coded heatmap
- Clickable component cards
- Gap priority matrix
- Test pyramid chart
- Search and filter
- Mobile responsive

---

## Integration with Other Commands

### Standalone Command

`/document-tests` is **fully standalone** - no dependencies on other commands.

**When to run:**

#### After `/generate-e2e-tests`
```bash
/generate-e2e-tests EPS-1234
npx playwright test
/document-tests EPS-1234  # ← See what was generated
```

#### After `/verify-ac`
```bash
/verify-ac EPS-1234
/document-tests EPS-1234  # ← Document verified coverage
```

#### Before `/post-to-jira`
```bash
/document-tests EPS-1234
# Review gaps in dashboard
/post-to-jira EPS-1234  # ← Include coverage metrics
```

#### Sprint Retrospective
```bash
/document-tests src/features/  # All features
/ac-quality-trends --sprint "Sprint 24"
```

---

## Best Practices

### 1. Run After Major Changes
```bash
# After implementing new feature
git commit -m "feat: Add shopping cart"
/document-tests shopping-cart
```

### 2. Use for Code Reviews
```bash
# Before submitting PR
/document-tests src/features/checkout
# Review gaps, add missing tests
git commit -m "test: Add missing edge cases"
```

### 3. Track Over Time
```bash
# Generate weekly docs
/document-tests src/
# Compare coverage trends
```

### 4. Share with Team
```bash
# Generate docs
/document-tests src/
# Commit to repo
git add .test-docs/
git commit -m "docs: Update test coverage dashboard"
git push
# Share link: https://your-repo.github.io/.test-docs/
```

---

## Troubleshooting

### "No files found"

**Cause:** Input doesn't match any files

**Fix:**
```bash
# Be more specific
/document-tests src/features/shopping-cart  # Not just "shopping"

# Or use directory path
/document-tests app/components/cart/
```

### "JIRA ticket not found"

**Cause:** JIRA CLI not authenticated or ticket doesn't exist

**Fix:**
```bash
jira init  # Re-authenticate
/document-tests EPS-1234
```

### "Coverage data unavailable"

**Cause:** No coverage tool configured

**Fix:**
```bash
# Add to package.json
"scripts": {
  "test:coverage": "jest --coverage"
}

# Or
npm install -D @vitest/coverage-v8
```

### Dashboard not updating

**Cause:** Git hook not triggered

**Fix:**
```bash
# Check hook exists
ls -la .git/hooks/post-commit

# Re-install hook
/document-tests shopping-cart  # Re-generates hook

# Or manually update
/document-tests shopping-cart
```

---

## Advanced Usage

### Custom Coverage Thresholds

Edit `.test-docs/data/${FEATURE}.json`:

```json
{
  "thresholds": {
    "high": 80,
    "medium": 50,
    "low": 30
  }
}
```

### CI/CD Integration

Add to `.github/workflows/test-docs.yml`:

```yaml
name: Update Test Docs

on:
  push:
    paths:
      - '**/*.spec.*'
      - '**/*.test.*'

jobs:
  update-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Generate test docs
        run: |
          /document-tests src/
          git add .test-docs/
          git commit -m "docs: Auto-update test coverage"
          git push
```

### Deploy to GitHub Pages

```bash
# Enable GitHub Pages for .test-docs/
# Settings → Pages → Source: main branch → /test-docs/

# Access at:
# https://your-username.github.io/your-repo/
```

---

## FAQ

**Q: Does this replace code coverage tools?**

A: No, it complements them. It shows:
- Coverage tool: Line/branch coverage %
- `/document-tests`: What behaviors are tested, gaps, risk assessment

**Q: Can I exclude files?**

A: Yes, create `.test-docs/exclude.txt`:
```
node_modules/
dist/
*.stories.tsx
```

**Q: Does it work with monorepos?**

A: Yes! Run from any package:
```bash
cd packages/web
/document-tests src/
```

**Q: Can I customize the dashboard?**

A: Yes, edit `.test-docs/assets/styles/dashboard.css`

**Q: Does it support TypeScript?**

A: Yes, fully supports TS/TSX/JS/JSX

**Q: What about React Native?**

A: Yes! Detects `testID` attributes and Maestro tests

---

## Tips

1. **Start small** - Document one feature first
2. **Share dashboards** - Commit to repo for team visibility
3. **Use in PRs** - Include coverage report in PR description
4. **Track trends** - Re-run weekly to see improvement
5. **Prioritize gaps** - Focus on Critical/High risk first

---

## Related Commands

- `/generate-e2e-tests` - Generate tests for gaps
- `/verify-ac` - Verify acceptance criteria
- `/ac-quality-trends` - Sprint-level metrics
- `/post-to-jira` - Share results with team

---

## Next Steps

1. **Generate your first dashboard:**
   ```bash
   /document-tests src/features/
   ```

2. **Review gaps in browser**

3. **Add missing tests** using `/generate-e2e-tests`

4. **Re-run** to see updated coverage

5. **Share with team** via GitHub Pages or Confluence

---

**Need help?** Check [docs/INDEX.md](INDEX.md) for more guides.
