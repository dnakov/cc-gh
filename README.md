# Claude Code GitHub Action - Use Your Claude Subscription

This repository contains a modified version of Anthropic's Claude Code GitHub Action that **removes API key validation requirements**, allowing you to use your existing **Claude subscription** directly in GitHub Actions without needing separate API keys for Anthropic API, AWS Bedrock, or Google Vertex AI.

## 🎯 What This Project Does

- **Use Your Claude Subscription**: Leverage your existing Claude Pro/Team subscription instead of separate API keys
- **Removes API Key Barriers**: Completely bypasses the requirement for `ANTHROPIC_API_KEY`, AWS credentials, or Google Cloud credentials  
- **Maintains Full Functionality**: All other Claude Code features remain intact
- **Secure Credential Handling**: Creates credential files in the container without logging sensitive content
- **Custom Forks**: Uses forked versions of both the main action and base action for complete control

## 💡 Why This Matters

Instead of needing to:
- ❌ Set up separate Anthropic API keys
- ❌ Configure AWS Bedrock credentials  
- ❌ Manage Google Vertex AI authentication
- ❌ Pay for additional API usage

You can now:
- ✅ **Use your existing Claude subscription**
- ✅ **No additional API costs**
- ✅ **Simplified authentication**
- ✅ **Same Claude capabilities you're already paying for**

## 🏗️ Architecture

This project consists of three main components:

### 1. **Forked Base Action** (`dnakov/claude-code-base-action`)
- **Original**: `anthropics/claude-code-base-action`
- **Modification**: Removed all validation checks from `src/validate-env.ts`
- **Commit**: `8d5f53ef1aaaabc6af13cf47681246a1f124460f`

### 2. **Forked Main Action** (`dnakov/claude-code-action`)
- **Original**: `anthropics/claude-code-action`
- **Modification**: Updated to use the forked base action
- **Commit**: `841a525627187f2b98aea2688c57f627cb4a0548`

### 3. **GitHub Workflow** (`.github/workflows/claude.yml`)
- Uses the forked main action
- Creates secure credential files without logging
- Unsets environment variables for clean execution

## 📁 Project Structure

```
cc-gh/
├── .github/workflows/
│   └── claude.yml              # Main workflow file
├── claude-code-action/         # Forked main action
│   └── action.yml             # Modified to use forked base
├── claude-code-base-action/    # Forked base action
│   └── src/validate-env.ts    # Validation removed
├── claude-instructions.md      # Instructions for Claude
└── README.md                  # This file
```

## 🚀 How to Use

### Step 1: Set Up Repository Secrets

1. Go to your repository **Settings** → **Secrets and variables** → **Actions**
2. Add a new repository secret named `CREDENTIALS_JSON`
3. **Use your Claude subscription credentials** (session tokens, auth cookies, etc.) - format as needed for your authentication method

> **Note**: This bypasses the need for separate Anthropic API keys, AWS Bedrock, or Google Vertex AI credentials. You're using your existing Claude subscription instead.

### Step 2: Trigger the Action

The action is triggered by mentioning `@claude` in:
- **Issue comments**
- **Pull request comments**  
- **Pull request reviews**
- **Issue titles or descriptions**

### Step 3: Examples

#### Basic Usage
```
@claude help me fix this bug
```

#### Code Implementation
```
@claude implement a function that calculates fibonacci numbers
```

#### Code Review
```
@claude review this pull request and suggest improvements
```

## 🔧 Workflow Configuration

The workflow (`.github/workflows/claude.yml`) includes several key features:

### Secure Credential Handling
```yaml
- name: Create Claude credentials file
  env:
    CREDENTIALS_JSON: ${{ secrets.CREDENTIALS_JSON }}
  run: |
    mkdir -p ~/.claude
    printf '%s' "$CREDENTIALS_JSON" > ~/.claude/.credentials.json
    chmod 600 ~/.claude/.credentials.json
```

### Environment Variable Management
```yaml
- name: Run Claude Code
  env:
    CLAUDE_CODE_ACTION: ""  # Unset for clean execution
  uses: dnakov/claude-code-action@841a525627187f2b98aea2688c57f627cb4a0548
```

### Trigger Conditions
```yaml
if: |
  (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
  (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
  (github.event_name == 'pull_request_review' && contains(github.event.review.body, '@claude')) ||
  (github.event_name == 'issues' && (contains(github.event.issue.body, '@claude') || contains(github.event.issue.title, '@claude')))
```

## 🔐 Security Features

### 1. **No API Key Logging**
- Uses `printf '%s'` for secure file writing
- Environment variables are masked in GitHub Actions logs
- Credentials never appear in workflow output

### 2. **Proper File Permissions**
- `chmod 600` restricts file access to owner only
- Files are created in temporary container environment
- Automatic cleanup after workflow completion

### 3. **Environment Isolation**
- `CLAUDE_CODE_ACTION=""` ensures clean execution
- No interference with other environment variables
- Isolated container execution

## 🛠️ Customization Options

### Change Trigger Phrase
Modify the trigger phrase in the workflow:
```yaml
# trigger_phrase: "/claude"  # Uncomment and modify
```

### Add Custom Instructions
Provide specific instructions for Claude:
```yaml
# custom_instructions: |
#   Follow our coding standards
#   Ensure all new code has tests
#   Use TypeScript for new files
```

### Configure Allowed Tools
Specify which tools Claude can use:
```yaml
# allowed_tools: "Bash(npm install),Bash(npm run build),Bash(npm run test:*)"
```

## 📋 What Was Modified

### In `claude-code-base-action/src/validate-env.ts`:
**Removed this entire validation block:**
```typescript
if (!useBedrock && !useVertex) {
  if (!anthropicApiKey) {
    errors.push("ANTHROPIC_API_KEY is required when using direct Anthropic API.");
  }
} else if (useBedrock) {
  // AWS Bedrock validation removed
} else if (useVertex) {
  // Google Vertex AI validation removed
}
```

**Replaced with:**
```typescript
// Validation checks removed
```

### In `claude-code-action/action.yml`:
**Changed:**
```yaml
uses: dnakov/claude-code-base-action@8d5f53ef1aaaabc6af13cf47681246a1f124460f # Modified version without validation
```

## 🔄 Workflow Execution Flow

1. **Trigger Detection**: Action detects `@claude` mention
2. **Repository Checkout**: Code is checked out with `fetch-depth: 1`
3. **Credential Setup**: `~/.claude/.credentials.json` is created securely using your Claude subscription
4. **Environment Cleanup**: `CLAUDE_CODE_ACTION` is unset
5. **Claude Execution**: Forked action runs without API key validation
6. **Results**: Claude processes the request using your existing subscription

## 🌟 Benefits

- ✅ **Use Your Existing Subscription**: Leverage Claude Pro/Team subscription you already pay for
- ✅ **No Additional API Costs**: No separate API usage charges
- ✅ **Full Feature Set**: All Claude Code capabilities remain available
- ✅ **Secure Execution**: Credentials handled safely without logging
- ✅ **Easy Setup**: Just add your Claude credentials as a repository secret
- ✅ **Version Control**: Pinned to specific commit hashes for reliability

## 🔗 Repository Links

- **Main Action Fork**: https://github.com/dnakov/claude-code-action
- **Base Action Fork**: https://github.com/dnakov/claude-code-base-action
- **This Repository**: https://github.com/dnakov/cc-gh

## 📝 Notes

- The forked actions are pinned to specific commit hashes for stability
- Credential files are temporary and cleaned up after workflow execution
- All modifications maintain the original action's functionality while removing API key validation barriers
- **This setup allows you to use your Claude subscription directly**, avoiding the need for separate API credentials
- Perfect for teams already using Claude Pro/Team who want to extend that access to GitHub Actions

## 🤝 Contributing

Feel free to:
- Report issues or bugs
- Suggest improvements
- Submit pull requests
- Fork for your own modifications

---

**Disclaimer**: This project modifies Anthropic's Claude Code Action to remove API key validation, allowing you to use your existing Claude subscription. Use responsibly and ensure you have proper authorization for AI model usage in your environment.