---
name: /ac-quality-trends
argument-hint: '[--sprint SPRINT-ID] [--team TEAM-NAME] [--export]'
description: Track and analyze AC quality metrics over time
mode: single-agent
---

# AC Quality Trends

## Purpose
Track acceptance criteria quality metrics over time to identify patterns, improve processes, and measure team performance.

## Workflow

### Step 1: Collect Historical Data

Scan `.ac-verification/` directory for all tickets:

```typescript
interface VerificationData {
  ticket: string;
  createdDate: Date;
  verifiedDate?: Date;
  createdBy: string;
  verifiedBy: string;
  totalACs: number;
  acsByStatus: {
    passed: number;
    failed: number;
    needsQA: number;
    notApplicable: number;
    deferred: number;
  };
  testCoverage: {
    e2e: number;
    api: number;
    unit: number;
    component: number;
    visual: number;
  };
  sources: string[]; // jira, confluence, figma, manual
  detectionConfidence: 'high' | 'medium' | 'low';
  timeToVerify: number; // hours
  issues: string[];
  sprint?: string;
  team?: string;
}
```

Parse all verification reports:

```typescript
async function collectData() {
  const tickets = [];
  
  // Find all tickets
  const dirs = await fs.readdir('.ac-verification/');
  
  for (const ticketId of dirs) {
    if (ticketId === 'archive' || ticketId === 'config.yml') continue;
    
    const data = await parseVerificationData(ticketId);
    tickets.push(data);
  }
  
  return tickets;
}
```

### Step 2: Calculate Metrics

**A. Coverage Metrics**
```typescript
function calculateCoverage(tickets) {
  return {
    totalTickets: tickets.length,
    totalACs: sum(tickets.map(t => t.totalACs)),
    
    avgACsPerTicket: average(tickets.map(t => t.totalACs)),
    
    passRate: {
      overall: percentage(
        sum(tickets.map(t => t.acsByStatus.passed)),
        sum(tickets.map(t => t.totalACs))
      ),
      byWeek: calculateWeeklyPassRate(tickets)
    },
    
    testCoverage: {
      e2e: percentage(
        tickets.filter(t => t.testCoverage.e2e > 0).length,
        tickets.length
      ),
      api: percentage(
        tickets.filter(t => t.testCoverage.api > 0).length,
        tickets.length
      ),
      unit: percentage(
        tickets.filter(t => t.testCoverage.unit > 0).length,
        tickets.length
      )
    }
  };
}
```

**B. Quality Metrics**
```typescript
function calculateQuality(tickets) {
  return {
    sourceQuality: {
      structured: tickets.filter(t => 
        t.sources.includes('confluence') || t.sources.includes('figma')
      ).length,
      manualOnly: tickets.filter(t => 
        t.sources.length === 1 && t.sources[0] === 'manual'
      ).length
    },
    
    detectionConfidence: {
      high: tickets.filter(t => t.detectionConfidence === 'high').length,
      medium: tickets.filter(t => t.detectionConfidence === 'medium').length,
      low: tickets.filter(t => t.detectionConfidence === 'low').length
    },
    
    acQuality: {
      avgSourcesPerTicket: average(tickets.map(t => t.sources.length)),
      figmaIntegration: percentage(
        tickets.filter(t => t.sources.includes('figma')).length,
        tickets.length
      ),
      confluenceIntegration: percentage(
        tickets.filter(t => t.sources.includes('confluence')).length,
        tickets.length
      )
    }
  };
}
```

**C. Velocity Metrics**
```typescript
function calculateVelocity(tickets) {
  const verified = tickets.filter(t => t.verifiedDate);
  
  return {
    avgTimeToVerify: {
      hours: average(verified.map(t => t.timeToVerify)),
      byComplexity: {
        low: average(verified.filter(t => t.totalACs <= 3).map(t => t.timeToVerify)),
        medium: average(verified.filter(t => t.totalACs <= 6).map(t => t.timeToVerify)),
        high: average(verified.filter(t => t.totalACs > 6).map(t => t.timeToVerify))
      }
    },
    
    verificationRate: {
      daily: verified.length / daysSince(firstTicket),
      weekly: verified.length / weeksSince(firstTicket)
    },
    
    pendingVerifications: tickets.length - verified.length
  };
}
```

**D. Issue Analytics**
```typescript
function analyzeIssues(tickets) {
  const allIssues = tickets.flatMap(t => t.issues || []);
  
  // Categorize issues
  const categories = {
    'Missing selectors': allIssues.filter(i => i.match(/selector|testid/i)).length,
    'Flaky tests': allIssues.filter(i => i.match(/flaky|intermittent/i)).length,
    'API errors': allIssues.filter(i => i.match(/api|endpoint|500|400/i)).length,
    'Timeout': allIssues.filter(i => i.match(/timeout|slow/i)).length,
    'Visual differences': allIssues.filter(i => i.match(/visual|screenshot|pixel/i)).length
  };
  
  // Find patterns
  const mostCommonIssues = Object.entries(categories)
    .sort((a, b) => b[1] - a[1])
    .slice(0, 5);
  
  return {
    totalIssues: allIssues.length,
    categories,
    mostCommon: mostCommonIssues,
    issuesPerTicket: allIssues.length / tickets.length
  };
}
```

### Step 3: Generate Trend Reports

**A. Sprint Report**
```markdown
# AC Quality Report - Sprint 24

Period: March 8-22, 2026
Team: Platform Engineering

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Overview
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Tickets Completed: 18
Total ACs: 67
Pass Rate: 94% ⬆️ (+5% from last sprint)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 AC Coverage
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Average ACs/Ticket: 3.7

Status Breakdown:
  ✅ Passed: 63 (94%)
  🔍 Needs QA: 3 (4%)
  N/A: 1 (2%)
  ❌ Failed: 0 (0%)

Test Coverage:
  E2E Tests: 100% (18/18 tickets)
  API Tests: 78% (14/18 tickets)
  Unit Tests: 89% (16/18 tickets)
  Visual Tests: 44% (8/18 tickets)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✨ Quality Metrics
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

AC Sources:
  📋 JIRA: 18 tickets (100%)
  📄 Confluence: 12 tickets (67%) ⬆️
  🎨 Figma: 8 tickets (44%) ⬆️
  ✍️ Manual Only: 0 tickets (0%)

Detection Confidence:
  ⭐⭐⭐⭐⭐ High: 15 tickets (83%)
  ⭐⭐⭐ Medium: 3 tickets (17%)
  ⭐ Low: 0 tickets (0%)

Average Sources per Ticket: 2.1 ⬆️

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚡ Velocity
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Average Time to Verify: 2.3 hours ⬇️ (was 3.1h)

By Complexity:
  Low (1-3 ACs): 1.2 hours
  Medium (4-6 ACs): 2.5 hours
  High (7+ ACs): 4.1 hours

Verification Rate: 1.3 tickets/day

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🐛 Issues Analysis
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Total Issues: 12 ⬇️ (was 18 last sprint)
Issues per Ticket: 0.67

Top Issues:
1. Missing selectors: 4 occurrences
2. Flaky tests: 3 occurrences
3. API timeouts: 2 occurrences
4. Visual differences: 2 occurrences
5. Accessibility: 1 occurrence

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🏆 Top Performers
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Most ACs Verified:
1. alice@hf.com - 24 ACs (5 tickets)
2. bob@hf.com - 18 ACs (4 tickets)
3. carol@hf.com - 15 ACs (3 tickets)

Highest Quality:
1. alice@hf.com - 100% pass rate, 3.2 sources/ticket
2. david@hf.com - 95% pass rate, 2.8 sources/ticket
3. bob@hf.com - 94% pass rate, 2.1 sources/ticket

Fastest Verification:
1. carol@hf.com - 1.8h average
2. alice@hf.com - 2.1h average
3. bob@hf.com - 2.5h average

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📈 Trends
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Pass Rate Trend (Last 4 Sprints):
Sprint 21: 84%
Sprint 22: 89% ⬆️ +5%
Sprint 23: 89% →
Sprint 24: 94% ⬆️ +5%

Test Coverage Trend:
E2E: 85% → 92% → 96% → 100% ⬆️
API: 62% → 70% → 75% → 78% ⬆️
Visual: 20% → 28% → 35% → 44% ⬆️

Time to Verify Trend:
Sprint 21: 3.8h
Sprint 22: 3.3h ⬇️
Sprint 23: 3.1h ⬇️
Sprint 24: 2.3h ⬇️ Significant improvement!

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
💡 Recommendations
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. 🎯 Continue Figma integration
   - 44% adoption is good, aim for 70%
   - 8 tickets benefited from visual ACs

2. 🔧 Address missing selectors
   - Run /smart-selector-scan on common components
   - Add data-testid to new components proactively

3. 🧪 Reduce test flakiness
   - 3 tickets hit flaky tests
   - Review and stabilize identified tests

4. 📊 Increase visual testing
   - Only 44% of tickets have visual tests
   - Consider automated visual regression for all UI changes

5. ⚡ Maintain velocity
   - 2.3h average is excellent!
   - Document best practices for team

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Sprint Goals Met
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ 100% E2E test coverage (Goal: >95%)
✅ <3h average verification time (Goal: <3.5h)
✅ >90% pass rate (Goal: >85%)
❌ Visual test coverage 44% (Goal: 60%) - Close!
```

**B. Visual Charts (ASCII)**

```
Pass Rate Trend
100% ┤                                      ●
 90% ┤                         ●────────●
 80% ┤         ●────●
 70% ┤
 60% ┤
 50% ┤
     └───────────────────────────────────────
      S21   S22   S23   S24

ACs by Status (Sprint 24)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Passed          ████████████████████ 63 (94%)
🔍 Needs QA        ██ 3 (4%)
N/A                █ 1 (2%)
❌ Failed          0 (0%)

Test Coverage by Type
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
E2E    ████████████████████████ 100%
Unit   ██████████████████████   89%
API    ███████████████████      78%
Visual ███████████              44%
```

### Step 4: Developer Leaderboard

```markdown
## 🏆 Developer Leaderboard (Sprint 24)

### Overall Score
(Based on: ACs verified, pass rate, sources used, time to verify)

| Rank | Developer | Score | ACs | Pass Rate | Avg Sources | Avg Time |
|------|-----------|-------|-----|-----------|-------------|----------|
| 🥇   | alice@hf  | 98    | 24  | 100%      | 3.2         | 2.1h     |
| 🥈   | david@hf  | 94    | 21  | 95%       | 2.8         | 2.4h     |
| 🥉   | bob@hf    | 91    | 18  | 94%       | 2.1         | 2.5h     |
| 4    | carol@hf  | 88    | 15  | 93%       | 1.8         | 1.8h     |

### Category Winners

**Most Comprehensive** 🏅
alice@hf.com - 3.2 sources per ticket (JIRA + Confluence + Figma)

**Fastest Verifier** ⚡
carol@hf.com - 1.8 hours average

**Highest Quality** 💎
alice@hf.com - 100% pass rate, 0 issues

**Most Improved** 📈
bob@hf.com - +12% pass rate from last sprint
```

### Step 5: Export Options

**A. JSON Export**
```bash
/ac-quality-trends --export json

Creates: .ac-verification/reports/sprint-24-report.json
```

**B. CSV Export**
```bash
/ac-quality-trends --export csv

Creates: .ac-verification/reports/sprint-24-metrics.csv
```

**C. Confluence Export**
```bash
/ac-quality-trends --export confluence

Posts to: Confluence team page
```

**D. Slack Report**
```bash
/ac-quality-trends --export slack

Posts to: #team-platform channel
```

### Step 6: Compare Sprints

```bash
/ac-quality-trends --compare "Sprint 23" "Sprint 24"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Sprint Comparison
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Metric              Sprint 23   Sprint 24   Change
──────────────────────────────────────────────────
Tickets             16          18          +2
Total ACs           58          67          +9
Pass Rate           89%         94%         +5% ⬆️
E2E Coverage        96%         100%        +4% ⬆️
Avg Time to Verify  3.1h        2.3h        -0.8h ⬆️
Issues per Ticket   1.1         0.67        -0.43 ⬆️

Improvement Score: +28 points

Key Improvements:
✅ Pass rate increased significantly (+5%)
✅ Verification time reduced by 26%
✅ 100% E2E coverage achieved
✅ Issues per ticket reduced by 39%

Areas for Focus:
⚠️ Visual test coverage still low (44%)
⚠️ API test coverage growth slowing
```

### Step 7: Predictive Analytics

```markdown
## 🔮 Predictions & Insights

Based on current trends:

### Next Sprint Forecast
- Expected tickets: 19-21
- Predicted pass rate: 95-97%
- Expected time to verify: 2.0-2.5h
- Confidence: HIGH (based on 4 sprint trend)

### Risk Analysis
⚠️ **Medium Risk:** Visual test coverage
  - Current: 44%
  - Growth rate: +9% per sprint
  - To reach 70%: 3 more sprints
  - Recommendation: Accelerate adoption

✅ **Low Risk:** Pass rate
  - Trending positively
  - Team well-calibrated

✅ **Low Risk:** Verification time
  - Consistent improvement
  - Process optimization working

### Opportunity Areas
1. **Figma Integration** - 56% adoption potential
2. **Visual Testing** - 56% tickets without visual tests
3. **API Testing** - 22% tickets without API tests

### Pattern Detection
🔍 **Found:** Tickets with Confluence specs verify 35% faster
🔍 **Found:** Figma-linked tickets have 18% fewer visual bugs
🔍 **Found:** Multi-source ACs have 94% higher pass rate
```

## Output Files

- `.ac-verification/reports/sprint-$ID-report.md` - Full report
- `.ac-verification/reports/sprint-$ID-metrics.json` - Raw data
- `.ac-verification/reports/trends.json` - Historical trends
- `.ac-verification/reports/leaderboard.md` - Developer stats

## Usage

```bash
# Current sprint report
/ac-quality-trends

# Specific sprint
/ac-quality-trends --sprint "Sprint 24"

# Specific team
/ac-quality-trends --team "Platform Engineering"

# Compare sprints
/ac-quality-trends --compare "Sprint 23" "Sprint 24"

# Export to Confluence
/ac-quality-trends --export confluence

# Developer stats only
/ac-quality-trends --developers

# JSON export for dashboards
/ac-quality-trends --export json
```

## Integration

- **Dashboard:** Power BI/Grafana via JSON export
- **Slack:** Auto-post sprint reports
- **Confluence:** Team wiki updates
- **JIRA:** Link metrics to sprint retrospectives
- **Team meetings:** Sprint review data

## Benefits

📊 Data-driven process improvement
🎯 Identify bottlenecks and patterns
🏆 Recognize high performers
📈 Track progress over time
💡 Actionable recommendations
🔮 Predictive insights
