# Git Conventions for AI-Assisted Development

Patterns for Git workflows when working with Claude Code and similar tools.

## Commit Messages

### Format

```
<summary line - what changed>

<optional body - why it changed, context>

Co-Authored-By: Claude <model> <noreply@anthropic.com>
```

### Summary Line Guidelines

- Use imperative mood ("Add feature" not "Added feature")
- Keep under 70 characters
- Focus on *what* changed, not *how*
- Don't end with a period

**Good examples:**
```
Add user authentication via JWT
Fix null pointer crash in config parser
Update dependencies to latest versions
Refactor database layer for testability
```

**Avoid:**
```
Fixed bug                          # Too vague
Updated the code to add the new feature for users  # Too long, wrong tense
Changes.                           # Meaningless
```

### Body Guidelines

When the summary isn't enough, add a body:

```
Add rate limiting to API endpoints

Prevents abuse by limiting requests to 100/minute per IP.
Uses token bucket algorithm with Redis backend.

Closes #123

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

### Co-Author Attribution

When AI assists with code, include attribution:

```
Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
Co-Authored-By: Claude Sonnet 4 <noreply@anthropic.com>
```

## Commit Workflow

### Before Committing

1. **Review changes**: `git diff` and `git status`
2. **Stage intentionally**: Add specific files, not `git add -A`
3. **Check for secrets**: Never commit `.env`, credentials, API keys
4. **Run tests**: Ensure changes don't break existing functionality

### Staging Best Practices

```bash
# Stage specific files (preferred)
git add src/auth.c src/auth.h

# Stage with review
git add -p  # Interactive patch mode

# Avoid - can include unintended files
git add -A
git add .
```

### Creating Commits

```bash
# For multi-line messages, use heredoc
git commit -m "$(cat <<'EOF'
Add rate limiting to API endpoints

Prevents abuse by limiting requests to 100/minute per IP.

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"

# Or let editor open for complex messages
git commit  # Opens $EDITOR
```

## Branching

### Branch Naming

```
feature/user-authentication
fix/null-pointer-config
refactor/database-layer
docs/api-reference
```

### Creating Feature Branches

```bash
# From main/master
git checkout main
git pull
git checkout -b feature/new-feature
```

### Keeping Branches Updated

```bash
# Rebase onto latest main (cleaner history)
git fetch origin
git rebase origin/main

# Or merge (preserves branch history)
git merge origin/main
```

## Pull Requests

### PR Title

- Keep under 70 characters
- Same conventions as commit summary
- Should describe the *change*, not the work done

### PR Description Template

```markdown
## Summary
<1-3 bullet points describing what this PR does>

## Test plan
- [ ] Unit tests pass
- [ ] Manual testing of <specific scenario>
- [ ] No regressions in <related area>

## Notes
<Any additional context, screenshots, or considerations>
```

### Creating PRs with GitHub CLI

```bash
gh pr create --title "Add user authentication" --body "$(cat <<'EOF'
## Summary
- Add JWT-based authentication
- Add login/logout endpoints
- Add auth middleware for protected routes

## Test plan
- [ ] Unit tests for auth module
- [ ] Manual test login flow
- [ ] Verify protected routes require auth

Generated with Claude Code
EOF
)"
```

## Safety Guidelines

### Never Commit

- `.env` files with real credentials
- API keys, tokens, passwords
- Private keys (`*.pem`, `*.key`)
- Large binary files (use Git LFS if needed)
- Build artifacts (`*.o`, `*.exe`, `node_modules/`)

### Dangerous Commands - Use with Caution

```bash
# These can lose work - understand before using:
git reset --hard    # Discards uncommitted changes
git push --force    # Overwrites remote history
git clean -f        # Deletes untracked files
git checkout .      # Discards uncommitted changes
git branch -D       # Force deletes branch
```

### Recovery

```bash
# Undo last commit (keep changes staged)
git reset --soft HEAD~1

# Undo last commit (keep changes unstaged)
git reset HEAD~1

# Recover deleted branch (if recent)
git reflog
git checkout -b recovered-branch <commit-hash>

# Recover discarded changes (if staged before)
git fsck --lost-found
```

## Repository Setup

### .gitignore Essentials

```gitignore
# Build artifacts
*.o
*.obj
*.exe
build/
bin/

# Dependencies
node_modules/
vendor/

# IDE/Editor
.vscode/
.idea/
*.swp

# Environment
.env
.env.local
*.pem
*.key

# OS
.DS_Store
Thumbs.db
```

### .gitattributes for Cross-Platform

```gitattributes
# Auto-detect text files and normalize line endings
* text=auto

# Force LF for scripts
*.sh text eol=lf

# Force CRLF for Windows batch files
*.bat text eol=crlf
*.cmd text eol=crlf

# Binary files
*.png binary
*.jpg binary
*.pdf binary
```
