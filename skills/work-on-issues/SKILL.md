---
name: work-on-issues
description: >
  Fetch issues from GitHub or GitLab, work on them systematically, and move completed issues to closed.
  Use when the user says "work on issues", "fetch issues", "pick up an issue", "start working on",
  or wants to triage and implement tickets from the issue tracker.
---

# Work on Issues

Fetch issues from the configured tracker (GitHub or GitLab), pick up work, implement it, and close completed issues.

## Detecting the Tracker

Determine the tracker at session start by checking remotes:

```bash
git remote -v
```

| Remote host | Tracker | CLI | Command prefix |
|-------------|---------|-----|----------------|
| `github.com` | GitHub Issues | `gh` | `gh issue ...` |
| `gitlab.com` | GitLab Issues | `glab` | `glab issue ...` |

Store the resolved CLI alias (`gh` or `glab`) as **`$TRACKER`** for the rest of this skill. All commands below use `$TRACKER`.

### GitLab terminology note

GitLab uses different nouns than GitHub:
- **issues** → same concept, CLI is `glab issue`
- **merge requests (MRs)** = GitHub's **pull requests (PRs)** → `glab mr` instead of `gh pr`
- **notes** = GitHub's **comments** → `glab issue note` / `glab mr note` instead of `gh issue comment` / `gh pr comment`

When commands are structurally identical, use `$TRACKER` as a shortcut. When the subcommand differs (`note` vs `comment`, `mr` vs `pr`), use the platform-specific form.

## Workflow

### Phase 1: Fetch & Triage

1. **List open issues** — fetch in machine-readable format:

   ```bash
   $TRACKER issue list --state open -F json
   ```

   For GitLab with label filtering:
   ```bash
   glab issue list --state open --label "bug" -F json
   ```

2. **Parse & present** — summarize each issue: number, title, labels, brief description. Present the list to the user.

3. **Let the user pick** — ask which issue(s) to work on, or suggest based on labels/priority.

4. **Read the full issue** — load details and comments:

   ```bash
   # GitHub
   gh issue view <number> --comments
   # GitLab
   glab issue view <number> --comments
   ```

   Use `-F json` for machine-readable output when parsing is needed.

5. **Assign / label** — mark the issue as in-progress if the tracker supports it:

   ```bash
   # GitHub
   gh issue edit <number> --add-label "in-progress"
   # GitLab
   glab issue update <number> --label "in-progress"
   ```

### Phase 2: Implement

6. **Create a branch** — use the issue number in the branch name:

   ```bash
   git checkout -b work-on-issue-<number>
   ```

7. **Implement the work** — follow the issue description, acceptance criteria, and any linked specs.

8. **Verify** — run tests, lint, build. Use [verification-before-completion](../verification-before-completion/SKILL.md) if available.

9. **Commit** — reference the issue in the commit message:

   ```bash
   git commit -m "fix: resolve #<number> — <description>"
   ```

### Phase 3: Submit & Close

10. **Create a PR/MR**:

    ```bash
    # GitHub
    gh pr create --title "Fix #<number>: <title>" --body "Closes #<number>"
    # GitLab
    glab mr create --title "Fix #<number>: <title>" --description "Closes #<number>"
    ```

11. **Post a summary comment** on the issue:

    ```bash
    # GitHub
    gh issue comment <number> --body "Implementation complete. PR: <url>"
    # GitLab
    glab issue note <number> --message "Implementation complete. MR: <url>"
    ```

12. **After merge** — close the issue (if not auto-closed by the PR/MR):

    For GitLab, `glab issue close` does not accept a closing comment, so post first then close:
    ```bash
    # GitLab — post explanation then close
    glab issue note <number> --message "Resolved in MR <number>"
    glab issue close <number>

    # GitHub
    gh issue close <number> --comment "Resolved in PR <number>"
    ```

13. **Clean up branch**:

    ```bash
    git branch -d work-on-issue-<number>
    ```

## Batch Mode

When the user wants to work through multiple issues:

1. Fetch the full open list (Phase 1).
2. For each issue, run Phases 2–3 sequentially.
3. Between issues, check with the user before proceeding to the next.
4. Maintain a summary table:

   | Issue | Title | Status | PR/MR |
   |-------|-------|--------|-------|
   | #12 | Fix login bug | ✅ Closed | !34 |
   | #15 | Add export | 🔄 In Progress | !36 |
   | #18 | Update docs | ⏳ Pending | — |

## Label Conventions

| Label | Meaning | Default Color |
|-------|---------|---------------|
| `in-progress` | Currently being worked on | `#E4E669` |
| `ready-for-review` | Implementation done, awaiting merge | `#0E8A16` |
| `blocked` | Cannot proceed (needs info, dependency, etc.) | `#D93F0B` |

Apply/remove as the issue moves through states.

### Auto-create labels

If a label does not exist, create it before applying:

```bash
# GitHub
gh label create "in-progress" --color "#E4E669" --description "Currently being worked on"
# GitLab
glab label create --name "in-progress" --color "#E4E669" --description "Currently being worked on"
```

Use the same pattern for `ready-for-review` (`#0E8A16`) and `blocked` (`#D93F0B`).

**Shortcut:** wrap label application in a helper — try to apply, and if the CLI reports "not found", create it first then retry:

```bash
# GitHub helper pattern
gh issue edit <number> --add-label "in-progress" || \
  (gh label create "in-progress" --color "#E4E669" --description "Currently being worked on" && \
   gh issue edit <number> --add-label "in-progress")

# GitLab helper pattern
glab issue update <number> --label "in-progress" || \
  (glab label create --name "in-progress" --color "#E4E669" --description "Currently being worked on" && \
   glab issue update <number> --label "in-progress")
```

## Edge Cases

- **No tracker CLI installed** — fall back to API calls or ask the user to install `gh`/`glab`.
- **Multiple remotes** — let the user specify which tracker to use.
- **Issue already assigned** — skip the assignment step.
- **Issue needs clarification** — comment/note on the issue asking for details, then pause implementation.
- **Partially done** — if an issue is too large, break it into sub-issues or a checklist and track progress in a comment/note.
