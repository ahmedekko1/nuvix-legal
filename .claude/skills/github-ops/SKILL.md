# GitHub Ops Skill

Manage GitHub workflows using the `gh` CLI — issues, PRs, CI, releases, and security monitoring.

## Issue Triage

```bash
# List open issues by label
gh issue list --label "bug" --state open

# View issue detail
gh issue view 123

# Assign and label
gh issue edit 123 --add-assignee @me --add-label "priority:high"

# Close with comment
gh issue close 123 --comment "Fixed in #456"

# Create issue from template
gh issue create --title "Bug: ..." --body "..." --label "bug"
```

## Pull Request Management

```bash
# List PRs awaiting review
gh pr list --search "review:required"

# Check out PR locally
gh pr checkout 456

# View PR with diff
gh pr view 456 --web

# Review a PR
gh pr review 456 --approve --body "LGTM"
gh pr review 456 --request-changes --body "Please fix X"

# Merge strategies
gh pr merge 456 --squash --delete-branch
gh pr merge 456 --rebase
gh pr merge 456 --merge

# Check PR status / CI
gh pr checks 456
```

## CI / Workflow Operations

```bash
# List recent workflow runs
gh run list --limit 10

# Watch a running workflow
gh run watch

# View failed run logs
gh run view 789 --log-failed

# Re-run failed jobs only
gh run rerun 789 --failed

# Trigger a workflow manually
gh workflow run deploy.yml --field environment=staging

# List all workflows
gh workflow list
```

## Release Management

```bash
# Create a release
gh release create v1.2.0 --title "v1.2.0" --notes "## Changes\n- ..."

# Auto-generate release notes from merged PRs
gh release create v1.2.0 --generate-notes

# Upload assets to a release
gh release upload v1.2.0 ./dist/app.tar.gz

# List releases
gh release list

# View latest
gh release view
```

## Security Monitoring

```bash
# View Dependabot alerts
gh api repos/{owner}/{repo}/dependabot/alerts --jq '.[] | {id:.number, severity:.security_advisory.severity, pkg:.dependency.package.name}'

# View secret scanning alerts
gh api repos/{owner}/{repo}/secret-scanning/alerts

# View code scanning alerts
gh api repos/{owner}/{repo}/code-scanning/alerts
```

## Branch Protection

```bash
# View branch protection rules
gh api repos/{owner}/{repo}/branches/main/protection

# Require PR reviews before merge (via API)
gh api repos/{owner}/{repo}/branches/main/protection \
  --method PUT \
  --field required_pull_request_reviews='{"required_approving_review_count":1}'
```

## Useful Aliases

```bash
# Add to ~/.gitconfig or use gh alias set
gh alias set prs 'pr list --author @me'
gh alias set ready 'pr ready'
gh alias set co 'pr checkout'
```

## Common Patterns

**Draft PR → Ready → Merge flow:**
```bash
gh pr create --draft --title "WIP: feature"
# ... work ...
gh pr ready
gh pr merge --squash --delete-branch
```

**Hotfix flow:**
```bash
gh issue create --label "bug,priority:critical" --title "Production down: ..."
git checkout -b hotfix/issue-123
# ... fix ...
gh pr create --base main --title "Hotfix: ..." --body "Fixes #123"
gh pr merge --merge
gh release create v1.2.1 --generate-notes
```
