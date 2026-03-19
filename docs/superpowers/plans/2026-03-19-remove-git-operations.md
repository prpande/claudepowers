# Remove Git Operations Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Remove all git mutation commands from skills and replace them with non-blocking user prompts.

**Architecture:** Edit 4 markdown skill files to replace git mutation instructions with structured prompts. No code changes — purely skill document edits.

**Tech Stack:** Markdown

**Spec:** `docs/superpowers/specs/2026-03-19-remove-git-operations-design.md`

---

### Task 1: Update `finishing-a-development-branch/SKILL.md`

**Files:**
- Modify: `skills/finishing-a-development-branch/SKILL.md`

- [ ] **Step 1: Replace Option 1 (Merge Locally) git commands (lines 70-85)**

Replace:
```markdown
#### Option 1: Merge Locally

```bash
# Switch to base branch
git checkout <base-branch>

# Pull latest
git pull

# Merge feature branch
git merge <feature-branch>

# Verify tests on merged result
<test command>

# If tests pass
git branch -d <feature-branch>
```

Then: Cleanup worktree (Step 5)
```

With:
```markdown
#### Option 1: Merge Locally

Prompt the user with the commands to run:

```
To merge this branch locally, run:

git checkout <base-branch>
git pull
git merge <feature-branch>
git branch -d <feature-branch>
```

Then: Cleanup worktree (Step 5)
```

- [ ] **Step 2: Replace Option 2 (Push and Create PR) git commands (lines 89-106)**

Replace:
```markdown
#### Option 2: Push and Create PR

```bash
# Push branch
git push -u origin <feature-branch>

# Create PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>
EOF
)"
```

Then: Cleanup worktree (Step 5)
```

With:
```markdown
#### Option 2: Push and Create PR

Prompt the user with the changed files, a suggested commit message, and the commands to run:

```
Here's what I suggest for pushing and creating a PR:

**Files changed:**
- <list each changed file and whether new/modified>

**Suggested commit message:**
<conventional commit message>

**Commands to run:**
git add <files>
git commit -m "<message>"
git push -u origin <feature-branch>
gh pr create --title "<title>" --body "## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>"
```

Then: Cleanup worktree (Step 5)
```

- [ ] **Step 3: Replace Option 4 (Discard) git commands (lines 128-133)**

Replace:
```markdown
If confirmed:
```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

Then: Cleanup worktree (Step 5)
```

With:
```markdown
If confirmed, prompt the user with the commands to run:

```
To discard this branch, run:

git checkout <base-branch>
git branch -D <feature-branch>
```

Then: Cleanup worktree (Step 5)
```

- [ ] **Step 4: Replace Step 5 worktree cleanup git command (lines 140-148)**

Keep the read-only `git worktree list` check. Replace only the `git worktree remove` command.

Replace:
```markdown
If yes:
```bash
git worktree remove <worktree-path>
```
```

With:
```markdown
If yes, prompt the user:

```
To clean up the worktree, run:

git worktree remove <worktree-path>
```
```

- [ ] **Step 5: Update Quick Reference table (lines 153-159)**

Replace:
```markdown
| Option | Merge | Push | Keep Worktree | Cleanup Branch |
|--------|-------|------|---------------|----------------|
| 1. Merge locally | ✓ | - | - | ✓ |
| 2. Create PR | - | ✓ | ✓ | - |
| 3. Keep as-is | - | - | ✓ | - |
| 4. Discard | - | - | - | ✓ (force) |
```

With:
```markdown
| Option | Prompts Merge | Prompts Push | Keep Worktree | Prompts Branch Cleanup |
|--------|--------------|-------------|---------------|----------------------|
| 1. Merge locally | Yes | - | - | Yes |
| 2. Create PR | - | Yes | Yes | - |
| 3. Keep as-is | - | - | Yes | - |
| 4. Discard | - | - | - | Yes (force) |
```

- [ ] **Step 6: Verify the file reads correctly**

Read the full file and confirm all changes are consistent and no git mutation commands remain.

---

### Task 2: Update `writing-plans/SKILL.md`

**Files:**
- Modify: `skills/writing-plans/SKILL.md`

- [ ] **Step 1: Update line 10**

Replace:
```
Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.
```

With:
```
Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Prompt user for frequent commits.
```

- [ ] **Step 2: Replace task template Step 5 (lines 98-103)**

Replace:
````markdown
- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

With:
````markdown
- [ ] **Step 5: Prompt user to commit**

Prompt the user with the list of changed files and a suggested commit message. Do NOT run git commands. This step is non-blocking — continue to the next task regardless of whether the user commits.
````

- [ ] **Step 3: Update line 111**

Replace:
```
- DRY, YAGNI, TDD, frequent commits
```

With:
```
- DRY, YAGNI, TDD, prompt user for frequent commits
```

- [ ] **Step 4: Verify the file reads correctly**

Read the full file and confirm all changes are consistent.

---

### Task 3: Update `brainstorming/SKILL.md`

**Files:**
- Modify: `skills/brainstorming/SKILL.md`

- [ ] **Step 1: Update checklist step 6 (line 29)**

Replace:
```
6. **Write design doc** — save to `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` and commit
```

With:
```
6. **Write design doc** — save to `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` and prompt the user to commit
```

- [ ] **Step 2: Update documentation section (line 117)**

Replace:
```
- Commit the design document to git
```

With:
```
- Prompt the user to commit the design document to git
```

- [ ] **Step 3: Update User Review Gate prompt (line 129)**

Replace:
```
> "Spec written and committed to `<path>`. Please review it and let me know if you want to make any changes before we start writing out the implementation plan."
```

With:
```
> "Spec written and saved to `<path>`. You may want to commit it. Please review it and let me know if you want to make any changes before we start writing out the implementation plan."
```

- [ ] **Step 4: Verify the file reads correctly**

Read the full file and confirm all changes are consistent.

---

### Task 4: Update `subagent-driven-development/implementer-prompt.md`

**Files:**
- Modify: `skills/subagent-driven-development/implementer-prompt.md`

- [ ] **Step 1: Update "Your Job" step 4 (line 35)**

This line is inside a template code fence (lines 5-113). Replace:
```
    4. Commit your work
```

With:
```
    4. Prompt the user to commit your work (list changed files and suggest a commit message)
```

- [ ] **Step 2: Verify the file reads correctly**

Read the full file and confirm the change is correct and the template structure is preserved.
