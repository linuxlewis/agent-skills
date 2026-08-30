---
name: github-stacked-prs
description: Plan, create, update, review, merge, or repair stacked GitHub pull requests with Git and the GitHub CLI. Use when a change should be split into dependent PRs; do not use for unrelated or independently mergeable PRs.
license: MIT
compatibility: Requires Git and GitHub CLI (gh) authenticated for remote PR operations
metadata:
  author: linuxlewis
  version: "1.0.0"
---

# GitHub Stacked PRs

A stack is an ordered chain of branches and pull requests. The root branch
starts from the repository's integration branch; every later branch starts
from the preceding branch. The root PR targets the integration branch, and
each child PR targets its immediate parent branch. GitHub then shows each PR's
unique slice rather than the cumulative feature diff.

Use plain `git` and `gh` unless the repository already standardizes on
Graphite, git-spice, or another stack manager. If it does, follow the
repository's documented tool and do not mix stack-management methods or
install a different tool without being asked.

## Decide Whether To Stack

Create a stack when all of these are true:

- the work has a real dependency order, such as schema -> data layer -> API ->
  UI;
- each slice has one reviewable purpose and a meaningful validation story;
- merging from the root upward leaves the repository in an acceptable state;
  and
- parallel review or earlier feedback is worth the bookkeeping and rebase
  cost.

Prefer a single PR when splitting would create artificial boundaries,
intermediate states cannot be understood or validated, or the total change is
already easy to review. Use independent branches from the integration branch
when changes do not depend on one another. Avoid a stack when repository
policy, merge queues, required checks, or contributor permissions cannot
support PRs based on non-default branches.

Keep the stack as short as the dependency chain permits. If it grows difficult
to describe in a small table, reconsider the boundaries or split it into
independently mergeable groups.

## Inspect Before Acting

Read repository instructions and inspect the worktree, branch graph, remotes,
default branch, existing PRs, branch protections, and merge strategy. Do not
assume the integration branch is named `main` or that the current branch is the
stack root.

Preserve unrelated user changes. Do not reset, discard, silently relocate, or
include them in stack commits. Before rewriting any existing stack, identify
shared branches and create recoverable backup refs for the affected tips.

Planning or explanation requests stop before branch creation, pushes, or PR
mutations. Push, create, edit, close, or merge PRs only when the user's request
authorizes those external changes.

## Plan The Stack

State the proposed stack before creating it. For every slice, record:

- order, branch, and intended PR title;
- base branch and dependency on the preceding slice;
- behavior introduced by this slice alone;
- tests or checks that establish its correctness; and
- whether it is ready for review or should initially be a draft.

Choose boundaries by behavior and dependency, not by equal line counts or
arbitrary file groups. Put shared contracts, migrations, and mechanical
prerequisites low in the stack. Keep follow-up behavior, integrations, and
presentation layers higher. A child may rely on its ancestors, but an ancestor
must not rely on code introduced only by a child.

## Create Root To Leaf

1. Update knowledge of the remote integration branch without overwriting local
   work.
2. Create the root branch from that integration branch, implement or isolate
   only the first slice, commit it, and run its relevant checks.
3. Create each child from its immediate parent. Commit only that child's unique
   slice and validate both the slice and the cumulative branch state.
4. Before publishing, verify each parent is an ancestor of its child and
   inspect `git log <parent>..<child>` plus `git diff <parent>...<child>`. The
   result must match the intended PR scope.
5. Push root to leaf. Create the root PR against the integration branch and each
   child PR against its immediate parent branch.
6. Verify the remote PR head/base pairs after creation. Do not rely on defaults
   selected by `gh` or the GitHub UI.

Every PR description should identify its position, parent, child if any,
unique scope, validation, and review order. Link the neighboring PRs. Mark
downstream PRs as draft when their ancestor or interface is still likely to
change; otherwise they may be reviewed in parallel, always as their unique
diffs.

For exact inspection, creation, and PR-body command patterns, read
[Git and GitHub CLI recipes](references/git-gh-recipes.md).

## Maintain The Stack

Apply a requested fix to the lowest branch that logically owns it. Then
propagate the changed ancestry from that branch toward the leaf.

- If a parent only gains commits, rebase its child onto the updated parent,
  then continue root to leaf.
- If a parent was amended, rebased, or otherwise rewritten, save the old and
  new parent tips and rebase the child with `--onto`. Repeat with the child's
  old and new tips for every descendant.
- Prefer `--force-with-lease` after an authorized rewrite. Never use an
  unqualified `--force`.
- If branches are shared, force-push is prohibited, or reviewers depend on
  stable commit IDs, prefer additive fix commits or coordinate before
  rewriting.
- After every restack, verify the unique diff, rerun relevant checks, push in
  order, and confirm that every PR still has the correct base.

Do not merge parent branches into children merely to make the graph current
unless the repository explicitly uses merge-based stack maintenance. Mixing
merge commits and rebases makes the unique diffs and later cleanup harder to
reason about.

## Review And Merge

Review and merge from root to leaf. A child PR's green checks do not make an
unmerged ancestor optional.

After merging a parent, fetch the integration branch and determine whether the
old parent tip is now its ancestor. A merge commit may preserve ancestry;
squash merge, rebase merge, and some merge queues usually do not.

- If ancestry is preserved, retarget the immediate child PR to the integration
  branch and verify that only the child's slice remains.
- If ancestry is not preserved, rebase the child so only its unique commits are
  replayed onto the updated integration branch, then restack every descendant
  from that rewritten child.

Retarget and verify downstream PR bases before deleting merged parent branches.
Never assume GitHub's automatic retargeting produced a clean diff. Repeat the
merge, ancestry check, retarget or restack, validation, and base verification
for each layer.

Honor the repository's required merge method and approval rules. Do not change
branch protection, bypass required checks, or switch merge strategy merely to
make a stack easier to land.

## Stop Conditions

Stop and report the state instead of guessing when:

- the integration branch, stack order, or intended PR bases cannot be
  established;
- a proposed slice is not independently understandable or leaves an unsafe
  intermediate state;
- unexpected commits appear in a unique diff;
- history rewriting would affect unowned or shared work;
- force-push or non-default PR bases conflict with repository policy;
- conflicts cannot be resolved without changing behavior outside the requested
  slice; or
- checks fail and the failure cannot be attributed and corrected within scope.

The handoff should show the stack in root-to-leaf order with PR links,
branch/base pairs, draft state, validation, and any required merge or restack
instructions.
