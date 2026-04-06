---
name: /generate-xray-tests
argument-hint: 'TICKET-123 [--format=bdd|manual] [--dry-run]'
description: Generate X-Ray test cases from JIRA ticket acceptance criteria
mode: single-agent
dependencies:
  - Atlassian MCP (required)
  - X-Ray for JIRA (required)
---

# Generate X-Ray Test Cases from JIRA Tickets

## Purpose
This command generates X-Ray test cases directly from a JIRA ticket's acceptance criteria using the Atlassian MCP. It extracts ACs from the ticket, converts them into test case format (BDD or Manual), and creates the test cases in X-Ray with proper linking back to the original ticket.

**Single Purpose:** Create X-Ray test cases only - no reports, no test executions, no test plans.

## Prerequisites Check

### Phase 0: Environment and Setup Validation

#### 1. Atlassian MCP Setup

Use the `CallMcpTool` to verify Atlassian MCP is available and authenticated:

```javascript
// Get user info to verify authentication
CallMcpTool({
  server: "atlassian",
  toolName: "atlassianUserInfo",
  arguments: {}
})
```

**If authentication fails:**
- The Atlassian MCP requires OAuth authentication
- You'll be prompted to authenticate via browser
- Follow the OAuth flow to grant access

#### 2. Get Accessible Resources

Fetch the list of accessible Atlassian sites:

```javascript
CallMcpTool({
  server: "atlassian",
  toolName: "getAccessibleAtlassianResources",
  arguments: {}
})
```

Store the `cloudId` from the response for subsequent API calls.

#### 3. X-Ray Integration Check

**Verify X-Ray is enabled:**
- Check that X-Ray app is installed in your JIRA instance
- Verify you have "Create Test" permissions in X-Ray
- X-Ray test types should be available in the project

**Note:** X-Ray test creation uses standard JIRA issue creation API with X-Ray-specific issue types.

#### 4. Ticket Validation

Verify the ticket exists and is accessible:

```javascript
CallMcpTool({
  server: "atlassian",
  toolName: "getJiraIssue",
  arguments: {
    cloudId: CLOUD_ID,
    issueIdOrKey: TICKET_KEY,
    fields: ["summary", "description", "customfield_*"],
    expand: "names"
  }
})
```

Check if ticket has acceptance criteria in description or custom fields.

## Workflow

### Step 1: Extract Acceptance Criteria from JIRA Ticket

1. **Fetch complete ticket details using Atlassian MCP:**

```javascript
const ticketData = CallMcpTool({
  server: "atlassian",
  toolName: "getJiraIssue",
  arguments: {
    cloudId: CLOUD_ID,
    issueIdOrKey: TICKET_KEY,
    fields: ["summary", "description", "project", "issuetype", "customfield_*"],
    fieldsByKeys: true,
    expand: "names",
    responseContentFormat: "markdown"
  }
})
```

2. **Extract Acceptance Criteria from ticket fields:**

Look for ACs in multiple locations (in order of priority):
- Custom field "Acceptance Criteria" (search in `customfield_*` fields)
- "Acceptance Criteria" section in description
- Structured requirements (look for patterns: "AC1:", "- AC:", "Given...When...Then")
- User story format ("As a...I want...So that")

Parse the content and create an array of individual acceptance criteria:

```javascript
const ACS_ARRAY = [
  { title: "AC1: User can login", content: "Given valid credentials, when user submits login form, then user is authenticated" },
  { title: "AC2: Invalid login shows error", content: "Given invalid credentials, when user submits, then error message appears" }
  // ... more ACs
]
```

If no structured ACs found, provide guidance:
```
⚠️  No clear acceptance criteria found in ticket.
   
   To use this command effectively, add acceptance criteria to your JIRA ticket in one of these formats:
   1. Add to custom "Acceptance Criteria" field
   2. Add section in description titled "Acceptance Criteria:"
   3. Use numbered format: AC1:, AC2:, etc.
   4. Use Gherkin format: Given...When...Then
```

### Step 2: Determine Test Case Format and Generate Content

1. **Analyze AC content to suggest format:**

```javascript
const hasGherkinPatterns = ACS_ARRAY.some(ac => 
  ac.content.match(/given.*when.*then/i)
)

let suggestedFormat = hasGherkinPatterns ? "bdd" : "manual"
```

2. **Ask user for format confirmation (if --format not specified):**

```
Detected ${ACS_ARRAY.length} acceptance criteria.
Suggested format: ${suggestedFormat} (${hasGherkinPatterns ? 'Gherkin patterns detected' : 'manual steps recommended'})

Proceed with ${suggestedFormat} format? (y/n/specify)
```

3. **Generate test case content for each AC:**

**For BDD Format:**
```gherkin
Feature: [TICKET] - [Summary]
  As a [user type from ticket]
  I want [main functionality]
  So that [business value]

Background:
  Given the system is in a valid state
  And the user has appropriate permissions

Scenario: [AC title]
  Given [preconditions from AC]
  When [action from AC]
  Then [expected result from AC]
```

**For Manual Format:**
```
Test Steps:
1. [Action derived from AC]
   Expected: [Expected result]
2. [Verification step]
   Expected: [Validation criteria]
3. [Final confirmation]
   Expected: [Success criteria]
```

### Step 3: Create X-Ray Test Cases Using Atlassian MCP

For each acceptance criteria, create an X-Ray test case in JIRA:

```javascript
const createdTests = []

for (const [index, ac] of ACS_ARRAY.entries()) {
  const testNumber = String(index + 1).padStart(3, '0')
  const testSummary = `${TICKET_KEY}-TC-${testNumber} - ${ac.title}`
  
  // Determine test type based on format
  const testType = TEST_FORMAT === 'bdd' ? 'Cucumber' : 'Manual'
  
  // Get project metadata for X-Ray test type
  const projectKey = TICKET_KEY.split('-')[0]
  
  // Get X-Ray test issue type ID
  const projectMetadata = CallMcpTool({
    server: "atlassian",
    toolName: "getJiraProjectIssueTypesMetadata",
    arguments: {
      cloudId: CLOUD_ID,
      projectKey: projectKey
    }
  })
  
  // Find X-Ray test issue type (look for "Test" issue type)
  const testIssueType = projectMetadata.issueTypes.find(
    type => type.name === 'Test' || type.name === 'X-Ray Test'
  )
  
  if (!testIssueType) {
    throw new Error(`X-Ray test issue type not found in project ${projectKey}. Ensure X-Ray is installed.`)
  }
  
  // Prepare test case content
  let testDescription = `Generated from JIRA ticket ${TICKET_KEY}\n\n**Original Acceptance Criteria:**\n${ac.content}`
  let testSteps = []
  
  if (TEST_FORMAT === 'bdd') {
    // For BDD, include Gherkin in description
    const gherkinContent = `
Feature: ${testSummary}

  Background:
    Given the system is in a valid state
    And the user has appropriate permissions

  Scenario: ${ac.title}
    ${ac.content}
    `.trim()
    
    testDescription += `\n\n**Gherkin Scenario:**\n\`\`\`gherkin\n${gherkinContent}\n\`\`\``
  } else {
    // For Manual, create structured test steps
    testSteps = [
      { 
        step: `Navigate to feature area: ${ac.title}`,
        result: "Feature area is accessible"
      },
      {
        step: `Execute: ${ac.content}`,
        result: "Action completes as specified in AC"
      },
      {
        step: "Verify expected outcome from acceptance criteria",
        result: "Expected behavior is observed and matches AC"
      }
    ]
  }
  
  // Create X-Ray test case
  try {
    const createResult = CallMcpTool({
      server: "atlassian",
      toolName: "createJiraIssue",
      arguments: {
        cloudId: CLOUD_ID,
        projectKey: projectKey,
        issueType: testIssueType.name,
        summary: testSummary,
        description: testDescription,
        labels: ["automated-generation", "ac-based", TEST_FORMAT],
        // X-Ray specific fields would go here if needed
        // For X-Ray test steps, you may need to use X-Ray REST API directly
      }
    })
    
    const testKey = createResult.key
    console.log(`✅ Created X-Ray test: ${testKey}`)
    createdTests.push(testKey)
    
    // Link test to original JIRA ticket
    CallMcpTool({
      server: "atlassian",
      toolName: "createIssueLink",
      arguments: {
        cloudId: CLOUD_ID,
        linkType: "Tests", // or "Test" depending on your JIRA configuration
        inwardIssueKey: TICKET_KEY,
        outwardIssueKey: testKey,
        comment: `Generated test case from acceptance criteria`
      }
    })
    
    console.log(`🔗 Linked ${testKey} to ${TICKET_KEY}`)
    
  } catch (error) {
    console.error(`❌ Failed to create test case for AC ${index + 1}: ${error.message}`)
  }
}
```

**Note on X-Ray Test Steps:** X-Ray stores test steps in custom fields. If the standard `createJiraIssue` doesn't support X-Ray test steps directly, you may need to:
1. Create the issue first
2. Use `editJiraIssue` to add X-Ray custom field data
3. Or use the X-Ray REST API directly via `fetchAtlassian` MCP tool

### Step 4: Handle Dry Run Mode

If `--dry-run` flag is provided, skip the creation step and display what would be created:

```
🔍 DRY RUN MODE - No tests will be created

Would create ${ACS_ARRAY.length} X-Ray test cases:

${ACS_ARRAY.map((ac, i) => `
Test ${i + 1}: ${TICKET_KEY}-TC-${String(i + 1).padStart(3, '0')}
Summary: ${ac.title}
Type: ${TEST_FORMAT}
Content: ${ac.content}
`).join('\n')}

To create these tests, run without --dry-run flag.
```

### Step 5: Display Results

```javascript
console.log(`
✅ Generated ${createdTests.length} X-Ray test cases from ${ACS_ARRAY.length} acceptance criteria
✅ All tests linked to ${TICKET_KEY}
✅ Test format: ${TEST_FORMAT}

Created tests:
${createdTests.map(key => `  - ${key}`).join('\n')}

View tests in JIRA: https://your-domain.atlassian.net/browse/${TICKET_KEY}
(Check the "Links" section for associated test cases)
`)
```

## Command Options

- `--format=bdd|manual`: Force specific test case format (default: auto-detect from AC content)
- `--dry-run`: Preview test cases without creating them in X-Ray
- `--environment=ENV`: Set target test environment in test metadata (if supported by X-Ray configuration)
- `--component=COMP`: Override component detection from ticket

## Error Handling

1. **Atlassian MCP Not Available:**
   ```
   ❌ Atlassian MCP not found or not authenticated.
   
   Please ensure:
   1. Atlassian MCP plugin is installed in Claude Code
   2. You've completed OAuth authentication
   3. Run the authentication flow if prompted
   ```

2. **Ticket Access Issues:**
   ```
   ❌ Could not access ticket ${TICKET_KEY}
   
   Possible causes:
   - Ticket doesn't exist
   - You don't have permission to view this ticket
   - Wrong cloud ID or project
   ```

3. **X-Ray Not Available:**
   ```
   ❌ X-Ray test issue type not found in project ${projectKey}
   
   Please ensure:
   - X-Ray for JIRA is installed in your instance
   - Project has X-Ray test issue types enabled
   - You have permission to create X-Ray tests
   ```

4. **AC Extraction Failures:**
   ```
   ⚠️  No acceptance criteria found in ticket
   
   Add acceptance criteria to your ticket:
   1. Use custom "Acceptance Criteria" field
   2. Add section in description: "## Acceptance Criteria"
   3. Use format: AC1:, AC2:, etc.
   4. Use Gherkin: Given...When...Then
   ```

5. **Test Creation Failures:**
   - Retry with exponential backoff
   - Provide detailed error messages from API
   - Suggest checking permissions and X-Ray configuration

## Example Usage

```bash
# Basic usage - extract ACs from ticket and generate X-Ray tests
/generate-xray-tests PROJ-123

# Force BDD format
/generate-xray-tests PROJ-123 --format=bdd

# Force Manual format
/generate-xray-tests PROJ-123 --format=manual

# Dry run to preview generated content before creating in X-Ray
/generate-xray-tests PROJ-123 --dry-run

# Create tests with specific environment metadata
/generate-xray-tests PROJ-123 --environment=staging
```

## Integration with Other Commands

This command works well with:
- `/collect-ac` - For extracting and refining acceptance criteria first
- `/document-tests` - For documenting test coverage after creation

## Success Metrics

- Number of X-Ray test cases successfully created
- Successful linking between X-Ray tests and original JIRA ticket
- Quality of generated test case content (scenarios/steps)
- Reduction in manual test case creation time

## Technical Notes

**Atlassian MCP Tools Used:**
- `atlassianUserInfo` - Verify authentication
- `getAccessibleAtlassianResources` - Get cloud ID
- `getJiraIssue` - Fetch ticket details
- `createJiraIssue` - Create X-Ray test cases
- `createIssueLink` - Link tests to original ticket
- `getJiraProjectIssueTypesMetadata` - Get X-Ray test type ID

**X-Ray Considerations:**
- X-Ray tests are special JIRA issue types
- Test steps may require X-Ray REST API for full functionality
- Some X-Ray features may need direct API calls via `fetchAtlassian` MCP tool
