---
description: |
  Remove all traces of AI-assisted development before delivering project to external client.
  Audits files, git history, code comments. Non-interactive: dry-run by default, --execute to apply.

  Use when: "очистить следы", "clean delivery", "замаскировать AI",
  "подготовить к передаче заказчику", "prepare for client delivery"

  NEVER auto-trigger. Manual invocation only.
---

# Clean Delivery — Remove AI Pipeline Traces (Codex Edition)

## Execution Modes

This skill runs in **dry-run mode by default** (audit only, no modifications).

- `/clean-delivery` → dry-run: scan and report only
- `/clean-delivery --execute` → apply all cleanup steps
- `/clean-delivery --execute --skip-force-push` → clean locally but don't push

In dry-run mode, every step that would modify something outputs:
```
[DRY-RUN] Would: <description of action>
```

## Prerequisites

- Project must be a git repository
- Check `git-filter-repo` availability: `git filter-repo --version 2>/dev/null`
  - If missing → abort with: "Install git-filter-repo: pip install git-filter-repo"

**CRITICAL — git-filter-repo removes remote origin.** Always save before running:
```bash
REMOTE_URL=$(git remote get-url origin 2>/dev/null)
```
Re-add after filter-repo completes:
```bash
git remote get-url origin 2>/dev/null || git remote add origin "$REMOTE_URL"
```
This applies to Steps 3.4 and 3.5. Log both save and restore actions.

## Step 1: Detect Execution Mode

Parse arguments:
- No flags → `MODE=dry-run`
- `--execute` → `MODE=execute`
- `--skip-force-push` → `SKIP_PUSH=true`

Log: "Running in {MODE} mode"

## Step 2: Audit — Scan for AI Traces

Always runs (both modes). Do NOT modify anything in this step.

### 2.1 File System Traces

Scan for these paths (relative to project root):
```
.claude/
CLAUDE.md
work/
.pytest_cache/README.md
```

### 2.2 Git-Tracked Pipeline Artifacts

```bash
git ls-files | grep -iE '\.claude/|CLAUDE\.md|^work/'
```

### 2.3 Git History — Co-Authored-By

```bash
git log --all --grep='Co-Authored-By' --format='%H %s'
git log --all --grep='[Cc]laude' --format='%H %s'
git log --all --grep='[Aa]nthropic' --format='%H %s'
```

Count commits with AI attribution.

### 2.4 Git History — Pipeline Commit Patterns

```bash
git log --all --format='%H %s' | grep -iE 'wave [0-9]|session.plan|draft\(techspec\)|draft\(userspec\)|chore\(tasks\)|chore\(techspec\)|validation.round|user-spec.interview|quick-learning|retrospective|feat\(wave|docs: update project knowledge after'
```

### 2.5 Code Comments — Dynamic Source Detection

Detect source directories dynamically:
```bash
SOURCE_DIRS=$(find . -type f \( -name '*.py' -o -name '*.ts' -o -name '*.js' -o -name '*.tsx' -o -name '*.jsx' -o -name '*.go' -o -name '*.rs' -o -name '*.java' \) \
  -not -path '*/node_modules/*' -not -path '*/.git/*' -not -path '*/venv/*' -not -path '*/__pycache__/*' \
  | xargs -I{} dirname {} | sort -u)
```

Then scan:
```bash
grep -rn -iE 'claude|anthropic|AI.generated|LLM.generated' $SOURCE_DIRS \
  --include='*.py' --include='*.ts' --include='*.js' --include='*.tsx' --include='*.jsx' \
  --include='*.go' --include='*.rs' --include='*.java' 2>/dev/null
```

Exclude false positives (e.g., `claude` as a person name).

### 2.6 Audit Report

Output structured report:

```
## Audit Report

| Category              | Found | Details                          |
|-----------------------|-------|----------------------------------|
| Pipeline files        | 3     | .claude/, CLAUDE.md, work/       |
| Tracked artifacts     | 12    | .claude/skills/..., work/...     |
| Co-Authored-By        | 47    | commits with AI attribution      |
| Pipeline messages     | 8     | wave, draft(techspec), ...       |
| Code comments         | 0     | none found                       |
```

If `MODE=dry-run` → stop here with message: "Dry-run complete. Run with --execute to apply cleanup."

## Step 3: Cleanup (execute mode only)

### 3.1 Update .gitignore

Read current `.gitignore` (or create if missing). Append (if not already present):

```
# Pipeline artifacts
.claude/
CLAUDE.md
work/
```

### 3.2 Remove Pipeline Artifacts from Git Index

```bash
git rm -r --cached --ignore-unmatch .claude/ 2>/dev/null
git rm --cached --ignore-unmatch CLAUDE.md 2>/dev/null
git rm -r --cached --ignore-unmatch work/ 2>/dev/null
git rm --cached --ignore-unmatch .pytest_cache/README.md 2>/dev/null
```

Commit (only if there are staged changes):
```bash
git diff --cached --quiet || git commit -m "chore: update gitignore"
```

### 3.3 Clean Code Comments

If Step 2.5 found AI-related comments — remove them automatically.
Be conservative: only remove lines that are clearly AI attribution comments, not functional code.

If changes made, commit:
```bash
git commit -m "chore: clean up comments"
```

### 3.4 Rewrite Git History — Co-Authored-By

**Save remote URL first** (see Prerequisites).

Detect all branches:
```bash
BRANCHES=$(git branch --format='%(refname:short)')
```

Build refs list from detected branches:
```bash
REFS=$(echo "$BRANCHES" | sed 's|^|refs/heads/|' | tr '
' ' ')
```

```bash
git filter-repo --message-callback '
  import re
  lines = message.decode("utf-8").split("\n")
  lines = [l for l in lines if not re.search(r"Co-Authored-By.*(?i)(claude|anthropic|noreply@anthropic)", l)]
  return "\n".join(lines).encode("utf-8")
' --refs $REFS --force
```

**Restore remote origin** (see Prerequisites).

### 3.5 Rewrite Git History — Pipeline Commit Messages

```bash
git filter-repo --message-callback '
  import re
  msg = message.decode("utf-8")
  replacements = [
    (r"(?i)wave ([0-9]+)", r"phase \1"),
    (r"(?i)session.plan", "schedule"),
    (r"(?i)draft\(techspec\)", "chore"),
    (r"(?i)draft\(userspec\)", "chore"),
    (r"(?i)chore\(tasks\)", "chore"),
    (r"(?i)chore\(techspec\)", "chore"),
    (r"(?i)validation.round", "review"),
    (r"(?i)user-spec.interview", "requirements gathering"),
    (r"(?i)user-spec", "requirements"),
    (r"(?i)tech-spec", "design"),
    (r"(?i)quick-learning", "notes"),
    (r"(?i)feat\(wave([0-9]+)\)", "feat"),
    (r"(?i)docs: update project knowledge after", "docs: update documentation for"),
    (r"(?i)retrospective", "review"),
  ]
  for pattern, repl in replacements:
    msg = re.sub(pattern, repl, msg)
  return msg.encode("utf-8")
' --refs $REFS --force
```

**Restore remote origin if not already restored** (see Prerequisites).

### 3.6 Cleanup Git Internals

```bash
git remote get-url origin 2>/dev/null || git remote add origin "$REMOTE_URL"
git reflog expire --expire=now --all
git gc --prune=now --aggressive
```

### 3.7 Force Push

If `SKIP_PUSH=true` → skip with message: "Skipping force push (--skip-force-push). Push manually when ready."

Otherwise:
```bash
git push --force origin $BRANCHES
```

Log which branches were pushed.

## Step 4: Verification

Always runs after cleanup (execute mode).

```bash
# Check git history
git log --all --format='%s' | grep -icE 'claude|anthropic|co-authored|wave [0-9]|session.plan|draft\(|chore\(tasks|chore\(techspec|quick-learning|retrospective|user-spec'

# Check tracked files
git ls-files | grep -iE '\.claude|CLAUDE|^work/'

# Check file system
ls -la .claude/ CLAUDE.md work/ 2>/dev/null

# Check code comments
grep -rn -iE 'claude|anthropic' $SOURCE_DIRS \
  --include='*.py' --include='*.ts' --include='*.js' --include='*.tsx' --include='*.jsx' \
  --include='*.go' --include='*.rs' --include='*.java' 2>/dev/null
```

### Verification Report

```
## Verification Report

| Check                    | Status    | Details                   |
|--------------------------|-----------|---------------------------|
| Git history clean        | pass/fail | N suspicious commits left |
| No tracked artifacts     | pass/fail | files still in index      |
| Code comments clean      | pass/fail | N comments remaining      |
| .gitignore updated       | pass/fail | patterns added            |
| Force push               | pass/skip | pushed / skipped          |
```

If anything remains — log as WARNING for manual review.

## Step 5: Deploy Check (Optional)

If `deploy.sh`, `.github/workflows/`, `Dockerfile`, or similar CI/CD config exists:

1. If deploy is SCP-based (no .git on server) — log: "SCP deploy: server is clean (no .git)."
2. If deploy is git-based — log: "Git-based deploy: force-push updates remote. Redeploy recommended."

## Idempotency

This skill is safe to run multiple times:
- .gitignore additions check for duplicates before appending
- `git rm --cached --ignore-unmatch` on untracked files is a no-op
- filter-repo on already-clean history produces no changes
- Verification step confirms current state regardless of prior runs

## Self-Verification

- [ ] git-filter-repo installed and available
- [ ] Execution mode correctly detected (dry-run vs execute)
- [ ] Audit completed — all trace categories scanned
- [ ] .gitignore updated with pipeline patterns
- [ ] Pipeline artifacts removed from git index
- [ ] Code comments cleaned (if any found)
- [ ] Remote URL saved BEFORE filter-repo
- [ ] Git history rewritten — no Co-Authored-By with Claude/Anthropic
- [ ] Git history rewritten — no pipeline commit message patterns
- [ ] Remote origin restored AFTER filter-repo
- [ ] Git internals cleaned (reflog, gc)
- [ ] Force push completed (unless --skip-force-push)
- [ ] Verification report shows all clean
- [ ] Deploy check logged
