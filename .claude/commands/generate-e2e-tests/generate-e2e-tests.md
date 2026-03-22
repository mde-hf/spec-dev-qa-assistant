---
name: /generate-e2e-tests
argument-hint: '[JIRA-TICKET] [--framework=playwright|cypress|jest|rtl|vitest]'
description: Generate tests from ACs with multi-framework support
mode: single-agent
---

# Generate Tests from Acceptance Criteria (Multi-Framework)

## Purpose
Generate automated tests from acceptance criteria with support for multiple testing frameworks and test types (E2E, API, unit, integration, visual).

## Workflow

### Step 1: Load Acceptance Criteria

Ask user: "What's your JIRA ticket number? (e.g., EPS-1234)"

Read AC from: `.ac-verification/$TICKET/ac-checklist.md`

### Step 2: Smart Selector Scan (Automatic)

**Automatically run selector scan before test generation:**

Display:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 Scanning for Selectors...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Execute smart selector scan:
```bash
# Internally runs selector scanning logic
→ Scans src/ for data-testid, data-cy, testID, aria-label
→ Builds selector map
→ Detects patterns and naming conventions
→ Identifies missing selectors
```

Show results:
```
✅ Selector Scan Complete

Found: 342 selectors
  • Buttons: 45
  • Inputs: 78
  • Forms: 23
  • Pages: 12

Quality: 87% coverage
Missing: 52 elements need selectors

Selector map saved: .ac-verification/selectors.json
```

If quality is low (<70%), warn:
```
⚠️ Selector coverage is low (52%)
Some tests may use generic selectors.
Consider running: /fix-selectors to improve coverage
```

### Step 3: Scan Repository & Detect Stack

Analyze codebase:

**Frontend Detection:**
```javascript
// Check package.json
const deps = readPackageJson();

// Detect framework
if (deps.react) framework = 'React';
if (deps.next) framework = 'Next.js';
if (deps.vue) framework = 'Vue';
if (deps['@angular/core']) framework = 'Angular';
if (deps['react-native']) framework = 'React Native';

// Detect existing test setup
if (deps.playwright) testFrameworks.push('Playwright');
if (deps.cypress) testFrameworks.push('Cypress');
if (deps.jest) testFrameworks.push('Jest');
if (deps.vitest) testFrameworks.push('Vitest');
if (deps['@testing-library/react']) testFrameworks.push('RTL');
```

**Backend Detection:**
```javascript
// Detect backend
if (deps.express || deps.fastify) backend = 'Node.js API';
if (deps.django) backend = 'Django';
if (deps.flask) backend = 'Flask';
if (files.includes('go.mod')) backend = 'Go';
if (files.includes('pom.xml')) backend = 'Java/Spring';

// Detect API test tools
if (deps.supertest) apiTools.push('Supertest');
if (deps.axios) apiTools.push('Axios');
```

Display findings:
```
📊 Repository Analysis
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Frontend: Next.js 14 (React, TypeScript)
Backend: Node.js/Express API
Database: PostgreSQL

Installed Test Frameworks:
  ✅ Playwright (E2E)
  ✅ Jest (Unit/Integration)
  ✅ React Testing Library (Component)
  ✅ Supertest (API)

Test Location: tests/
Naming: *.spec.ts, *.test.ts
Selectors: data-testid attributes (87% coverage)
```

### Step 4: Choose Test Framework (User Selection)

Ask user: "Which test framework do you want to use?"

**Show detected + recommended options:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Select Test Framework
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Detected in your project:
  1. ✅ Playwright (installed)     - Recommended
  2. ✅ Jest (installed)
  3. ✅ Supertest (installed)

Not installed (available):
  4. Cypress - Alternative E2E
  5. Vitest - Modern unit testing
  6. React Testing Library - Component tests

Choose one or more (comma separated): 
Example: 1,2,3 for Playwright + Jest + Supertest
```

**Smart defaults based on detection:**
- If Playwright installed → Default to Playwright
- If Cypress installed → Default to Cypress  
- If neither → Recommend Playwright (offer to install)

**For each framework choice:**

```
You selected: Playwright + Jest + Supertest

Will generate:
  ✅ E2E tests (Playwright) - tests/e2e/
  ✅ Unit tests (Jest) - tests/unit/
  ✅ API tests (Supertest) - tests/api/

Continue? (Y/n)
```

### Step 5: Analyze Each AC & Suggest Test Types

For each AC, analyze and recommend test types:

```javascript
function analyzeAC(ac) {
  const testTypes = [];
  
  // E2E tests for user-facing behavior
  if (ac.match(/user can|user sees|user receives/i)) {
    testTypes.push('E2E');
  }
  
  // API tests for backend behavior
  if (ac.match(/api|endpoint|returns|sends/i)) {
    testTypes.push('API');
  }
  
  // Unit tests for logic/calculations
  if (ac.match(/calculates|validates|expires|expires after/i)) {
    testTypes.push('Unit');
  }
  
  // Component tests for UI components
  if (ac.match(/button|form|modal|dropdown|shows|displays/i)) {
    testTypes.push('Component');
  }
  
  // Visual tests for appearance
  if (ac.match(/color|size|layout|design|style/i)) {
    testTypes.push('Visual');
  }
  
  // Accessibility tests
  if (ac.match(/accessible|screen reader|keyboard|aria/i)) {
    testTypes.push('Accessibility');
  }
  
  // Performance tests
  if (ac.match(/load time|performance|fast|seconds/i)) {
    testTypes.push('Performance');
  }
  
  return testTypes;
}
```

Display analysis:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 AC Analysis & Test Recommendations
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC #1: User receives verification email
  Recommended: E2E, API
  Frameworks: Playwright, Supertest

AC #2: Code expires after 15 minutes
  Recommended: Unit, E2E
  Frameworks: Jest, Playwright

AC #3: Success notification shown
  Recommended: Component, E2E
  Frameworks: React Testing Library, Playwright

AC #4: Page loads in under 2 seconds
  Recommended: Performance, E2E
  Frameworks: Lighthouse, Playwright
```

### Step 6: Generate Tests Using Selector Map

**Use the selector map from Step 2:**

```typescript
// Load selector map
const selectorMap = JSON.parse(
  fs.readFileSync('.ac-verification/selectors.json', 'utf8')
);

// Find real selectors for components
function getSelector(componentName) {
  // Check if selector exists in map
  const selector = selectorMap.selectors.buttons[componentName] ||
                   selectorMap.selectors.inputs[componentName];
  
  if (selector) {
    return `[${selector.type}="${selector.value}"]`;
  }
  
  // Fallback to generic
  console.warn(`No selector found for ${componentName}, using generic`);
  return `[data-testid="${componentName}"]`;
}
```

### Step 7: Generate Tests for Selected Frameworks

#### A. Playwright (E2E)

```typescript
import { test, expect } from '@playwright/test';

test.describe('EPS-1234: Email Verification', () => {
  
  /**
   * AC #1: User receives verification email within 30 seconds
   * 
   * Given user is on signup page
   * When user submits valid email
   * Then verification email is sent within 30 seconds
   */
  test('AC #1: User receives verification email', async ({ page }) => {
    // Given - user is on signup page
    await page.goto('/signup');
    
    // When - user submits valid email
    // Using REAL selectors from selector map:
    const testEmail = `test-${Date.now()}@example.com`;
    await page.fill('[data-testid="email-input"]', testEmail); // ← From selector scan
    await page.click('[data-testid="submit-button"]'); // ← From selector scan
    
    // Then - verification email sent confirmation shown
    await expect(page.locator('[data-testid="success-message"]'))
      .toHaveText(/email sent/i, { timeout: 30000 });
    
    // Verify email was actually sent (API check)
    const response = await page.request.get(
      `/api/test/emails?to=${testEmail}`
    );
    expect(response.status()).toBe(200);
    const emails = await response.json();
    expect(emails).toHaveLength(1);
    expect(emails[0].subject).toContain('Verify');
  });

  /**
   * AC #2: Code expires after 15 minutes
   */
  test('AC #2: Code expires after 15 minutes', async ({ page }) => {
    // This would be a long-running test - consider time mocking
    // Using time manipulation with Playwright
    await page.goto('/verify');
    
    // Fast-forward time by 16 minutes
    await page.addInitScript(() => {
      const now = Date.now();
      Date.now = () => now + (16 * 60 * 1000);
    });
    
    // Attempt to verify with expired code
    await page.fill('[data-testid="code-input"]', '123456');
    await page.click('[data-testid="verify-button"]');
    
    // Should show expiration error
    await expect(page.locator('[data-testid="error-message"]'))
      .toHaveText(/expired/i);
  });
  
  /**
   * AC #3: Success notification shown
   */
  test('AC #3: Success notification displayed', async ({ page }) => {
    await page.goto('/verify');
    
    // Enter valid code
    await page.fill('[data-testid="code-input"]', '123456');
    await page.click('[data-testid="verify-button"]');
    
    // Success notification appears
    const notification = page.locator('[data-testid="notification"]');
    await expect(notification).toBeVisible();
    await expect(notification).toHaveClass(/success/);
    await expect(notification).toHaveText(/verified successfully/i);
    
    // Notification auto-dismisses after 3s
    await expect(notification).not.toBeVisible({ timeout: 4000 });
  });
});
```

#### B. Jest + Supertest (API Tests)

```typescript
import request from 'supertest';
import app from '../src/app';
import { setupTestDB, teardownTestDB } from './helpers/db';

describe('EPS-1234: Email Verification API', () => {
  beforeAll(async () => {
    await setupTestDB();
  });

  afterAll(async () => {
    await teardownTestDB();
  });

  /**
   * AC #1: User receives verification email
   * API Test: POST /api/auth/send-verification
   */
  test('AC #1 (API): Send verification email', async () => {
    const response = await request(app)
      .post('/api/auth/send-verification')
      .send({
        email: 'test@example.com'
      })
      .expect(200);

    expect(response.body).toMatchObject({
      success: true,
      message: expect.stringContaining('sent'),
      emailId: expect.any(String)
    });

    // Verify email job was queued
    // (assuming you use a queue like Bull)
    const jobs = await emailQueue.getJobs();
    expect(jobs).toHaveLength(1);
    expect(jobs[0].data.to).toBe('test@example.com');
  });

  /**
   * AC #2: Code expires after 15 minutes
   * API Test: POST /api/auth/verify with expired code
   */
  test('AC #2 (API): Expired code returns 401', async () => {
    // Create an expired verification code
    const expiredCode = await createExpiredVerificationCode({
      email: 'test@example.com',
      expiresAt: new Date(Date.now() - 1000) // 1 second ago
    });

    const response = await request(app)
      .post('/api/auth/verify')
      .send({
        email: 'test@example.com',
        code: expiredCode.code
      })
      .expect(401);

    expect(response.body).toMatchObject({
      error: 'Code expired',
      code: 'VERIFICATION_CODE_EXPIRED'
    });
  });
});
```

#### C. Jest + React Testing Library (Component Tests)

```typescript
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { VerificationForm } from './VerificationForm';
import { mockEmailService } from '../mocks/emailService';

describe('EPS-1234: Verification Form Component', () => {
  
  /**
   * AC #1: User receives verification email
   * Component Test: Form submission triggers email
   */
  test('AC #1 (Component): Form triggers email send', async () => {
    const user = userEvent.setup();
    const onSuccess = jest.fn();
    
    render(<VerificationForm onSuccess={onSuccess} />);
    
    // User enters email
    const emailInput = screen.getByLabelText(/email/i);
    await user.type(emailInput, 'test@example.com');
    
    // User submits
    const submitButton = screen.getByRole('button', { name: /send/i });
    await user.click(submitButton);
    
    // Loading state shown
    expect(submitButton).toBeDisabled();
    expect(screen.getByText(/sending/i)).toBeInTheDocument();
    
    // Success message appears
    await waitFor(() => {
      expect(screen.getByText(/email sent/i)).toBeInTheDocument();
    });
    
    // Email service was called
    expect(mockEmailService.sendVerification).toHaveBeenCalledWith({
      email: 'test@example.com'
    });
  });

  /**
   * AC #3: Success notification shown
   */
  test('AC #3 (Component): Success notification displays', async () => {
    render(<VerificationForm />);
    
    // Trigger success (mock API response)
    mockEmailService.sendVerification.mockResolvedValue({ success: true });
    
    const user = userEvent.setup();
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.click(screen.getByRole('button', { name: /send/i }));
    
    // Notification appears
    const notification = await screen.findByRole('alert');
    expect(notification).toHaveClass('notification-success');
    expect(notification).toHaveTextContent(/verified successfully/i);
    
    // Notification auto-dismisses
    await waitFor(() => {
      expect(notification).not.toBeInTheDocument();
    }, { timeout: 4000 });
  });
});
```

#### D. Cypress (Alternative E2E)

```javascript
describe('EPS-1234: Email Verification', () => {
  
  /**
   * AC #1: User receives verification email
   */
  it('AC #1: User receives verification email', () => {
    cy.visit('/signup');
    
    // Intercept email API
    cy.intercept('POST', '/api/auth/send-verification', {
      statusCode: 200,
      body: { success: true }
    }).as('sendEmail');
    
    // User submits email
    cy.get('[data-cy=email-input]').type('test@example.com');
    cy.get('[data-cy=submit-button]').click();
    
    // Wait for API call
    cy.wait('@sendEmail');
    
    // Success message shown
    cy.get('[data-cy=success-message]')
      .should('be.visible')
      .and('contain', 'email sent');
  });
  
  /**
   * AC #2: Code expires after 15 minutes
   */
  it('AC #2: Expired code shows error', () => {
    // Use Cypress clock to fast-forward time
    cy.clock();
    
    cy.visit('/verify');
    cy.get('[data-cy=code-input]').type('123456');
    
    // Fast-forward 16 minutes
    cy.tick(16 * 60 * 1000);
    
    cy.get('[data-cy=verify-button]').click();
    
    // Error shown
    cy.get('[data-cy=error-message]')
      .should('contain', 'expired');
  });
});
```

#### E. Lighthouse (Performance)

```javascript
import lighthouse from 'lighthouse';
import * as chromeLauncher from 'chrome-launcher';

/**
 * AC #4: Page loads in under 2 seconds
 */
test('AC #4 (Performance): Page loads under 2s', async () => {
  const chrome = await chromeLauncher.launch({ chromeFlags: ['--headless'] });
  
  const options = {
    logLevel: 'info',
    output: 'json',
    port: chrome.port
  };
  
  const runnerResult = await lighthouse('http://localhost:3000/signup', options);
  
  // Performance metrics
  const metrics = runnerResult.lhr.audits['metrics'].details.items[0];
  
  // Time to Interactive under 2 seconds
  expect(metrics.interactive).toBeLessThan(2000);
  
  // First Contentful Paint under 1 second
  expect(metrics.firstContentfulPaint).toBeLessThan(1000);
  
  // Performance score above 90
  expect(runnerResult.lhr.categories.performance.score).toBeGreaterThan(0.9);
  
  await chrome.kill();
});
```

#### F. Axe (Accessibility)

```typescript
import { test, expect } from '@playwright/test';
import { injectAxe, checkA11y, getViolations } from 'axe-playwright';

/**
 * AC #5: Accessible to screen readers
 */
test('AC #5 (Accessibility): Page is accessible', async ({ page }) => {
  await page.goto('/signup');
  await injectAxe(page);
  
  // Run accessibility checks
  await checkA11y(page, null, {
    detailedReport: true,
    detailedReportOptions: {
      html: true
    }
  });
  
  // Get violations
  const violations = await getViolations(page);
  
  // Ensure no critical violations
  const criticalViolations = violations.filter(v => 
    v.impact === 'critical' || v.impact === 'serious'
  );
  
  expect(criticalViolations).toHaveLength(0);
  
  // Check specific WCAG criteria
  expect(violations.find(v => v.id === 'color-contrast')).toBeUndefined();
  expect(violations.find(v => v.id === 'label')).toBeUndefined();
});
```

**Add selector source comments:**
```typescript
test('AC #1: Email input validation', async ({ page }) => {
  // Selector: email-input (found in src/components/Form.tsx:45)
  // Quality: ✅ High (data-testid)
  await page.fill('[data-testid="email-input"]', 'test@example.com');
  
  // Selector: submit-button (found in src/components/Form.tsx:52)
  // Quality: ✅ High (data-testid)
  await page.click('[data-testid="submit-button"]');
});
```

### Step 8: Generate Test Configuration Files

#### playwright.config.ts
```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  timeout: 30000,
  expect: { timeout: 5000 },
  
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],
  
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### Step 9: Create Test Documentation

Create: `tests/e2e/EPS-1234-README.md`

```markdown
# Test Suite: EPS-1234 - Email Verification

## Overview
Automated tests covering all acceptance criteria for email verification feature.

## Test Coverage

| AC # | Description | E2E | API | Component | Unit | Visual | A11y | Perf |
|------|-------------|-----|-----|-----------|------|--------|------|------|
| 1    | User receives email | ✅ | ✅ | ✅ | - | - | - | - |
| 2    | Code expires | ✅ | ✅ | - | ✅ | - | - | - |
| 3    | Notification shown | ✅ | - | ✅ | - | ✅ | - | - |
| 4    | Loads under 2s | - | - | - | - | - | - | ✅ |
| 5    | Accessible | ✅ | - | - | - | - | ✅ | - |

**Total Tests:** 12
- E2E (Playwright): 5 tests
- API (Supertest): 3 tests
- Component (RTL): 2 tests
- Unit (Jest): 1 test
- Performance (Lighthouse): 1 test

## Running Tests

### All Tests
\`\`\`bash
npm test
\`\`\`

### E2E Tests Only
\`\`\`bash
npx playwright test tests/e2e/EPS-1234.spec.ts
\`\`\`

### API Tests Only
\`\`\`bash
npm test -- tests/api/EPS-1234.test.ts
\`\`\`

### Component Tests Only
\`\`\`bash
npm test -- tests/components/VerificationForm.test.tsx
\`\`\`

### Performance Tests
\`\`\`bash
npm run test:performance
\`\`\`

### Accessibility Tests
\`\`\`bash
npm run test:a11y
\`\`\`

## Test Reports

- Playwright: `playwright-report/index.html`
- Jest: `coverage/lcov-report/index.html`
- Lighthouse: `lighthouse-report.html`
- Axe: `a11y-report.html`
```

### Step 10: Display Summary

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎭 Tests Generated for EPS-1234
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Selector Scan:
- Scanned: 156 files
- Found: 342 selectors
- Quality: 87% coverage
- Map: .ac-verification/selectors.json

✅ Test Files Created:
- tests/e2e/EPS-1234.spec.ts (5 tests) - Playwright
- tests/api/EPS-1234.test.ts (3 tests) - Supertest
- tests/unit/expiration.test.ts (1 test) - Jest

✅ Configuration:
- playwright.config.ts
- jest.config.js

✅ Documentation:
- tests/e2e/EPS-1234-README.md

📊 Test Summary:
Total: 9 tests across 3 frameworks
- E2E: 5 tests (Playwright) - Using real selectors ✅
- API: 3 tests (Supertest)
- Unit: 1 test (Jest)

🎯 AC Coverage: 100% (5/5 ACs covered)
🎯 Selector Quality: 87% (uses real selectors, not generic)

🚀 Next Steps:
1. Review generated tests (check selector accuracy)
2. Run tests: npx playwright test
3. If selectors missing, run: /fix-selectors
4. Implement feature to pass tests
5. Run: /verify-ac EPS-1234
```

## Output Files

- `tests/e2e/$TICKET.spec.ts` - Playwright E2E tests
- `tests/api/$TICKET.test.ts` - API tests
- `tests/components/*.test.tsx` - Component tests
- `tests/unit/*.test.ts` - Unit tests
- `tests/performance/$TICKET.perf.ts` - Performance tests
- `tests/e2e/$TICKET-README.md` - Documentation
- Test config files as needed
