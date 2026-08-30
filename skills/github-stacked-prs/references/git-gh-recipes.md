# Git and GitHub CLI Recipes

Use these recipes only after reading the repository's instructions and adapting branch names, remotes, merge method, and checks. Placeholders such as `<trunk>` are not literal shell input.

## Contents

- Inspect the repository and current stack.
- Create a stack from new work or an existing cumulative branch.
- Add a leaf, insert a middle layer, fold or remove a layer, or reorder layers.
- Apply review feedback, amend a parent, or sync the stack with the integration branch.
- Resolve rebase conflicts and recover from a failed restack.
- Land a layer with merge-commit, squash, or rebase-style ancestry.
- Repair PR bases, change draft state, and clean up merged branches.

Most concrete examples use this stack:

```text
main
  `-- feature/accounts-schema    PR #101 -> main
       `-- feature/accounts-api  PR #102 -> feature/accounts-schema
            `-- feature/accounts-ui  PR #103 -> feature/accounts-api
```

Substitute the repository's actual branch names and PR numbers. Commands that push, edit, close, or merge PRs require authorization for that remote mutation.

## Inspect The Repository And Existing Stack

```bash
git status --short --branch
git remote -v
git branch -vv
git log --graph --decorate --oneline --all --max-count=80
gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'
gh pr list --state open --json number,title,headRefName,baseRefName,isDraft,url
```

For one branch or PR:

```bash
git log --oneline <parent>..<child>
git diff --stat <parent>...<child>
git diff <parent>...<child>
gh pr view <number-or-branch> --json number,title,headRefName,baseRefName,isDraft,state,url
```

GitHub pull requests use a three-dot comparison, so `git diff <base>...<head>` is the useful local approximation. Confirm ancestry separately:

```bash
git merge-base --is-ancestor <parent> <child>
```

Exit status `0` means the relationship holds; status `1` means it does not.

## Create A New Stack

Example topology:

```text
origin/<trunk>
  `-- <topic>-01-foundation       PR 1 -> <trunk>
       `-- <topic>-02-api         PR 2 -> <topic>-01-foundation
            `-- <topic>-03-ui     PR 3 -> <topic>-02-api
```

Create and validate branches from root to leaf:

```bash
git fetch origin
git switch --create <topic>-01-foundation origin/<trunk>
# Implement or isolate the first slice, validate it, and commit it.

git switch --create <topic>-02-api
# Implement or isolate the second slice, validate it, and commit it.

git switch --create <topic>-03-ui
# Implement or isolate the third slice, validate it, and commit it.
```

Push only after checking each unique commit range and diff:

```bash
git push --set-upstream origin <topic>-01-foundation
git push --set-upstream origin <topic>-02-api
git push --set-upstream origin <topic>-03-ui
```

Create PRs with explicit bases:

```bash
gh pr create --head <topic>-01-foundation --base <trunk> --title '<title>' --body-file <body-file>
gh pr create --head <topic>-02-api --base <topic>-01-foundation --title '<title>' --body-file <body-file>
gh pr create --head <topic>-03-ui --base <topic>-02-api --title '<title>' --body-file <body-file>
```

Add `--draft` when the PR should not yet request review. Do not put secrets, internal tokens, or unredacted command output in the body file.

## Create The Concrete Three-PR Example

The following shows the full branch and PR sequence. Replace the sample paths and validation commands with the repository's real ones; do not stage unrelated changes.

```bash
git fetch origin

git switch --create feature/accounts-schema origin/main
# Edit only the schema or migration slice.
git add --patch
git commit -m 'feat: add account schema'
make test-schema

git switch --create feature/accounts-api
# Edit only the API slice.
git add --patch
git commit -m 'feat: add accounts API'
make test-api

git switch --create feature/accounts-ui
# Edit only the UI slice.
git add --patch
git commit -m 'feat: add accounts UI'
make test-ui
```

Verify the topology and unique diffs before pushing:

```bash
git merge-base --is-ancestor origin/main feature/accounts-schema
git merge-base --is-ancestor feature/accounts-schema feature/accounts-api
git merge-base --is-ancestor feature/accounts-api feature/accounts-ui

git log --oneline origin/main..feature/accounts-schema
git log --oneline feature/accounts-schema..feature/accounts-api
git log --oneline feature/accounts-api..feature/accounts-ui

git diff --stat origin/main...feature/accounts-schema
git diff --stat feature/accounts-schema...feature/accounts-api
git diff --stat feature/accounts-api...feature/accounts-ui
```

Publish root to leaf and create explicit PR bases:

```bash
git push --set-upstream origin feature/accounts-schema
git push --set-upstream origin feature/accounts-api
git push --set-upstream origin feature/accounts-ui

gh pr create \
  --base main \
  --head feature/accounts-schema \
  --title 'Add account schema' \
  --body $'Stack: 1 of 3\n\nUnique scope: account schema and migration.\n\nChild: feature/accounts-api'

gh pr create \
  --base feature/accounts-schema \
  --head feature/accounts-api \
  --title 'Add accounts API' \
  --body $'Stack: 2 of 3\n\nParent: feature/accounts-schema\nChild: feature/accounts-ui\n\nUnique scope: accounts API.'

gh pr create \
  --base feature/accounts-api \
  --head feature/accounts-ui \
  --title 'Add accounts UI' \
  --body $'Stack: 3 of 3\n\nParent: feature/accounts-api\n\nUnique scope: accounts UI.'

gh pr list \
  --state open \
  --json number,title,headRefName,baseRefName,isDraft,url
```

## PR Body Shape

Use the repository's PR template when present. Add stack navigation without duplicating the entire feature description:

```markdown
## Stack

2 of 3

- Parent: #<parent-pr>
- This PR: <short unique purpose>
- Child: #<child-pr>

Merge order: #<root-pr> -> #<this-pr> -> #<leaf-pr>

## Unique Scope

<What this PR introduces beyond its parent.>

## Validation

- `<command>`
- <manual or CI check>

## Review Notes

Review this PR against `<parent-branch>`, not against `<trunk>`.
```

After all PRs exist, edit descriptions to replace provisional branch references with PR links when useful.

## Add A New Leaf PR

To add an audit-log layer above the UI PR:

```bash
git switch feature/accounts-ui
git switch --create feature/accounts-audit-log
# Implement only the audit-log slice.
git add --patch
git commit -m 'feat: add account audit log'
make test-audit-log

git diff --stat feature/accounts-ui...feature/accounts-audit-log
git push --set-upstream origin feature/accounts-audit-log

gh pr create \
  --base feature/accounts-ui \
  --head feature/accounts-audit-log \
  --title 'Add account audit log' \
  --body $'Stack: 4 of 4\n\nParent: #103\n\nUnique scope: audit-log behavior.'
```

## Split An Existing Cumulative Branch

First preserve the existing tip with a clearly named backup branch. Inspect the commits and identify the last commit for each intended slice. If the commits already form the desired order, branches can point at those boundary commits without rewriting them:

```bash
git branch backup/<topic>-before-stack <existing-branch>
git branch <topic>-01-foundation <first-boundary-sha>
git branch <topic>-02-api <second-boundary-sha>
git branch <topic>-03-ui <existing-branch>
```

If commits mix concerns, prefer building fresh root-to-leaf branches from the integration branch and cherry-picking or recreating only the changes owned by each slice. Use interactive rebase only when rewriting the existing branch is authorized and the commit boundaries can be verified. Never delete the backup ref until the published stack and diffs are confirmed.

## Convert Ordered Commits Into Branches

Suppose `feature/accounts-complete` contains three already-ordered commits and the commit SHAs are `a111111`, `b222222`, and `c333333` from root to leaf:

```bash
git branch backup/accounts-complete feature/accounts-complete
git branch feature/accounts-schema a111111
git branch feature/accounts-api b222222
git branch feature/accounts-ui c333333

git merge-base --is-ancestor feature/accounts-schema feature/accounts-api
git merge-base --is-ancestor feature/accounts-api feature/accounts-ui
git diff --stat origin/main...feature/accounts-schema
git diff --stat feature/accounts-schema...feature/accounts-api
git diff --stat feature/accounts-api...feature/accounts-ui
```

If one commit mixes multiple slices, rebuild fresh branches and use `git cherry-pick --no-commit <sha>` followed by `git add --patch` and a focused commit on each appropriate branch. Keep the backup branch until the published stack is verified.

## Insert A New Middle Layer

This inserts `feature/accounts-validation` between the schema and API branches. It rewrites the API and UI branches, so save their old tips first:

```bash
git fetch origin
git branch backup/accounts-api-before-validation feature/accounts-api
git branch backup/accounts-ui-before-validation feature/accounts-ui
old_api=$(git rev-parse feature/accounts-api)

git switch feature/accounts-schema
git switch --create feature/accounts-validation
# Implement the validation slice.
git add --patch
git commit -m 'feat: validate account records'
make test-validation

git switch feature/accounts-api
git rebase --onto feature/accounts-validation feature/accounts-schema
new_api=$(git rev-parse feature/accounts-api)

git switch feature/accounts-ui
git rebase --onto "$new_api" "$old_api"

git diff --stat feature/accounts-schema...feature/accounts-validation
git diff --stat feature/accounts-validation...feature/accounts-api
git diff --stat feature/accounts-api...feature/accounts-ui
```

Publish the new and rewritten branches, then change the API PR base:

```bash
git push --set-upstream origin feature/accounts-validation
git push --force-with-lease origin feature/accounts-api
git push --force-with-lease origin feature/accounts-ui

gh pr create \
  --base feature/accounts-schema \
  --head feature/accounts-validation \
  --title 'Validate account records' \
  --body $'Inserted between #101 and #102.\n\nUnique scope: account validation.'
gh pr edit 102 --base feature/accounts-validation
gh pr view 102 --json number,headRefName,baseRefName,url
```

## Fold A Middle PR Into Its Child

If the validation layer no longer deserves a separate PR but its changes should remain, make the API PR include both validation and API changes. No history rewrite is needed because the API branch already contains the validation commits:

```bash
gh pr edit 102 --base feature/accounts-schema
gh pr diff 102 --name-only
gh pr close 104 --comment 'Folded this validation layer into #102.'
```

Close PR `#104` only after confirming PR `#102` now contains exactly the combined scope. Delete the validation branch only when explicitly authorized and no open PR still uses it as a base.

## Remove A Middle Layer And Drop Its Changes

To remove the validation commits rather than fold them into the API PR, replay only the API commits onto the schema branch, then restack the UI:

```bash
git branch backup/accounts-api-before-dropping-validation feature/accounts-api
git branch backup/accounts-ui-before-dropping-validation feature/accounts-ui
old_api=$(git rev-parse feature/accounts-api)

git switch feature/accounts-api
git rebase --onto feature/accounts-schema feature/accounts-validation
new_api=$(git rev-parse feature/accounts-api)

git switch feature/accounts-ui
git rebase --onto "$new_api" "$old_api"

git diff --stat feature/accounts-schema...feature/accounts-api
git diff --stat feature/accounts-api...feature/accounts-ui
git push --force-with-lease origin feature/accounts-api
git push --force-with-lease origin feature/accounts-ui

gh pr edit 102 --base feature/accounts-schema
gh pr close 104 --comment 'Removed this layer from the stack.'
```

## Reorder Two Adjacent Layers

This changes `schema -> API -> UI` into `schema -> UI -> API`. Do this only if the UI slice does not require the API slice and both intermediate states remain valid:

```bash
git branch backup/accounts-api-before-reorder feature/accounts-api
git branch backup/accounts-ui-before-reorder feature/accounts-ui
old_api=$(git rev-parse feature/accounts-api)

git switch feature/accounts-ui
git rebase --onto feature/accounts-schema "$old_api"
new_ui=$(git rev-parse feature/accounts-ui)

git switch feature/accounts-api
git rebase --onto "$new_ui" feature/accounts-schema

git diff --stat feature/accounts-schema...feature/accounts-ui
git diff --stat feature/accounts-ui...feature/accounts-api
git push --force-with-lease origin feature/accounts-ui
git push --force-with-lease origin feature/accounts-api

gh pr edit 103 --base feature/accounts-schema
gh pr edit 102 --base feature/accounts-ui
```

Restack any branches that were above the old UI tip using their saved old and new parent tips.

## Restack After A Parent Changes

Before rewriting descendants, record the old tip of every affected branch with backup refs. Suppose `<parent>` changed from `<old-parent-tip>` to `<new-parent-tip>`:

```bash
git switch <child>
git rebase --onto <new-parent-tip> <old-parent-tip>
git diff <new-parent-tip>...<child>
git push --force-with-lease origin <child>
```

For a grandchild, use the child's old tip and newly rebased tip as the old and new parent tips. Continue one edge at a time toward the leaf. Do not calculate later rebases from a branch whose old tip was not recorded.

If the parent only gained commits and its old history is still intact, this simpler form may be sufficient:

```bash
git switch <child>
git rebase <parent>
```

Still record backups and inspect the resulting commit range and three-dot diff before pushing.

## Add Review Feedback To The Root Layer

When review feedback belongs to the schema PR, commit it on the schema branch and propagate the rewritten ancestry through API and UI:

```bash
git branch backup/accounts-api-before-schema-fix feature/accounts-api
git branch backup/accounts-ui-before-schema-fix feature/accounts-ui
old_api=$(git rev-parse feature/accounts-api)

git switch feature/accounts-schema
# Apply the requested schema fix.
git add --patch
git commit -m 'fix: address account schema review'
make test-schema

git switch feature/accounts-api
git rebase feature/accounts-schema
new_api=$(git rev-parse feature/accounts-api)

git switch feature/accounts-ui
git rebase --onto "$new_api" "$old_api"

git push origin feature/accounts-schema
git push --force-with-lease origin feature/accounts-api
git push --force-with-lease origin feature/accounts-ui
```

If feedback belongs only to the API layer, commit it on `feature/accounts-api`, save the old API tip first, and restack only `feature/accounts-ui`.

## Amend Or Rebase The Root Layer

When the root commit itself is rewritten, save both the old root and old child tips:

```bash
git branch backup/accounts-schema-before-amend feature/accounts-schema
git branch backup/accounts-api-before-amend feature/accounts-api
git branch backup/accounts-ui-before-amend feature/accounts-ui
old_schema=$(git rev-parse feature/accounts-schema)
old_api=$(git rev-parse feature/accounts-api)

git switch feature/accounts-schema
# Edit the existing root change.
git add --patch
git commit --amend --no-edit
new_schema=$(git rev-parse feature/accounts-schema)

git switch feature/accounts-api
git rebase --onto "$new_schema" "$old_schema"
new_api=$(git rev-parse feature/accounts-api)

git switch feature/accounts-ui
git rebase --onto "$new_api" "$old_api"

git push --force-with-lease origin feature/accounts-schema
git push --force-with-lease origin feature/accounts-api
git push --force-with-lease origin feature/accounts-ui
```

## Sync The Entire Stack With An Updated Integration Branch

Rebase the root onto the updated integration branch, then restack each descendant using saved old and new tips:

```bash
git fetch origin
git branch backup/accounts-schema-before-main-sync feature/accounts-schema
git branch backup/accounts-api-before-main-sync feature/accounts-api
git branch backup/accounts-ui-before-main-sync feature/accounts-ui
old_schema=$(git rev-parse feature/accounts-schema)
old_api=$(git rev-parse feature/accounts-api)

git switch feature/accounts-schema
git rebase origin/main
new_schema=$(git rev-parse feature/accounts-schema)

git switch feature/accounts-api
git rebase --onto "$new_schema" "$old_schema"
new_api=$(git rev-parse feature/accounts-api)

git switch feature/accounts-ui
git rebase --onto "$new_api" "$old_api"

git diff --stat origin/main...feature/accounts-schema
git diff --stat feature/accounts-schema...feature/accounts-api
git diff --stat feature/accounts-api...feature/accounts-ui

git push --force-with-lease origin feature/accounts-schema
git push --force-with-lease origin feature/accounts-api
git push --force-with-lease origin feature/accounts-ui
```

## Resolve Or Abort A Rebase Conflict

During any restack, inspect conflicts before resolving them:

```bash
git status --short
git diff --name-only --diff-filter=U
git diff --cc
```

After resolving only the intended files:

```bash
git add path/to/resolved-file
git rebase --continue
```

If the correct resolution is unclear or would change another layer's behavior, abort and return to the saved backup refs:

```bash
git rebase --abort
git log --graph --decorate --oneline --all --max-count=40
```

Do not push a partially restacked chain. Complete and verify every affected descendant or leave the remote stack unchanged.

## Land One Layer

After the parent PR merges:

```bash
git fetch origin
git merge-base --is-ancestor <old-parent-tip> origin/<trunk>
```

If the command succeeds, the integration branch contains the old parent ancestry. Retarget the child and verify its diff:

```bash
gh pr edit <child-pr> --base <trunk>
git diff origin/<trunk>...<child>
```

If the ancestry check fails, replay only the child's unique commits onto the updated integration branch:

```bash
git switch <child>
git rebase --onto origin/<trunk> <old-parent-tip>
git diff origin/<trunk>...<child>
git push --force-with-lease origin <child>
gh pr edit <child-pr> --base <trunk>
```

The child's rewrite changes the parent of every remaining descendant. Restack those descendants root to leaf using their saved old tips, then verify all remote PR bases:

```bash
gh pr list --state open --json number,title,headRefName,baseRefName,isDraft,url
```

Delete a merged parent branch only after its child PR has the intended base and unique diff.

## Land The Concrete Stack With Merge Commits

When the repository uses merge commits, the merged root tip may remain an ancestor of `main`:

```bash
old_schema=$(git rev-parse feature/accounts-schema)
gh pr merge 101 --merge
git fetch origin

git merge-base --is-ancestor "$old_schema" origin/main
gh pr edit 102 --base main
git diff --stat origin/main...feature/accounts-api
gh pr diff 102 --name-only
```

If the ancestry check succeeds and the API diff contains only API changes, no API history rewrite is needed. Continue by merging PR `#102`, then repeat for PR `#103`.

## Land The Concrete Stack With Squash Or Rebase Merge

Squash and rebase merges usually do not preserve the original root commit IDs. Save the old tips before merging, then replay only child-owned commits:

```bash
old_schema=$(git rev-parse feature/accounts-schema)
old_api=$(git rev-parse feature/accounts-api)

gh pr merge 101 --squash
git fetch origin

git switch feature/accounts-api
git rebase --onto origin/main "$old_schema"
new_api=$(git rev-parse feature/accounts-api)

git switch feature/accounts-ui
git rebase --onto "$new_api" "$old_api"

git diff --stat origin/main...feature/accounts-api
git diff --stat feature/accounts-api...feature/accounts-ui
git push --force-with-lease origin feature/accounts-api
git push --force-with-lease origin feature/accounts-ui

gh pr edit 102 --base main
gh pr view 102 --json number,headRefName,baseRefName,url
```

For a repository using rebase merge, use its required merge option but perform the same ancestry check instead of assuming commit IDs were preserved.

## Repair A Wrong PR Base

Changing only the GitHub PR base is correct when the branch ancestry already matches the intended stack:

```bash
git merge-base --is-ancestor <intended-parent> <child>
git diff <intended-parent>...<child>
gh pr edit <child-pr> --base <intended-parent>
```

If the intended parent is not an ancestor, first repair the Git history with a reviewed rebase or rebuild. A cosmetic `gh pr edit --base` can hide the topology problem temporarily but usually produces misleading commits or diffs.

## Repair A Branch Built From The Wrong Parent

If the API branch was accidentally created from `main` but should be based on the schema branch, find its fork point, replay its unique commits, and then restack the UI:

```bash
git fetch origin
git branch backup/accounts-api-wrong-parent feature/accounts-api
git branch backup/accounts-ui-before-parent-repair feature/accounts-ui
api_fork=$(git merge-base origin/main feature/accounts-api)
old_api=$(git rev-parse feature/accounts-api)

git switch feature/accounts-api
git rebase --onto feature/accounts-schema "$api_fork"
new_api=$(git rev-parse feature/accounts-api)

git switch feature/accounts-ui
git rebase --onto "$new_api" "$old_api"

git merge-base --is-ancestor feature/accounts-schema feature/accounts-api
git diff --stat feature/accounts-schema...feature/accounts-api
git diff --stat feature/accounts-api...feature/accounts-ui
git push --force-with-lease origin feature/accounts-api
git push --force-with-lease origin feature/accounts-ui
gh pr edit 102 --base feature/accounts-schema
```

## Change PR State Or Metadata

```bash
# Mark a draft PR ready for review.
gh pr ready 102

# Return a downstream PR to draft while its parent is changing.
gh pr ready 103 --undo

# Change title, add a reviewer, or leave a stack-status comment.
gh pr edit 102 --title 'Add validated accounts API'
gh pr edit 102 --add-reviewer octocat
gh pr comment 102 --body 'Restacked onto the updated schema branch; unique diff reverified.'

# Verify the final remote relationship.
gh pr view 102 --json number,title,headRefName,baseRefName,isDraft,state,url
```

## Delete A Retired Stack Branch

Only after every child PR has been retargeted and verified, and only when branch deletion is authorized:

```bash
gh pr list --state open --json number,headRefName,baseRefName,url
git push origin --delete feature/accounts-validation
git branch --delete feature/accounts-validation
```

Use `git branch --delete`, not `--force`, so Git refuses to remove a local branch whose commits are not safely reachable.
