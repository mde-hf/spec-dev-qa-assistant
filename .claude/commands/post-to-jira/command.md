---
name: /post-to-jira
argument-hint: '[JIRA-TICKET]'
description: Post AC verification results back to JIRA ticket
mode: single-agent
dependencies:
  - jira CLI (ankitpokhrel/jira-cli)
---

# Post AC Verification to JIRA

## Purpose
Automatically post AC verification results back to the JIRA ticket as a comment, including test results, coverage, and verification status.

## Prerequisites

Check for JIRA CLI and authentication:
```bash
# Check if JIRA CLI is installed
if which jira >/dev/null 2>&1; then
  # Check if authenticated
  if jira me >/dev/null 2>&1; then
    echo "✅ JIRA CLI ready"
  else
    echo "❌ JIRA CLI not authenticated"
    echo ""
    echo "Please authenticate:"
    echo "  1. Run: jira init"
    echo "  2. Or run: /setup-qa-assistant"
    echo ""
    echo "You'll need:"
    echo "  • JIRA URL: https://yourcompany.atlassian.net"
    echo "  • API Token: https://id.atlassian.com/manage-profile/security/api-tokens"
    exit 1
  fi
else
  echo "❌ JIRA CLI not found"
  echo ""
  echo "Install and configure:"
  echo "  1. Run: /setup-qa-assistant"
  echo "  2. Or manually:"
  echo "     brew install ankitpokhrel/jira-cli/jira-cli"
  echo "     jira init"
  exit 1
fi
```

## Workflow

### Step 1: Get Ticket ID

Ask user: "What's your JIRA ticket number? (e.g., EPS-1234)"

Store as `$TICKET`

### Step 2: Load Verification Report

Read from: `.ac-verification/$TICKET/verification-report.md`

If file doesn't exist:
- Error: "No verification report found for $TICKET"
- Suggest: "Run /verify-ac $TICKET first"
- Exit

### Step 3: Check for E2E Test Results

Look for test results in multiple locations:
1. `tests/e2e/$TICKET.spec.ts` - Test file
2. `playwright-report/` - HTML report
3. `.ac-verification/$TICKET/test-results.json` - JSON results
4. Parse from most recent test run

Extract:
- Total tests
- Passed tests
- Failed tests
- Skipped tests
- Duration
- Screenshot/video links (if available)

### Step 4: Format JIRA Comment

Create formatted comment in Jira markup:

```
h3. ✅ AC Verification Complete - $TICKET

*Verification Date:* $(date)
*Verified By:* $(git config user.email)
*Branch:* $(git branch --show-current)

----

h4. 📋 Acceptance Criteria Results

||#||Acceptance Criterion||Status||
|1|User receives verification email|{color:green}✅ Tested and PASSED{color}|
|2|Code expires after 15 minutes|{color:green}✅ Tested and PASSED{color}|
|3|Clear error messages shown|{color:orange}🔍 Needs QA testing{color}|

*Summary:*
* Total ACs: 3
* Tested & Passed: 2
* Needs QA: 1
* Not Applicable: 0
* Failed: 0

----

h4. 🎭 E2E Test Results

*Test Suite:* tests/e2e/$TICKET.spec.ts
* {color:green}✅ Passed: 8{color}
* {color:red}❌ Failed: 0{color}
* ⏭ Skipped: 0
* ⏱ Duration: 45.2s

[View Full Report|file://playwright-report/index.html]

----

h4. 📊 Code Coverage

* Statements: 87% (required: 80%) {color:green}✅{color}
* Branches: 82% (required: 80%) {color:green}✅{color}
* Functions: 90% (required: 80%) {color:green}✅{color}
* Lines: 88% (required: 80%) {color:green}✅{color}

----

h4. ✅ Ready for PR?

*Status:* {color:green}*YES*{color}

All acceptance criteria verified and tests passing. Ready to create pull request.

_Verification Report: [.ac-verification/$TICKET/verification-report.md|file://.ac-verification/$TICKET/verification-report.md]_
```

### Step 5: Post to JIRA

Execute:
```bash
jira issue comment add $TICKET --comment="$(cat formatted-comment.txt)"
```

Handle errors:
- If ticket not found → show error
- If no permissions → show error
- If API error → retry once, then fail gracefully

### Step 6: Update Ticket Status (Optional)

Ask user: "Update ticket status based on results? (y/N)"

If yes and all ACs passed:
```bash
# Transition ticket to appropriate status
jira issue move $TICKET "Ready for QA"
```

If ACs need QA:
```bash
jira issue move $TICKET "QA Review"
```

Add labels:
```bash
jira issue label add $TICKET "ac-verified" "e2e-tested"
```

### Step 7: Confirmation

Display:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Posted to JIRA: $TICKET
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Actions taken:
✅ Posted AC verification results
✅ Posted E2E test results
✅ Posted code coverage
✅ Added labels: ac-verified, e2e-tested
✅ Updated status: Ready for QA

View in JIRA:
https://hellofresh.atlassian.net/browse/$TICKET
```

## Advanced Features

### A. Link CI Build Results

If GitHub Actions or Jenkins build exists:
```
h4. 🤖 CI Build Results

*Build:* [#1234|https://github.com/org/repo/actions/runs/1234]
* Status: {color:green}✅ Passed{color}
* Duration: 5m 32s
* All checks passed
```

### B. Link Screenshots/Videos

If test artifacts exist:
```
h4. 📸 Test Evidence

* [Screenshot: Happy Path|file://test-results/screenshot-1.png]
* [Video: Full Flow|file://test-results/video-1.webm]
* [Trace: Debug Info|file://test-results/trace-1.zip]
```

### C. Comparison with Previous Runs

If historical data exists:
```
h4. 📈 Compared to Last Run

* Tests: 8 (unchanged)
* Pass Rate: 100% (+12%)
* Duration: 45s (-3s, faster!)
```

### D. Tag Relevant Users

Mention assignee and watchers:
```
[~accountid:abc123] - Ready for your review!
```

## Error Handling

| Error | Solution |
|-------|----------|
| JIRA CLI not found | Install and configure |
| No verification report | Run /verify-ac first |
| Ticket not found | Check ticket ID |
| No permissions | Check JIRA_API_TOKEN |
| Network error | Retry or skip |

## Configuration

Optional `.ac-verification/config.yml`:
```yaml
jira:
  auto_post: true                    # Auto-post after /verify-ac
  auto_transition: true              # Auto-update status
  transition_on_all_passed: "Ready for QA"
  transition_on_needs_qa: "QA Review"
  add_labels: ["ac-verified", "e2e-tested"]
  mention_assignee: true
  include_coverage: true
  include_screenshots: true
```

## Integration with Other Commands

### Auto-post after /verify-ac

Update `/verify-ac` to ask:
```
"Post results to JIRA? (Y/n)"

If yes:
  → Run /post-to-jira $TICKET automatically
```

## Output Files

- `.ac-verification/$TICKET/jira-comment.txt` - Formatted comment
- `.ac-verification/$TICKET/jira-post-log.txt` - Post response log

## Usage Examples

### Basic Usage
```bash
/post-to-jira EPS-1234
```

### With Custom Message
```bash
/post-to-jira EPS-1234 --message="Additional notes: Fixed edge case bug"
```

### Skip Status Update
```bash
/post-to-jira EPS-1234 --no-transition
```

## Notes

- Respects JIRA markdown syntax
- Handles long comments (JIRA limit: 32,767 chars)
- Includes relative timestamps
- Links to local files for internal use
- Creates audit trail of all posts
