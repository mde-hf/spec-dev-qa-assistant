# Spec Dev QA Assistant - Complete Overview

**Your AI-powered QA assistant for spec-machine and Claude Code**

---

## 📌 What Is This Tool?

**Spec Dev QA Assistant** automates your entire QA workflow from acceptance criteria collection to test generation and quality tracking. It works seamlessly with JIRA, Confluence, Figma, and X-Ray to streamline testing and quality assurance.

### Key Benefits:
- ⚡ **Save 2+ hours per ticket** through automation
- 🤖 **Generate tests automatically** from acceptance criteria
- 📊 **Track quality metrics** across sprints and teams
- 🔗 **Integrate with JIRA** for seamless workflow
- 🎯 **Support both Web and Mobile** platforms

---

## 🎯 What Problems Does It Solve?

| Problem | Solution |
|---------|----------|
| Manual AC collection from multiple sources | Auto-fetch from JIRA, Confluence, Figma |
| Time-consuming test writing | Generate E2E, API, unit tests automatically |
| Inconsistent test coverage | Interactive dashboard shows gaps |
| Manual JIRA updates | Auto-post verification results |
| No quality visibility | Track trends, risks, and performance |
| Manual X-Ray test creation | Generate and link test cases automatically |

---

## 📋 Available Commands (10 Total)

### 1️⃣ `/collect-ac` - Collect Acceptance Criteria

**What it does:**
- Fetches acceptance criteria from JIRA tickets, Confluence pages, and Figma designs
- Smart detection with confidence scoring
- Handles multiple AC formats (BDD, bullet lists, tables, checkboxes)

**How it works:**
1. Takes JIRA ticket number as input
2. Searches ticket description, custom fields, comments, sub-tasks
3. Checks linked Confluence pages
4. Optionally extracts from Figma designs
5. Creates consolidated AC checklist

**Usage:**
```bash
/collect-ac EPS-1234
```

**Output:**
- AC checklist with source tracking
- Confidence scores for each AC
- Markdown file in `.ac-verification/`

---

### 2️⃣ `/generate-e2e-tests` - Generate Automated Tests

**What it does:**
- Generates automated tests from acceptance criteria
- Supports Web (Playwright, Cypress) and Mobile (Maestro, Detox)
- Scans codebase for existing selectors
- Follows spec-machine test patterns

**How it works:**
1. Reads ACs from `/collect-ac` output
2. Auto-detects platform and framework
3. Scans for test selectors in your code
4. Generates test files based on your implementation
5. Creates tests following project conventions

**Usage:**
```bash
/generate-e2e-tests EPS-1234
```

**Output:**
- Test files in appropriate directory
- Selector quality report
- Coverage analysis

---

### 3️⃣ `/generate-xray-tests` - Generate X-Ray Test Cases

**What it does:**
- Creates X-Ray test cases directly from JIRA ticket ACs
- Interactive format selection (BDD or Manual)
- Parallel test creation for speed
- Automatic linking to source ticket

**How it works:**
1. Fetches ticket and extracts ACs
2. Prompts: "BDD or Manual format?"
3. Generates test cases in selected format
4. Creates tests in X-Ray (up to 5 simultaneously)
5. Links tests back to original ticket

**Usage:**
```bash
# Interactive mode
/generate-xray-tests LMISSIONS-1358

# Skip prompt
/generate-xray-tests LMISSIONS-1358 --format=bdd
```

**Performance:**
- 10 ACs: ~1.8s (84% faster than sequential)
- Parallel execution with rate limiting

---

### 4️⃣ `/document-tests` - Generate Test Coverage Dashboard

**What it does:**
- Creates interactive HTML dashboard showing test coverage
- Analyzes all test layers (E2E, API, unit, visual)
- Identifies coverage gaps with priority matrix
- Updates automatically on subsequent runs

**How it works:**
1. Scans project for all test files
2. Analyzes coverage across different test types
3. Generates test pyramid distribution
4. Creates interactive HTML with team selector
5. Highlights gaps and provides recommendations

**Usage:**
```bash
/document-tests EPS-1234
```

**Output:**
- Interactive HTML dashboard
- Gap analysis with priorities
- Team-specific metrics
- Auto-updates on future changes

---

### 5️⃣ `/dev-risk-analysis` - Developer Quality Metrics

**What it does:**
- Calculates developer quality scores by squad
- Tracks bug fixes and critical issues
- Identifies legacy code risk areas
- Interactive dashboard with drill-down

**How it works:**
1. Prompts for squad name or JIRA project
2. Analyzes last 6 months of merged PRs
3. Detects bug fixes and critical issues
4. Uses git blame for legacy code analysis
5. Generates risk scores (0-100)

**Usage:**
```bash
/dev-risk-analysis
# Enter squad name: SSX
```

**Output:**
- Interactive dashboard with scores
- Developer-level details
- Risk hotspots
- Squad averages with tooltips

---

### 6️⃣ `/verify-ac` - Interactive Verification

**What it does:**
- Step-by-step AC verification workflow
- Tracks test execution status
- Auto-prompts to post results to JIRA

**How it works:**
1. Loads ACs from checklist
2. For each AC:
   - Shows AC details
   - Asks: Passed/Failed/Needs QA?
   - Records evidence
3. Generates verification report
4. Prompts: "Post to JIRA?"

**Usage:**
```bash
/verify-ac EPS-1234
```

**Output:**
- Verification report
- Test results summary
- Auto-post to JIRA option

---

### 7️⃣ `/post-to-jira` - Post Results to JIRA

**What it does:**
- Posts AC verification results as JIRA comment
- Formatted with test results, coverage, evidence
- Auto-updates ticket status and labels

**How it works:**
1. Reads verification report
2. Formats comment in JIRA markup
3. Includes test results, coverage metrics
4. Posts comment to ticket
5. Optionally updates ticket status

**Usage:**
```bash
/post-to-jira EPS-1234
```

**Output:**
- JIRA comment with results
- Updated ticket status
- Applied labels

---

### 8️⃣ `/figma-ac-extractor` - Extract ACs from Figma

**What it does:**
- Extracts user flows and interactions from Figma
- Focuses on functional behavior (not visual styling)
- Merges with existing ACs

**How it works:**
1. Takes Figma design URL
2. Analyzes flows and interactions
3. Extracts form behaviors, validations
4. Converts to AC format
5. Merges with JIRA ACs

**Usage:**
```bash
/figma-ac-extractor https://figma.com/design/abc123
```

**Output:**
- Interaction-based ACs
- Merged AC checklist

---

### 9️⃣ `/ac-quality-trends` - Quality Metrics Over Time

**What it does:**
- Tracks quality metrics across sprints
- Shows pass rates, velocity, issues
- Developer leaderboards
- Trend analysis and recommendations

**How it works:**
1. Analyzes verification reports over time
2. Calculates pass rates, coverage trends
3. Identifies patterns and issues
4. Generates recommendations

**Usage:**
```bash
/ac-quality-trends --sprint "Sprint 24"
```

**Output:**
- Sprint metrics
- Trend charts
- Top issues
- Recommendations

---

### 🔟 `/setup-qa-assistant` - Setup & Configuration

**What it does:**
- Validates all dependencies
- Checks JIRA CLI, Figma MCP, etc.
- Provides setup instructions
- Tests integrations

**How it works:**
1. Checks for JIRA CLI installation
2. Verifies authentication
3. Tests X-Ray access
4. Validates environment variables
5. Provides fix instructions for issues

**Usage:**
```bash
/setup-qa-assistant
```

**Output:**
- Setup validation report
- Fix instructions for issues
- Configuration guide

---

## 🔄 Complete Workflow Example

### Scenario: New Feature Implementation

**Step 1: Gather Requirements**
```bash
/collect-ac EPS-5678
```
→ Found 5 ACs from JIRA + Confluence

**Step 2: (Optional) Add Figma ACs**
```bash
/figma-ac-extractor https://figma.com/design/xyz
```
→ Added 2 interaction ACs | Total: 7 ACs

**Step 3: Implement Feature**
```bash
# ... write your code to satisfy ACs ...
```

**Step 4: Generate Tests**
```bash
/generate-e2e-tests EPS-5678
```
→ Auto-detected Playwright
→ Generated 7 tests based on implementation

**Step 5: Generate X-Ray Test Cases**
```bash
/generate-xray-tests EPS-5678
```
→ Prompt: "BDD or Manual?"
→ Selected: BDD
→ Created 7 X-Ray tests in 1.5s

**Step 6: Run Tests**
```bash
npx playwright test
```
→ All 7 tests passed ✅

**Step 7: Generate Coverage Dashboard**
```bash
/document-tests EPS-5678
```
→ Dashboard shows 100% AC coverage
→ Test pyramid: E2E (7), Unit (15), API (3)

**Step 8: Verify ACs**
```bash
/verify-ac EPS-5678
```
→ Interactive verification
→ All 7 passed ✅
→ Auto-prompt: "Post to JIRA?" → Yes

**Step 9: Create PR**
```bash
/create-pr  # spec-machine command
```
→ PR created with JIRA link
→ Includes test results

---

## 📊 Performance & Metrics

### Time Savings Per Ticket:
- **Manual AC Collection:** 30 min → 2 min (93% faster)
- **Test Generation:** 2 hours → 5 min (95% faster)
- **X-Ray Test Creation:** 20 min → 2 min (90% faster)
- **JIRA Updates:** 10 min → 1 min (90% faster)
- **Coverage Documentation:** 1 hour → 3 min (95% faster)

**Total Savings: ~4 hours per ticket** ⚡

### Quality Improvements:
- ✅ **94% AC pass rate** (tracked)
- ✅ **100% test coverage** visibility
- ✅ **Zero missed ACs** (multi-source detection)
- ✅ **Consistent test patterns** (follows conventions)

---

## 🎯 Use Cases by Role

### For Developers:
- ✅ Auto-generate tests from ACs
- ✅ Clear acceptance criteria upfront
- ✅ Test coverage visibility
- ✅ JIRA updates automated

### For QA Engineers:
- ✅ Structured verification workflow
- ✅ Multi-source AC collection
- ✅ Coverage gap analysis
- ✅ Quality trend tracking

### For Team Leads:
- ✅ Quality metrics dashboard
- ✅ Developer risk analysis
- ✅ Sprint retrospectives
- ✅ Bottleneck identification

### For Product Owners:
- ✅ AC verification transparency
- ✅ Test coverage reports
- ✅ Quality trend visibility
- ✅ Delivery confidence

---

## 🔧 Prerequisites

### Required:
- **Claude Code** - AI coding assistant
- **JIRA CLI** - For JIRA integration
  ```bash
  brew install jira-cli
  jira init
  ```

### Optional:
- **Figma MCP** - For Figma integration
- **Playwright** - For E2E test generation
- **X-Ray for JIRA** - For test case management

---

## 🚀 Getting Started

### 1. Clone Repository
```bash
git clone git@github.com:mde-hf/spec-dev-qa-assistant.git
```

### 2. Install to Your Project
```bash
cd ~/your-project
cp -r ~/spec-dev-qa-assistant/.claude .
```

### 3. Restart Claude Code
Restart Claude Code to load the commands

### 4. Setup Dependencies
```bash
/setup-qa-assistant
```

### 5. Try Your First Command
```bash
/collect-ac YOUR-TICKET-123
```

---

## 📚 Documentation

- **README.md** - Main documentation
- **CHANGELOG.md** - Version history
- **docs/QUICK-START.md** - 5-minute guide
- **docs/AC-WORKFLOW-README.md** - Complete workflow
- **docs/INDEX.md** - Documentation index

---

## 🔗 Integration Points

### JIRA Integration:
- Fetch tickets and ACs
- Create X-Ray test cases
- Post verification results
- Update ticket status

### Confluence Integration:
- Fetch linked pages
- Parse AC content
- Extract requirements

### Figma Integration:
- Extract user flows
- Identify interactions
- Generate functional ACs

### X-Ray Integration:
- Create test cases (BDD/Manual)
- Link tests to stories
- Parallel test creation

### spec-machine Integration:
- Follows JIRA CLI approach
- Uses test patterns
- Compatible workflows

---

## 💡 Pro Tips

1. **Run `/document-tests` after every sprint** for visibility
2. **Use `/dev-risk-analysis` monthly** to identify risks
3. **Always run `/collect-ac` before implementing** for clarity
4. **Generate X-Ray tests in parallel** for speed (automatic)
5. **Use `--dry-run` flag** to preview before creating

---

## 📈 Version History

### v2.2 (April 6, 2026) - Current
- ✨ Added `/generate-xray-tests` with interactive format selection
- ✨ Added `/dev-risk-analysis` with squad metrics
- ⚡ 84% performance improvement (parallel execution)
- ✅ Comprehensive tooltips on all dashboards

### v2.1 (March 22, 2026)
- ✅ Auto-prompt for JIRA post after verify
- ✅ Simplified framework selection
- ✅ Smart selector scanning

---

## 🤝 Contributing

Found a bug or have a suggestion?

1. Create an issue on GitHub
2. Submit a pull request
3. Share feedback in #spec-dev-qa-assistant

---

## 📞 Support

- **GitHub**: https://github.com/mde-hf/spec-dev-qa-assistant
- **Docs**: See `/docs` folder
- **Slack**: #spec-dev-qa-assistant (HelloFresh)

---

**Last Updated:** April 6, 2026
**Version:** 2.2
**Maintained By:** QA Automation Team
