---
name: /setup-qa-assistant
argument-hint: ''
description: Check and install dependencies for QA workflow
mode: single-agent
---

# Setup QA Assistant

## Purpose
Check for required dependencies and guide installation of missing tools.

## Workflow

### Step 1: Welcome

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔧 QA Assistant Dependency Check
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Checking your machine for required dependencies...
```

### Step 2: Check Core Dependencies

**A. JIRA CLI (Required for JIRA integration)**

```bash
# Check if JIRA CLI is installed
which jira

if [ $? -eq 0 ]; then
  jira --version
  echo "✅ JIRA CLI installed"
  
  # Check if JIRA CLI is authenticated
  if [ -f "$HOME/.config/.jira/.config.yml" ]; then
    # Try a simple command to verify authentication
    jira me 2>/dev/null
    if [ $? -eq 0 ]; then
      echo "✅ JIRA CLI authenticated"
    else
      echo "⚠️ JIRA CLI installed but not authenticated"
      NEEDS_AUTH+=("jira")
    fi
  else
    echo "⚠️ JIRA CLI not configured (run: jira init)"
    NEEDS_AUTH+=("jira")
  fi
else
  echo "❌ JIRA CLI not found"
  MISSING_DEPS+=("jira-cli")
fi
```

**B. Git (Required)**

```bash
# Check git
git --version

if [ $? -eq 0 ]; then
  echo "✅ Git installed"
else
  echo "❌ Git not found"
  MISSING_DEPS+=("git")
fi
```

**C. GitHub CLI (Required for PR creation)**

```bash
# Check gh CLI
gh --version

if [ $? -eq 0 ]; then
  echo "✅ GitHub CLI installed"
  
  # Check if authenticated
  gh auth status 2>/dev/null
  if [ $? -eq 0 ]; then
    echo "✅ GitHub CLI authenticated"
  else
    echo "⚠️ GitHub CLI not authenticated"
    NEEDS_AUTH+=("gh")
  fi
else
  echo "❌ GitHub CLI not found"
  MISSING_DEPS+=("gh")
fi
```

**D. Node.js (Required for test frameworks)**

```bash
# Check Node.js
node --version

if [ $? -eq 0 ]; then
  NODE_VERSION=$(node --version)
  echo "✅ Node.js installed ($NODE_VERSION)"
  
  # Check if version is >= 18
  MAJOR_VERSION=$(echo $NODE_VERSION | cut -d. -f1 | sed 's/v//')
  if [ $MAJOR_VERSION -lt 18 ]; then
    echo "⚠️ Node.js 18+ recommended (current: $NODE_VERSION)"
  fi
else
  echo "❌ Node.js not found"
  MISSING_DEPS+=("node")
fi
```

### Step 3: Check Optional Dependencies

**A. Playwright (for Web E2E tests)**

```bash
# Check if Playwright is installed in project
if [ -f "package.json" ]; then
  if grep -q "@playwright/test" package.json; then
    echo "✅ Playwright installed in project"
  else
    echo "⚠️ Playwright not installed (optional for Web E2E)"
    OPTIONAL_DEPS+=("playwright")
  fi
fi
```

**B. Maestro (for React Native E2E tests)**

```bash
# Check if Maestro is installed
which maestro

if [ $? -eq 0 ]; then
  maestro --version
  echo "✅ Maestro installed"
else
  echo "⚠️ Maestro not installed (optional for React Native)"
  OPTIONAL_DEPS+=("maestro")
fi
```

**C. Detox (for React Native E2E tests)**

```bash
# Check if Detox is installed
if [ -f "package.json" ]; then
  if grep -q "detox" package.json; then
    echo "✅ Detox installed in project"
  else
    echo "⚠️ Detox not installed (optional for React Native)"
  fi
fi
```

**D. Figma MCP (for Figma integration)**

```bash
# Check if Figma MCP is configured
if [ -f "$HOME/.claude.json" ]; then
  if grep -q "figma" "$HOME/.claude.json"; then
    echo "✅ Figma MCP configured"
  else
    echo "⚠️ Figma MCP not configured (optional)"
    OPTIONAL_DEPS+=("figma-mcp")
  fi
fi
```

### Step 4: Display Results

Show summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Dependency Check Results
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Required Dependencies (4/4)
  ✅ Git
  ✅ Node.js
  ✅ JIRA CLI
  ✅ GitHub CLI

🔐 Authentication Status
  ✅ JIRA CLI authenticated
  ⚠️ GitHub CLI not authenticated

⚠️ Optional Dependencies (2/4)
  ✅ Playwright
  ✅ Figma MCP
  ❌ Maestro
  ❌ Detox
```

### Step 5: Offer Installation

If missing dependencies found, offer to install **each one individually**:

For each missing required dependency, ask:

```
❌ JIRA CLI is not installed (Required for JIRA integration)

Install instructions:
  macOS: brew install ankitpokhrel/jira-cli/jira-cli
  Linux: curl -sL https://github.com/ankitpokhrel/jira-cli/releases/latest/download/jira_linux_x86_64.tar.gz | tar xz

Would you like me to install JIRA CLI now? (Y/n)
  [Y] Install automatically
  [n] Skip (I'll install manually)
  [open] Open installation docs
```

**If user says "n" (no):**
- Show: `⏭️ Skipping JIRA CLI - you can install it manually later`
- Proceed immediately to the next missing dependency

**If user says "open":**
- Open the relevant installation docs URL
- After opening, ask again: `Would you like me to install it now? (Y/n)`
- If user says "n" after opening docs, proceed to next dependency

**If user says "Y" (yes):**
- Proceed with automatic installation

For each missing optional dependency, ask:

```
⚠️ Playwright is not installed (Optional for Web E2E tests)

Would you like me to install Playwright? (Y/n)
  [Y] Install now
  [n] Skip
  [open] Open Playwright docs
```

Same logic applies for optional dependencies.

### Step 6: Install Missing Dependencies

**Process each dependency in order:**
1. JIRA CLI (required)
2. GitHub CLI (required)
3. Playwright (optional)
4. Maestro (optional)
5. Figma MCP (optional)

**A. JIRA CLI**

```bash
echo "📦 Installing JIRA CLI..."

# Detect OS
if [[ "$OSTYPE" == "darwin"* ]]; then
  # macOS
  brew install ankitpokhrel/jira-cli/jira-cli
elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
  # Linux
  curl -sL https://github.com/ankitpokhrel/jira-cli/releases/latest/download/jira_linux_x86_64.tar.gz | tar xz
  sudo mv jira /usr/local/bin/
fi

echo "✅ JIRA CLI installed"
echo "Run 'jira init' to configure your JIRA instance"
```

**B. GitHub CLI**

```bash
echo "📦 Installing GitHub CLI..."

# Detect OS
if [[ "$OSTYPE" == "darwin"* ]]; then
  # macOS
  brew install gh
elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
  # Linux
  curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
  sudo apt-get update
  sudo apt-get install gh
fi

echo "✅ GitHub CLI installed"
echo "Run 'gh auth login' to authenticate"
```

**C. Playwright (if user wants)**

```bash
echo "📦 Installing Playwright..."
npm install -D @playwright/test
npx playwright install

echo "✅ Playwright installed"
```

**D. Maestro (if user wants)**

```bash
echo "📦 Installing Maestro..."
curl -Ls "https://get.maestro.mobile.dev" | bash

echo "✅ Maestro installed"
echo "Add to PATH: export PATH=$PATH:$HOME/.maestro/bin"
```

**E. Figma MCP**

```bash
echo "📦 Configuring Figma MCP..."

# Run the setup command
claude mcp add --scope user --transport http figma https://mcp.figma.com/mcp

echo "✅ Figma MCP configured"
echo "Restart Claude Code and authenticate: /mcp"
```

### Installation Documentation URLs

When user selects "open" option, open these URLs:

| Dependency | Documentation URL |
|-----------|------------------|
| JIRA CLI | https://github.com/ankitpokhrel/jira-cli#installation |
| GitHub CLI | https://cli.github.com/manual/installation |
| Node.js | https://nodejs.org/en/download |
| Playwright | https://playwright.dev/docs/intro |
| Maestro | https://maestro.mobile.dev/getting-started/installing-maestro |
| Figma MCP | https://github.com/figma/mcp-server-figma#installation |

### Step 7: Authentication Setup

After all installations, check if any tools need authentication:

**If JIRA CLI needs authentication:**

```
🔐 JIRA CLI Authentication Required

JIRA CLI is installed but not configured.

Would you like me to guide you through JIRA setup? (Y/n)
  [Y] Start jira init
  [n] Skip (I'll configure later)
  [open] Open JIRA CLI docs
```

**If user says "Y":**

```bash
echo "🔐 Starting JIRA CLI configuration..."
echo ""
echo "You'll need:"
echo "  • Your JIRA instance URL (e.g., https://yourcompany.atlassian.net)"
echo "  • Your JIRA login type (browser, api-token, or basic)"
echo "  • API token (recommended): https://id.atlassian.com/manage-profile/security/api-tokens"
echo ""
echo "Press Enter to start..."
read

# Run interactive setup
jira init

# Verify authentication
if jira me 2>/dev/null; then
  echo "✅ JIRA CLI authenticated successfully!"
  JIRA_USER=$(jira me --plain 2>/dev/null | head -n1)
  echo "   Logged in as: $JIRA_USER"
else
  echo "❌ JIRA authentication failed"
  echo "   Try running: jira init"
fi
```

**If user says "open":**
- Open: `https://github.com/ankitpokhrel/jira-cli#authentication`
- Then ask again: `Would you like me to start jira init now? (Y/n)`

**If JIRA CLI authentication fails, show troubleshooting:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️ JIRA Authentication Failed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Common issues:

1. Invalid API token
   → Create new token: https://id.atlassian.com/manage-profile/security/api-tokens
   → Run: jira init

2. Wrong JIRA URL format
   ✅ Correct: https://yourcompany.atlassian.net
   ❌ Wrong: https://yourcompany.atlassian.net/browse

3. Network/firewall issues
   → Check VPN connection
   → Test access: curl -I https://yourcompany.atlassian.net

4. Permission issues with config file
   → Check: ls -la ~/.config/.jira/
   → Fix: chmod 600 ~/.config/.jira/.config.yml

Try again: jira init
```

**If GitHub CLI needs authentication:**

```
🔐 GitHub CLI Authentication Required

Would you like me to authenticate GitHub CLI? (Y/n)
  [Y] Start gh auth login
  [n] Skip (I'll authenticate later)
```

**If user says "Y":**

```bash
echo "🔐 Starting GitHub CLI authentication..."
echo ""
echo "You'll be prompted to:"
echo "  • Choose GitHub.com or GitHub Enterprise"
echo "  • Select HTTPS or SSH"
echo "  • Authenticate via browser (recommended)"
echo ""

# Run interactive authentication
gh auth login

# Verify authentication
if gh auth status 2>/dev/null; then
  echo "✅ GitHub CLI authenticated successfully!"
  GH_USER=$(gh api user -q .login 2>/dev/null)
  echo "   Logged in as: $GH_USER"
else
  echo "❌ GitHub authentication failed"
  echo "   Try running: gh auth login"
fi
```

**If Figma MCP needs authentication:**

```
🔐 Figma MCP Authentication Required

Figma MCP is configured but needs authentication.

Would you like me to guide you? (Y/n)
  [Y] Open Figma authentication
  [n] Skip (I'll authenticate later)
```

**If user says "Y":**

```bash
echo "🔐 Figma MCP Authentication..."
echo ""
echo "Steps:"
echo "  1. Restart Claude Code (Cmd+Q and reopen)"
echo "  2. In Claude Code chat, type: /mcp"
echo "  3. Select 'figma' from the list"
echo "  4. Click 'Authenticate' and follow browser prompts"
echo ""
echo "After authentication, you can use: /figma-ac-extractor"
```

### Step 8: Configuration Guidance

After installation and authentication, provide setup summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎉 Setup Complete!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Installed & Configured:
  ✅ JIRA CLI (authenticated as: user@example.com)
  ✅ GitHub CLI (authenticated as: username)
  ✅ Playwright

📝 Manual Steps Remaining:
  ⚠️ Figma MCP - Needs authentication (restart Claude Code → /mcp)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🚀 Ready to start!

Try your first workflow:
  /collect-ac EPS-1234

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If any authentication was skipped:

```
⚠️ Skipped Authentication:
  • JIRA CLI - Run: jira init
  • GitHub CLI - Run: gh auth login

You'll need to authenticate before using:
  • /collect-ac (requires JIRA)
  • /post-to-jira (requires JIRA)
  • /create-pr (requires GitHub)
```

### Step 9: Save Configuration

Create a local config file to track setup:

```bash
# Create .qa-assistant-config
cat > .qa-assistant-config.json << EOF
{
  "setupComplete": true,
  "dependencies": {
    "jira-cli": {
      "installed": true,
      "authenticated": true,
      "user": "$(jira me --plain 2>/dev/null | head -n1 || echo 'not-authenticated')"
    },
    "gh": {
      "installed": true,
      "authenticated": true,
      "user": "$(gh api user -q .login 2>/dev/null || echo 'not-authenticated')"
    },
    "playwright": {
      "installed": true
    },
    "figma-mcp": {
      "configured": true,
      "authenticated": false
    }
  },
  "setupDate": "$(date -u +"%Y-%m-%dT%H:%M:%SZ")",
  "platform": "$(uname)"
}
EOF

echo "✅ Configuration saved to .qa-assistant-config.json"
```

## Error Handling

| Issue | Solution |
|-------|----------|
| Homebrew not found (macOS) | Provide manual installation instructions |
| Permission denied | Suggest using sudo or checking permissions |
| Network issues | Suggest checking internet connection |
| Already installed | Skip and show current version |

## Output

Configuration file: `.qa-assistant-config.json`

## Usage

Run this command:
- First time using QA Assistant
- After cloning repo on new machine
- When dependencies seem broken
- To check what's installed

## Example Flow

**Scenario 1: Missing JIRA CLI (not installed)**

```
❌ JIRA CLI is not installed (Required for JIRA integration)

Install instructions:
  macOS: brew install ankitpokhrel/jira-cli/jira-cli

Would you like me to install JIRA CLI now? (Y/n)
> Y

📦 Installing JIRA CLI...
✅ JIRA CLI installed

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔐 JIRA CLI Authentication Required

JIRA CLI is installed but not configured.

Would you like me to guide you through JIRA setup? (Y/n)
> Y

🔐 Starting JIRA CLI configuration...

You'll need:
  • Your JIRA instance URL (e.g., https://yourcompany.atlassian.net)
  • Your JIRA login type (browser, api-token, or basic)
  • API token (recommended): https://id.atlassian.com/manage-profile/security/api-tokens

Press Enter to start...

[Interactive jira init runs...]

✅ JIRA CLI authenticated successfully!
   Logged in as: john.doe@company.com
```

**Scenario 2: JIRA installed but not authenticated**

```
✅ JIRA CLI installed
⚠️ JIRA CLI installed but not authenticated

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔐 JIRA CLI Authentication Required

Would you like me to guide you through JIRA setup? (Y/n)
> open

Opening: https://github.com/ankitpokhrel/jira-cli#authentication

Would you like me to start jira init now? (Y/n)
> n

⏭️ Skipping JIRA authentication - you can run 'jira init' later
```

**Scenario 3: Authentication fails**

```
🔐 Starting JIRA CLI configuration...
[User enters credentials...]

❌ JIRA authentication failed

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️ JIRA Authentication Failed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Common issues:

1. Invalid API token
   → Create new token: https://id.atlassian.com/manage-profile/security/api-tokens
   → Run: jira init

2. Wrong JIRA URL format
   ✅ Correct: https://yourcompany.atlassian.net
   ❌ Wrong: https://yourcompany.atlassian.net/browse

3. Network/firewall issues
   → Check VPN connection
   → Test access: curl -I https://yourcompany.atlassian.net

Try again: jira init
```

**Scenario 4: Skip authentication for later**

```
⚠️ Playwright is not installed (Optional for Web E2E tests)

Would you like me to install Playwright? (Y/n)
> n

⏭️ Skipping Playwright - you can install it manually later

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🎉 Setup Complete!

⚠️ Skipped Authentication:
  • JIRA CLI - Run: jira init

You'll need to authenticate before using:
  • /collect-ac (requires JIRA)
  • /post-to-jira (requires JIRA)
```
