---
name: commit-msg
description: Generate a commit message for uncommitted changes. Extracts JIRA ticket from branch name, validates branch context, follows repo commit style. Use when user wants a commit message written or suggested.
argument-hint: [optional extra context]
---

Generate a 1-sentence commit message for all uncommitted changes. Do NOT actually commit — only output the message.

**Investigate freely.** Don't guess from just the diff summary — read the actual changed files, grep for context, follow imports, check related tests. The more you understand *why* the change exists, the better the message. Git is your friend: use `git diff`, `git show`, `git log -p`, `git blame` as needed.

## Steps

1. Run `git branch --show-current`, `git log --oneline -10`, and `git diff HEAD` in parallel to get branch name, recent history, and the full diff at once.
2. Extract the JIRA ticket number from the branch name (e.g. `BACK-15026`, `KAHOOT-71534`).
3. Sanity-check: do the recent commits and code on this branch make sense with your uncommitted changes? If the branch looks wrong (e.g. user forgot to create a new branch, or checked out the wrong PR), ask the user before proceeding.
4. If the ticket number is ambiguous or missing, ask the user for it.
5. If the diff alone doesn't make the intent obvious, read the relevant source files or run `git log -p -- <file>` to understand the context before writing the message.
6. Look at the last few commit messages to match the repository's style.
7. Write the commit message:
   - Starts with the JIRA ticket number.
   - One sentence, imperative mood.
   - Think one level higher in abstraction — describe the *why* or *what it enables*, not the list of files touched.
   - Do not overcomplicate it.

$ARGUMENTS
