# Remove Automatic Git Operations From Skills

## Goal

Modify all superpowers skills so that Claude never executes git commands (`git add`, `git commit`, `git push`, `gh pr create`, etc.) directly. Instead, Claude should prompt the user with a suggested commit message and list of changed files, letting the user handle all git operations manually.

## Motivation

The user wants full control over git operations. Claude should focus on writing code and providing guidance, not touching version control.

## Key Principle: Commit Prompts Are Non-Blocking

Commit prompts are **suggestions, not gates**. When Claude prompts the user to commit:
- The skill continues to the next step regardless of whether the user commits
- The user may commit immediately, later, or not at all
- Claude should never wait for or verify that a commit happened before proceeding
- Prompts are informational ("here's what changed, here's a suggested commit message") not blocking ("please commit before we continue")

## Pattern

For every git mutation command (`git add`, `git commit`, `git push`, `gh pr create`, `git merge`, `git branch -d/-D`, `git worktree remove`), replace the command block with a structured prompt listing changed files, suggested command(s), and a suggested commit message. Then continue the workflow.

Read-only git commands (`git status`, `git log`, `git merge-base`, `git worktree list`, `git branch --show-current`) remain unchanged — Claude can still read git state.

## Scope

4 skill files contain git operation instructions that need to be modified:

1. `skills/finishing-a-development-branch/SKILL.md`
2. `skills/writing-plans/SKILL.md`
3. `skills/brainstorming/SKILL.md`
4. `skills/subagent-driven-development/implementer-prompt.md`

## Changes

### 1. `skills/finishing-a-development-branch/SKILL.md`

All 4 options currently execute git commands directly. Replace each with user prompts:

**Option 1 (Merge locally):** Replace the `git checkout`, `git pull`, `git merge`, `git branch -d` commands with a prompt telling the user what commands to run. Claude still runs tests on the current branch as part of Step 1 — that doesn't change.

**Option 2 (Push and Create PR):** Replace `git push` and `gh pr create` with a prompt that includes:
- List of changed files
- Suggested commit message
- Suggested PR title and body
- The commands the user can copy-paste

**Option 3 (Keep as-is):** No change needed.

**Option 4 (Discard):** Replace `git checkout` and `git branch -D` with a prompt showing the commands. Still require typed "discard" confirmation before suggesting the commands.

**Step 5 (Cleanup Worktree):** Replace `git worktree remove` with a prompt showing the command. The read-only `git worktree list` command remains — Claude can still check worktree state.

**Quick Reference table:** Update to reflect that Claude prompts the user rather than performing operations directly:

| Option | Prompts Merge | Prompts Push | Keep Worktree | Prompts Branch Cleanup |
|--------|--------------|-------------|---------------|----------------------|
| 1. Merge locally | Yes | - | - | Yes |
| 2. Create PR | - | Yes | Yes | - |
| 3. Keep as-is | - | - | Yes | - |
| 4. Discard | - | - | - | Yes (force) |

**Output format for prompts:**
```
Here's what I suggest for committing and creating a PR:

**Files changed:**
- src/foo.py (new)
- src/bar.py (modified)

**Suggested commit message:**
feat: add widget parsing

**Commands to run:**
git add src/foo.py src/bar.py
git commit -m "feat: add widget parsing"
git push -u origin feature-branch
gh pr create --title "Add widget parsing" --body "..."
```

### 2. `skills/writing-plans/SKILL.md`

- **Line 10:** Change "Frequent commits" to "Prompt user for frequent commits"
- **Lines 98-103 (task template Step 5):** Replace the `git add` + `git commit` commands with an instruction to prompt the user with the list of changed files and a suggested commit message. Do NOT include git commands as steps for the agent to execute. Make clear this step is non-blocking — the workflow continues regardless.
- **Line 111:** Change "DRY, YAGNI, TDD, frequent commits" to "DRY, YAGNI, TDD, prompt user for frequent commits"

### 3. `skills/brainstorming/SKILL.md`

- **Line 29 (checklist step 6):** Change "save to `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` and commit" to "save to `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` and prompt the user to commit"
- **Line 117:** Change "Commit the design document to git" to "Prompt the user to commit the design document to git"
- **Line 129:** Change "Spec written and committed to `<path>`." to "Spec written and saved to `<path>`. You may want to commit it." (The old phrasing implies the commit already happened, which is no longer true.)

### 4. `skills/subagent-driven-development/implementer-prompt.md`

- **Line 35 (Your Job step 4):** Change "Commit your work" to "Prompt the user to commit your work (list changed files and suggest a commit message)"
- Note: This line is inside a template code fence (lines 5-113) that gets pasted into subagent dispatches. The change propagates to all future subagent invocations, which is the desired behavior.

## What Does NOT Change

- Claude still writes and edits code files normally
- Claude still runs tests
- Claude still reads git state (e.g., `git status`, `git log`, `git merge-base`, `git worktree list`) for informational purposes
- The overall skill workflows remain the same — only the git mutation steps change
- Skill workflows are never blocked waiting for the user to commit

## Testing

- After changes, verify each modified skill file reads correctly
- Manually test by invoking skills and confirming Claude prompts instead of executing git commands
- Verify that skills continue to the next step after prompting (no blocking on commit)
