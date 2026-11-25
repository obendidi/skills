---
name: pr
description: Open, create, comment on, reply to review threads, or babysit a pull request through CI and review. Use when the user asks to file, create, open, comment on, reply to, check status of, watch CI on, or babysit a PR.
---

# Pull Requests

In T3 Code, use the `pull_request_*`, `ci_*`, and `watch_pull_request` tools; elsewhere use `gh` (or the GitHub REST API if GraphQL is rate-limited).

## 1. Filing

- Check if a PR already exists for the branch; inspect the local diff against the base branch to verify it matches the goal.
- Follow repo title conventions (check recent merged PRs). Lead with the user-visible outcome, not implementation mechanics.
- Open the description with the problem in plain words, then briefly explain the fix — never open with a file-by-file inventory.
- Check title, body, quotes, and attachments for secrets or private details.
- Open a non-draft PR by default (unless a draft was requested) so review bots and CI run. Stop there unless asked to babysit.

## 2. Commenting on GitHub

Prefix every agent-authored PR comment, review reply, or dismissal explanation with:

```md
> [!NOTE]
> 🤖 **<verified model name> responding on behalf of <user>**

Comment body here.
```

- Never add this prefix to PR descriptions, commit messages, or text the user asked to post verbatim.
- Use the verified public model name (never guess) and resolve `<user>` from `git config user.name` or the repo's authenticated GitHub handle (`gh api user -q .login`); check for secrets before posting.
- Lead with the conclusion, cite code/check evidence, skip filler, and verify GitHub returned the comment URL before resolving a thread.

## 3. Babysitting

- Handle existing checks and unresolved comments first. In T3 Code, start `watch_pull_request` and end your turn (`unwatch_pull_request` before handing back or asking a question). Elsewhere, wait on pending CI with `gh pr checks --watch` (never a manual `sleep` loop) and re-read comments via `gh pr view`.
- Scope CI checks to the current `HEAD` SHA, and evaluate all **unresolved** review threads against current `HEAD` regardless of when they were posted. Verify every bot finding against the source before editing.
- **Fight scope creep**: fix real bugs in the intended case and real CI failures (distinguish infra flakes). Decline nits, speculative guards, or extra tests with a brief written reason — auto-resolve advisory bot threads only, and leave human disagreements open for the reviewer. Prefer fixes that delete or narrow code.
- Treat advisory bots (e.g. CodeRabbit) as feedback only — never hold a merge for them.
- Check whether the PR is `BEHIND` or `DIRTY` before calling it ready. Require a clean working tree; prefer `git merge origin/<base>`, or rebase + `git push --force-with-lease` (never bare `--force`) only on agent-owned branches (abort and ask on conflicts or shared branches), then re-verify if base commits touched the PR's files.
- Merge only when explicitly asked; otherwise stop and report readiness once required checks and reviews pass.
