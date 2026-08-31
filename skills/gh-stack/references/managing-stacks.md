# Managing Stacks

## View, Checkout, And Navigate

```bash
# Human-readable stack map
gh stack view --short

# Structured output
gh stack view --json

# Move through the current stack
gh stack top
gh stack bottom
gh stack up
gh stack up 2
gh stack down
gh stack trunk

# Check out by stack number, PR number, PR URL, or branch
gh stack checkout 7
gh stack checkout 42
gh stack checkout https://github.com/OWNER/REPO/pull/42
gh stack checkout checkout/service
```

## Add A New Top Layer

`gh stack add` only adds above the current top:

```bash
gh stack top
gh stack add checkout/analytics
git add src/analytics/checkout.ts
git commit -m "Add checkout analytics"
gh stack submit --auto
```

`submit` pushes the new branch, creates its PR, and updates native membership.

## Update A Layer

Make a change on the lowest layer that owns it, then cascade the new history
through every layer above it:

```bash
gh stack checkout checkout/service
git add src/services/checkout.ts
git commit -m "Handle expired checkout sessions"

gh stack rebase --upstack
gh stack push
```

Use `push` when the PRs and membership already exist. It only pushes active
branches; it does not create or update PRs. Use `submit` instead when a PR is
missing, bases changed, membership changed, or PR draft state should change:

```bash
gh stack submit --auto
```

Useful rebase scopes:

```bash
# Current layer through the top
gh stack rebase --upstack

# Trunk through the current layer
gh stack rebase --downstack

# Align stack branches without rebasing onto the latest trunk
gh stack rebase --no-trunk
```

## Sync With Trunk And GitHub

```bash
gh stack sync
gh stack sync --prune
gh stack sync --remote origin
```

`sync` fetches, reconciles local and GitHub stack state, fast-forwards the
trunk, cascade-rebases when needed, pushes active branches, and refreshes
native membership. `--prune` also removes local branches for merged PRs.

`sync` does not create missing PRs. Run `gh stack submit` after syncing when
new branches need PRs.

## Resolve Rebase Conflicts

If `rebase` stops at a conflict:

```bash
# Resolve the files, then:
git add path/to/resolved-file
gh stack rebase --continue
```

Repeat for later conflicts. To restore the whole stack to its state before the
rebase:

```bash
gh stack rebase --abort
```

A failed `sync` restores the branches before returning. Run `gh stack rebase`
afterward when you want to stop at conflicts, resolve them, and continue.

## Restructure A Stack

Use the interactive modify UI to drop, combine, insert, rename, or reorder
layers:

```bash
gh stack modify
```

Common controls:

| Operation | Key |
|---|---|
| Drop selected branch | `x` |
| Fold into branch below | `d` |
| Fold into branch above | `u` |
| Insert below / above | `i` / `I` |
| Rename | `r` |
| Move down / up | `Shift+Down` / `Shift+Up` |
| Undo | `z` |
| Apply | `Ctrl+S` |
| Cancel | `q` |

After applying, update the remote branches, PR bases, and Stack:

```bash
gh stack submit
```

If modify encounters conflicts:

```bash
git add path/to/resolved-file
gh stack modify --continue
```

Abort and restore the pre-modify state with:

```bash
gh stack modify --abort
```

`modify` requires a clean working tree and an interactive terminal. Merged
layers are locked. Reordering cannot be combined with drop, fold, insert, or
rename operations in the same session.

## Merge A Stack

Use native stack merge, not `gh pr merge`:

```bash
# Merge PR #42 and every unmerged PR below it
gh stack merge 42 --yes --squash

# Merge every unmerged PR in native stack #7
gh stack merge 7 --yes --merge

# Rebase-merge the selected portion
gh stack merge 42 --yes --rebase
```

Selecting a middle PR includes every unmerged PR below it; lower layers cannot
be skipped. A direct stack merge is atomic. If the trunk uses a merge queue,
GitHub queues the selected PRs and preserves bottom-to-top order.

After a partial merge, GitHub retargets the next unmerged PR to the trunk and
cascade-rebases the remaining branches. `gh stack sync --prune` refreshes local
state and removes merged local branches.

## Unstack

```bash
# Remove local and GitHub Stack membership for the current stack
gh stack unstack

# Remove native stack number 7
gh stack unstack 7

# Remove local tracking but keep the Stack on GitHub
gh stack unstack --local
```

Unstacking removes grouping only. It does not delete branches or PRs. Merged or
queued PRs remain associated with their Stack.

## Resolve Local And GitHub Divergence

If local tracking and GitHub membership changed independently, choose which
composition to keep.

Keep GitHub's version:

```bash
gh stack unstack --local
gh stack checkout 7
```

Keep the local version:

```bash
gh stack unstack
gh stack submit --auto
```

Both paths keep the branches and PRs; they replace only stack tracking or
membership.
