# Integration Guides

Pandorian integrates into your existing development workflow to enforce guidelines continuously and automatically. This guide covers setting up PR scanning, CI/CD integration, notifications, and more.

## GitHub App & Pull Request Scanning

The fastest way to enforce guidelines is through automatic PR scanning via the GitHub App.

### How PR Scanning Works

Once you install the Pandorian GitHub App (done during initial setup), you control PR scanning at the **guideline level**. This means you explicitly choose which guidelines should run automatically on pull requests.

**What happens during a PR scan:**

1. Developer opens or updates a pull request
2. Pandorian scans only the modified files against guidelines configured for PR scanning
3. Violations appear in two places:
   - **Inline comments** on specific lines of code where violations occur
   - **Summary comment** on the PR with an overview of all violations found
4. The PR check appears in GitHub's status checks (can be configured as required or optional)

### Configuring Which Guidelines Run on PRs

PR scanning is controlled at the **individual guideline level**, giving you granular control over what gets enforced and when. When creating or editing a guideline:

1. Navigate to the **Guidelines** page
2. Create or edit a guideline
3. Look for the **Enforce On** field (or scan level configuration)
4. Choose the scope:
   - **Full Scan** - Runs only during manual full repository scans
   - **Repository Scan** - Runs during repository scans (same as Full)
   - **PR Scan** - Runs automatically on every pull request
   - **All** - Runs on both full scans and PR scans (recommended for most guidelines)

**Best Practice:** Set critical guidelines (security, production-readiness, API design standards) to **PR Scan** or **All** to catch issues before they're merged. Use **Full Scan** for broader quality checks that don't need immediate PR enforcement.

### Managing PR Scan Results

When violations are found on a PR:

**Inline Comments:**
- Appear directly on the lines of code that violate guidelines
- Include the guideline name, severity, and brief explanation
- Link back to the full guideline in Pandorian dashboard

**Summary Comment:**
- Posted at the PR level with a complete list of violations
- Grouped by file and guideline
- Includes violation counts and severity breakdown

**Status Check:**
- Appears in GitHub's checks section
- Can be configured as a required check to block merges (recommended)
- Links to detailed scan results in Pandorian dashboard

### Repository-Level Configuration

To adjust which repositories use PR scanning:

1. Navigate to the **Repositories** page in Pandorian
2. Find your repository
3. Toggle PR scanning on/off (enabled by default)
4. Configure whether PR scan failures should block merges (in GitHub's branch protection rules)

## CI/CD Integration

For teams that want custom enforcement workflows, Pandorian provides an API for direct integration into CI/CD pipelines.

### Integration Method

Pandorian uses **direct API calls** to trigger scans and retrieve results. This gives you full control over:
- When scans run (on commit, nightly, before deploy, etc.)
- How failures are handled (block pipeline, send notifications, generate reports)
- Which guidelines to enforce in different environments

### GitHub Actions Integration

Here's an example workflow that runs Pandorian scans on every push and blocks the pipeline if violations are found:

```yaml
name: Pandorian Code Enforcement

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  pandorian-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Trigger Pandorian Scan
        env:
          PANDORIAN_API_KEY: ${{ secrets.PANDORIAN_API_KEY }}
          PANDORIAN_ORG_ID: ${{ secrets.PANDORIAN_ORG_ID }}
        run: |
          SCAN_ID=$(curl -X POST https://api.pandorian.ai/v1/scans \
            -H "Authorization: Bearer $PANDORIAN_API_KEY" \
            -H "Content-Type: application/json" \
            -d '{
              "organization_id": "'$PANDORIAN_ORG_ID'",
              "repository": "'$GITHUB_REPOSITORY'",
              "commit": "'$GITHUB_SHA'",
              "scan_type": "commit"
            }' | jq -r '.scan_id')
          
          echo "SCAN_ID=$SCAN_ID" >> $GITHUB_ENV

      - name: Wait for Scan Completion
        env:
          PANDORIAN_API_KEY: ${{ secrets.PANDORIAN_API_KEY }}
        run: |
          while true; do
            STATUS=$(curl -s https://api.pandorian.ai/v1/scans/$SCAN_ID \
              -H "Authorization: Bearer $PANDORIAN_API_KEY" | jq -r '.status')
            
            if [ "$STATUS" = "completed" ]; then
              break
            elif [ "$STATUS" = "failed" ]; then
              echo "Scan failed"
              exit 1
            fi
            
            sleep 10
          done

      - name: Check for Violations
        env:
          PANDORIAN_API_KEY: ${{ secrets.PANDORIAN_API_KEY }}
        run: |
          VIOLATIONS=$(curl -s https://api.pandorian.ai/v1/scans/$SCAN_ID/violations \
            -H "Authorization: Bearer $PANDORIAN_API_KEY" | jq '.total_violations')
          
          if [ $VIOLATIONS -gt 0 ]; then
            echo "Found $VIOLATIONS guideline violations"
            echo "View details: https://app.pandorian.ai/scans/$SCAN_ID"
            exit 1
          else
            echo "No violations found"
          fi
```

**Configuration:**
1. Generate an API key from Pandorian dashboard (Settings → API Keys)
2. Add secrets to your GitHub repository:
   - `PANDORIAN_API_KEY`
   - `PANDORIAN_ORG_ID`
3. Commit the workflow file to `.github/workflows/pandorian.yml`

### CircleCI Integration

```yaml
version: 2.1

jobs:
  pandorian-scan:
    docker:
      - image: cimg/base:stable
    steps:
      - checkout
      
      - run:
          name: Trigger Pandorian Scan
          command: |
            SCAN_ID=$(curl -X POST https://api.pandorian.ai/v1/scans \
              -H "Authorization: Bearer $PANDORIAN_API_KEY" \
              -H "Content-Type: application/json" \
              -d '{
                "organization_id": "'$PANDORIAN_ORG_ID'",
                "repository": "'$CIRCLE_PROJECT_USERNAME/$CIRCLE_PROJECT_REPONAME'",
                "commit": "'$CIRCLE_SHA1'",
                "scan_type": "commit"
              }' | jq -r '.scan_id')
            
            echo "export SCAN_ID=$SCAN_ID" >> $BASH_ENV
      
      - run:
          name: Wait for Scan and Check Results
          command: |
            # Wait for completion (similar logic to GitHub Actions)
            # Check violations and exit with code 1 if found

workflows:
  main:
    jobs:
      - pandorian-scan
```

### Blocking vs Reporting Mode

**Blocking Mode** (recommended for production):
- Pipeline fails if violations are found
- Prevents non-compliant code from being deployed
- Set exit code 1 when violations > 0

**Reporting Mode** (useful for gradual rollout):
- Pipeline continues regardless of violations
- Scan results logged for visibility
- Set exit code 0 always, just log violations

Choose your mode by adjusting the final step's exit code behavior.

## Slack Integration

Get notified when scans complete and violations are found.

### Setting Up Slack Notifications

1. Navigate to **Settings → Integrations** in Pandorian dashboard
2. Click **Connect Slack**
3. Authorize Pandorian to access your Slack workspace
4. Choose which channel receives notifications
5. Configure notification preferences:
   - Scan completion (all scans or only failed scans)
   - New violations found
   - Scan summary reports

### Notification Types

**Scan Completion:**
- Triggered when any scan finishes
- Includes violation count, severity breakdown, and link to results

**New Violations:**
- Alerts when new violations are introduced (PR scans)
- Shows file paths and guideline names

**Daily/Weekly Summaries:**
- Aggregated reports of organization-wide compliance
- Trends and improvement metrics

## Enforcement Layer Positioning

Pandorian is designed as an **enforcement layer that sits above your development tools**—agnostic to how code is written (Cursor, Windsurf, manual reviews, AI review tools) and flexible in how it's integrated.

### Integration Models

**Model 1: Parallel to CI/CD**
- Pandorian runs alongside your existing CI/CD pipeline
- Catches issues that traditional linters and tests miss
- Works as an additional required check before merge

**Model 2: As a CI/CD Check**
- Pandorian integrated directly into your pipeline (via API)
- Runs after tests, before deployment
- Blocks deployment if critical violations found

**Model 3: GitHub App Only**
- Simplest setup—no CI/CD configuration required
- Automatic enforcement on all pull requests
- Developers see violations immediately in their workflow

Most teams use a combination: GitHub App for PR feedback + CI/CD integration for pre-deployment validation.

## VS Code Extension (Coming Soon)

We're building a VS Code extension that will bring Pandorian directly into your editor:

**Planned Features:**
- Real-time guideline enforcement as you write code
- Inline suggestions and quick fixes
- Browse and search your organization's guideline catalog
- One-click scanning of open files or workspace
- Integration with AI coding assistants

Stay tuned for the beta release.

## Best Practices

**Start with PR Scanning:**
Enable automatic PR scanning first—this gives developers immediate feedback without additional configuration.

**Use CI/CD for Critical Paths:**
Add CI/CD integration for production deployments where you need absolute enforcement.

**Configure Severity Appropriately:**
- **High severity** → Block merges/deployments
- **Medium severity** → Require review/acknowledgment
- **Low severity** → Report only, don't block

**Gradual Rollout:**
When introducing Pandorian to an existing codebase, start in reporting mode, identify violation trends, then enable blocking gradually.

## Troubleshooting

**PR scans not running:**
- Verify GitHub App is installed for the repository
- Check that guidelines have PR Scan enforcement enabled
- Confirm repository has PR scanning active (Repositories page)

**CI/CD pipeline failing:**
- Verify API key is valid and has correct permissions
- Check API endpoint URLs are correct
- Ensure scan completion polling doesn't timeout

**Too many false positives:**
- Refine guideline descriptions to be more specific
- Adjust enforcement scope (Full vs PR vs All)
- Contact support for guideline tuning assistance (Pro and Enterprise tiers only)

## Need Help?

- API documentation: [API Reference](/api-reference)
- Guideline management: [Managing Guidelines](/managing-guidelines)
- Contact support: support@pandorian.ai (Pro and Enterprise tiers only)