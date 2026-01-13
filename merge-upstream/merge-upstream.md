# Merge Upstream

Intelligently merge upstream changes (e.g., from Gary) into your current branch, squashing them into a single well-organized commit while resolving conflicts in your favor.

## Usage

Run `/merge-upstream` or `/merge-upstream <upstream-branch>` to merge changes from upstream into your current branch.

If no upstream branch is specified, defaults to `origin/main`.

## Process

### 1. Analyze the Current State

```bash
# Get current branch name
current_branch=$(git branch --show-current)

# Get upstream branch (default: origin/main)
upstream="${1:-origin/main}"

# Fetch latest changes
git fetch origin

# Find the merge base (common ancestor)
merge_base=$(git merge-base HEAD "$upstream")

# List commits from upstream that aren't in current branch
echo "Analyzing upstream commits..."
git log --oneline "$merge_base..$upstream"
```

### 2. Analyze and Categorize Commits

Extract all commit messages from upstream and categorize them:
- Identify reverts (commits that undo previous commits)
- Identify pairs of commits that cancel each other out
- Group related changes (features, fixes, refactors, etc.)
- Note any breaking changes or major refactors

### 3. Create Squashed Commit Message

Generate an intelligent commit message that:
- Has a clear, concise title (50 chars or less)
- Summarizes all meaningful changes in bullet points
- Omits commits that were reverted or canceled out
- Groups related changes together
- Uses conventional commit format when appropriate (feat:, fix:, refactor:, etc.)

Example format:
```
feat: merge upstream changes from Gary

- Add new terminal status tracking system
- Improve error handling in session manager
- Refactor configuration parsing
- Update dependencies (react@18.3.0, vite@5.1.0)
- Fix race condition in cleanup handler

Technical notes:
- Removed experimental feature X (unstable)
- Breaking: Changed SessionConfig interface
```

### 4. Create Temporary Branch and Squash

```bash
# Create a temporary branch at upstream
git branch temp-merge-upstream "$upstream"
git checkout temp-merge-upstream

# Squash all commits since merge base into one
git reset --soft "$merge_base"
git commit -m "<generated message from step 3>"
```

### 5. Merge into Current Branch

```bash
# Return to original branch
git checkout "$current_branch"

# Attempt merge
if git merge temp-merge-upstream --no-ff -m "Merge upstream changes"; then
  echo "✓ Merge successful with no conflicts"
else
  echo "⚠ Conflicts detected - resolving..."
  # Continue to conflict resolution
fi
```

### 6. Intelligent Conflict Resolution

When conflicts occur:

1. **List all conflicts:**
   ```bash
   git diff --name-only --diff-filter=U
   ```

2. **For each conflicted file:**
   - Read both versions (ours vs theirs)
   - Analyze the changes:
     - If changes are in different sections → keep both
     - If changes overlap → prefer ours (current branch) unless Gary's is clearly better
     - "Clearly better" means:
       - Fixes a bug that ours doesn't
       - Adds error handling or type safety
       - Uses more recent API or best practices
       - Includes tests or documentation
   - Ask user if uncertain about which version to keep

3. **Apply resolution strategy:**
   ```bash
   # For simple cases where we want ours:
   git checkout --ours <file>

   # For simple cases where we want theirs:
   git checkout --theirs <file>

   # For complex merges, manually edit and resolve
   ```

4. **Stage resolved files:**
   ```bash
   git add <resolved-file>
   ```

### 7. Verify Code Quality

After resolving all conflicts:

```bash
# Run type checking
if command -v bun &> /dev/null; then
  bun run typecheck || echo "⚠ Type errors detected"
fi

# Run linting
if [ -f "package.json" ]; then
  if bun run lint &> /dev/null; then
    echo "✓ Linting passed"
  else
    echo "⚠ Linting issues detected"
  fi
fi

# Run tests
if [ -f "package.json" ]; then
  if bun test &> /dev/null; then
    echo "✓ Tests passed"
  else
    echo "⚠ Tests failed - review changes"
  fi
fi
```

### 8. Complete Merge and Cleanup

```bash
# Complete the merge if there were conflicts
if [ -f ".git/MERGE_HEAD" ]; then
  git commit --no-edit
fi

# Delete temporary branch
git branch -D temp-merge-upstream

# Show summary
echo "✓ Merge complete!"
echo ""
echo "Summary:"
git log -1 --stat
```

## Conflict Resolution Guidelines

Default preference order:
1. **Ours (current branch)** - the default choice
2. **Theirs (upstream)** - only if clearly better based on:
   - Bug fixes
   - Security improvements
   - Better error handling
   - More complete type definitions
   - More comprehensive tests
   - Better documentation
3. **Manual merge** - when both have valuable changes

## Safety Checks

Before running:
- Ensure working directory is clean (no uncommitted changes)
- Confirm current branch is not main/master
- Verify tests pass before starting merge
- Create a backup branch: `git branch backup-$(git branch --show-current)-$(date +%Y%m%d-%H%M%S)`

## Recovery

If something goes wrong:
```bash
# Abort the merge
git merge --abort

# Or restore from backup
git checkout backup-<branch>-<timestamp>
```

## Notes

- This skill assumes you want to preserve your branch's history and add upstream changes as a single commit
- Conflicts are resolved conservatively (preferring your code) unless upstream is clearly superior
- All quality checks must pass before the merge is considered complete
- The skill will ask for confirmation on ambiguous conflict resolutions
