---
name: commit-msg
description: Generate a commit message for uncommitted changes. Extracts JIRA ticket from branch name, validates branch context, follows repo commit style. Use when user wants a commit message written or suggested.
argument-hint: [optional extra context]
---

Generate a 1-sentence commit message for all uncommitted changes. Do NOT actually commit — only output the message.

## Steps

1. Run `git branch --show-current` and `git log --oneline -10` in parallel.
2. Extract the JIRA ticket number from the branch name (e.g. `BACK-15026`, `KAHOOT-71534`).
3. Sanity-check: do the recent commits on this branch make sense with your uncommitted changes? If the branch looks wrong (e.g. user forgot to create a new branch, or checked out the wrong PR), ask the user before proceeding.
4. If the ticket number is ambiguous or missing, ask the user for it.
5. Look at the last few commit messages to match the repository's style.
6. Write the commit message:
   - Starts with the JIRA ticket number.
   - One sentence, imperative mood.
   - Think one level higher in abstraction — describe the *why* or *what it enables*, not the list of files touched.
   - Do not overcomplicate it.

$ARGUMENTS
