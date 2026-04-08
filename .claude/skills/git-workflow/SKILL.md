# Git Workflow Skill

Branch strategy, commit conventions, and collaboration patterns for clean git history.

## Branch Naming

```
main          — production-ready code, always deployable
develop       — integration branch (if using gitflow)

feature/      — new features
  feature/user-auth
  feature/stripe-integration

fix/          — bug fixes
  fix/login-redirect
  fix/null-pointer-123

hotfix/       — urgent production fixes
  hotfix/payment-timeout

chore/        — maintenance, deps, config
  chore/upgrade-dependencies
  chore/add-eslint-rule

docs/         — documentation only
  docs/api-reference

release/      — release preparation
  release/v2.1.0
```

## Commit Message Format

Follow Conventional Commits:

```
<type>(<scope>): <short summary>

[optional body]

[optional footer: Fixes #123]
```

**Types:**
- `feat` — new feature
- `fix` — bug fix
- `chore` — maintenance, no production change
- `docs` — documentation
- `style` — formatting, no logic change
- `refactor` — restructure without behavior change
- `test` — add or update tests
- `perf` — performance improvement
- `ci` — CI/CD pipeline changes

**Examples:**
```
feat(auth): add OAuth2 login with Google
fix(api): handle null response from Stripe webhook
chore(deps): upgrade Next.js to 14.2.0
docs(readme): add local setup instructions
test(auth): add unit tests for token refresh logic
```

## Daily Workflow

```bash
# Start new work
git checkout main && git pull
git checkout -b feature/my-feature

# Work in small commits
git add src/components/Button.tsx
git commit -m "feat(ui): add loading state to Button component"

# Keep branch up to date
git fetch origin main
git rebase origin/main

# Push
git push -u origin feature/my-feature
```

## Clean History with Rebase

```bash
# Squash last N commits before PR
git rebase -i HEAD~3

# In the interactive editor:
# pick abc1234 feat: initial implementation
# squash def5678 fix typo
# squash ghi9012 remove debug log

# Rebase onto main (instead of merge)
git rebase origin/main
```

## Merge Strategies

| Strategy | When to use |
|----------|-------------|
| Squash merge | Feature branches — clean linear history |
| Rebase merge | Clean series of logical commits to preserve |
| Merge commit | Long-lived branches (release, develop) |

Prefer **squash** for feature branches. One PR = one squashed commit on main.

## Conflict Resolution

```bash
# Rebase and fix conflicts step by step
git rebase origin/main

# For each conflict:
# 1. Edit the file to resolve
# 2. git add <file>
# 3. git rebase --continue

# Abort if needed
git rebase --abort
```

## Tagging Releases

```bash
# Annotated tag (preferred)
git tag -a v1.2.0 -m "Release v1.2.0: add dashboard and billing"

# Push tag
git push origin v1.2.0

# Push all tags
git push origin --tags
```

## Useful Aliases

```bash
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.st "status -s"
git config --global alias.co "checkout"
git config --global alias.br "branch"
git config --global alias.undo "reset HEAD~1 --mixed"
```

## Pre-commit Checklist

Before pushing:
- [ ] `git diff --stat` — review what's changing
- [ ] Tests pass locally
- [ ] No `console.log`, `debugger`, or TODO left in committed code
- [ ] `.env` not staged (`git status` check)
- [ ] Commit message follows Conventional Commits format
