---
description: Review staged/unstaged changes, a PR, or branch diff with the reviewer agent
argument-hint: "[branch|PR-URL]"
---

Use the subagent tool with the "reviewer" agent to perform a code review.

User input: $@

If no input was provided:
1. Run `git status` to check for staged/unstaged changes
2. If changes exist, run `git diff` (unstaged) and `git diff --cached` (staged) to get all local changes, then review them
3. If the working tree is clean, review the current branch's changes compared to the default branch:
   - Detect the default branch: `default_branch=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's|refs/remotes/origin/||' || (git show-ref --verify --quiet refs/remotes/origin/main && echo "main" || echo "master"))`
   - Get the merge base: `merge_base=$(git merge-base HEAD "$default_branch")`
   - Run `git diff "$merge_base"...HEAD` to get the full branch diff
   - Run `git log --oneline "$merge_base"..HEAD` to list commits
   - Read all modified files and provide a complete structured review

If a branch name was provided: review `git diff "$@"...HEAD` (or `git diff "$@"..HEAD` if not a remote branch)

If a PR URL was provided:
- Try `gh pr diff <PR-number>` if the GitHub CLI is available
- Otherwise try `curl -sL "<PR-URL>.diff"` or `curl -sL "<PR-URL>/files.diff"`
- Review the fetched diff

The reviewer agent should read all modified files and provide a structured review with:
- Files Reviewed
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (consider)
- Summary
