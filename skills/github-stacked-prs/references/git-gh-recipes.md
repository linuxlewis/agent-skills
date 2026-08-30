# Git and GitHub CLI Recipes

Use these recipes only after reading the repository's instructions and
adapting branch names, remotes, merge method, and checks. Placeholders such as
`<trunk>` are not literal shell input.

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

GitHub pull requests use a three-dot comparison, so
`git diff <base>...<head>` is the useful local approximation. Confirm ancestry
separately:

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

Add `--draft` when the PR should not yet request review. Do not put secrets,
internal tokens, or unredacted command output in the body file.

## PR Body Shape

Use the repository's PR template when present. Add stack navigation without
duplicating the entire feature description:

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

After all PRs exist, edit descriptions to replace provisional branch
references with PR links when useful.

## Split An Existing Cumulative Branch

First preserve the existing tip with a clearly named backup branch. Inspect
the commits and identify the last commit for each intended slice. If the
commits already form the desired order, branches can point at those boundary
commits without rewriting them:

```bash
git branch backup/<topic>-before-stack <existing-branch>
git branch <topic>-01-foundation <first-boundary-sha>
git branch <topic>-02-api <second-boundary-sha>
git branch <topic>-03-ui <existing-branch>
```

If commits mix concerns, prefer building fresh root-to-leaf branches from the
integration branch and cherry-picking or recreating only the changes owned by
each slice. Use interactive rebase only when rewriting the existing branch is
authorized and the commit boundaries can be verified. Never delete the backup
ref until the published stack and diffs are confirmed.

## Restack After A Parent Changes

Before rewriting descendants, record the old tip of every affected branch with
backup refs. Suppose `<parent>` changed from `<old-parent-tip>` to
`<new-parent-tip>`:

```bash
git switch <child>
git rebase --onto <new-parent-tip> <old-parent-tip>
git diff <new-parent-tip>...<child>
git push --force-with-lease origin <child>
```

For a grandchild, use the child's old tip and newly rebased tip as the old and
new parent tips. Continue one edge at a time toward the leaf. Do not calculate
later rebases from a branch whose old tip was not recorded.

If the parent only gained commits and its old history is still intact, this
simpler form may be sufficient:

```bash
git switch <child>
git rebase <parent>
```

Still record backups and inspect the resulting commit range and three-dot diff
before pushing.

## Land One Layer

After the parent PR merges:

```bash
git fetch origin
git merge-base --is-ancestor <old-parent-tip> origin/<trunk>
```

If the command succeeds, the integration branch contains the old parent
ancestry. Retarget the child and verify its diff:

```bash
gh pr edit <child-pr> --base <trunk>
git diff origin/<trunk>...<child>
```

If the ancestry check fails, replay only the child's unique commits onto the
updated integration branch:

```bash
git switch <child>
git rebase --onto origin/<trunk> <old-parent-tip>
git diff origin/<trunk>...<child>
git push --force-with-lease origin <child>
gh pr edit <child-pr> --base <trunk>
```

The child's rewrite changes the parent of every remaining descendant. Restack
those descendants root to leaf using their saved old tips, then verify all
remote PR bases:

```bash
gh pr list --state open --json number,title,headRefName,baseRefName,isDraft,url
```

Delete a merged parent branch only after its child PR has the intended base and
unique diff.

## Repair A Wrong PR Base

Changing only the GitHub PR base is correct when the branch ancestry already
matches the intended stack:

```bash
git merge-base --is-ancestor <intended-parent> <child>
git diff <intended-parent>...<child>
gh pr edit <child-pr> --base <intended-parent>
```

If the intended parent is not an ancestor, first repair the Git history with a
reviewed rebase or rebuild. A cosmetic `gh pr edit --base` can hide the topology
problem temporarily but usually produces misleading commits or diffs.
