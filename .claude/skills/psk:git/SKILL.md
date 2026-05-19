---
name: psk:git
description: Git workflow shortcuts for committing, pushing, creating PRs, and merging. Use for quick commit+push (cp), PR creation (pr), or merge+cleanup (merge).
disable-model-invocation: true
argument-hint: "[cp|pr|merge]"
---

# Git Workflow

Streamline common git operations while enforcing the project's branching rules (never commit on main, always use feature branches, never force push).

## Commands

Determine the command from the user's input. If no subcommand is provided, ask which operation they want using `AskUserQuestion`.

| Command | Usage | Description |
|---------|-------|-------------|
| **cp** | `/git cp` | Commit all changes and push to remote |
| **pr** | `/git pr` | Create a PR from current branch to main |
| **merge** | `/git merge` | Merge current branch's PR, checkout main, pull, delete branch |

---

## Command: cp (commit & push)

Commit staged/unstaged changes and push to the remote branch.

### Workflow

1. **Check current branch**
   ```bash
   git branch --show-current
   ```
   If on `main`, you MUST ask the user for a branch name using `AskUserQuestion`, then create and switch to it:
   ```bash
   git checkout -b <branch-name>
   ```

2. **Understand the changes**
   Run these in parallel:
   ```bash
   git status
   git diff
   git diff --cached
   ```

3. **Stage relevant files**
   - Stage files related to the work. Use specific file paths, not `git add -A` or `git add .`
   - NEVER stage files like `.DS_Store`, `.claude/session-id`, or other non-project files
   - If there are no changes to commit, inform the user and stop

4. **Draft a commit message**
   - Analyze the staged changes to write a concise, descriptive commit message
   - Use conventional commit format: `<type>(<scope>): <description>` (e.g., `feat(auth): add JWT validation`)
   - Common types: `feat`, `fix`, `docs`, `refactor`, `chore`, `test`
   - Focus on the "why" rather than the "what"
   - Also check `git log --oneline -10` and follow any existing commit style if it differs

5. **Commit**
   ```bash
   git commit -m "$(cat <<'EOF'
   <commit message>

   Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
   EOF
   )"
   ```

6. **Push**
   ```bash
   git push -u origin <current-branch>
   ```
   If push fails, report the error to the user. Never use `--force`.

7. **Report result** — show the commit hash and pushed branch name

---

## Command: pr (create pull request)

Create a GitHub pull request from the current branch to main.

### Workflow

1. **Verify not on main**
   ```bash
   git branch --show-current
   ```
   If on `main`, abort and tell the user to switch to a feature branch first (or use `/git cp` to create one).

2. **Check for uncommitted changes**
   ```bash
   git status --porcelain
   ```
   If there are uncommitted changes, warn the user and suggest running `/git cp` first. Do not proceed until resolved.

3. **Check for existing PR**
   ```bash
   gh pr view --json url,state 2>/dev/null
   ```
   If a PR already exists for this branch, show its URL and stop. Do not create a duplicate.

4. **Gather context** (run in parallel)
   ```bash
   git diff main...HEAD
   git log main..HEAD --oneline
   ```
   If there are no commits on this branch beyond main, inform the user there is nothing to create a PR for and stop.

5. **Push branch to remote**
   ```bash
   git push -u origin <current-branch>
   ```

6. **Create the PR**
   - If the user provided a title or description as an argument, use that
   - Otherwise generate a concise title (under 70 characters) from the branch commits
   - Generate a body with a summary section and test plan
   ```bash
   gh pr create --base main --title "<title>" --body "$(cat <<'EOF'
   ## Summary
   <1-3 bullet points summarizing the changes>

   ## Test plan
   <bulleted checklist of testing steps>

   Generated with [Claude Code](https://claude.com/claude-code)
   EOF
   )"
   ```
   If the user requests a draft PR, add the `--draft` flag.
   If the user specifies reviewers, add `--reviewer <users>`.

7. **Report result** — show the PR URL

---

## Command: merge (merge PR & cleanup)

Merge the current branch's PR, return to main, and clean up.

### Workflow

1. **Detect current branch**
   ```bash
   git branch --show-current
   ```
   If on `main`, abort and tell the user they're already on main.

2. **Find and verify the PR**
   ```bash
   gh pr view --json number,title,state,mergeable
   ```
   - If no PR exists, inform the user and suggest running `/git pr` first
   - If the PR is not open (already merged or closed), inform the user and stop
   - If `mergeable` is `CONFLICTING`, tell the user to resolve merge conflicts first
   - If `mergeable` is `UNKNOWN`, suggest waiting a moment and retrying

3. **Confirm with the user**
   Use `AskUserQuestion` to confirm the merge, showing the PR title and number.

4. **Merge the PR**
   ```bash
   gh pr merge --merge --delete-branch
   ```
   If the merge fails:
   - **Required reviews**: tell the user how many approvals are still needed
   - **Failing checks**: tell the user which checks failed
   - **Any other error**: report the raw error message and stop

5. **Return to main and pull**
   ```bash
   git checkout main && git pull
   ```

6. **Delete the local feature branch** (if not already deleted)
   ```bash
   git branch -d <branch-name>
   ```

7. **Report result** — confirm the PR was merged, branch deleted, and main is up to date

---

## Safety Rules

These rules are **non-negotiable** and align with the project's CLAUDE.md:

- **Never commit on main** — always create a feature branch first
- **Never push to main** — only push feature branches
- **Never force push** — if push is rejected, report the issue to the user
- **Never use `git add -A` or `git add .`** — always stage specific files
- **Never stage secrets** — skip `.env`, credentials, keys, etc.
- **Never skip hooks** — do not use `--no-verify`

---

## Tools to Use

- **Bash**: Run git and gh CLI commands
- **AskUserQuestion**: Get branch names, confirm merges, clarify intent