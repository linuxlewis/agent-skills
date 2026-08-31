---
name: gh-stack
description: Create and manage GitHub's native Stacked PRs with the official gh stack CLI extension or GitHub UI. Use when deciding whether work should be a stack, creating or linking a stack, updating its layers, syncing or restructuring it, merging part or all of it, or removing native stack membership.
license: MIT
metadata:
  author: linuxlewis
  version: "1.0.0"
  upstream: "github/gh-stack@2bd699a544a09cb5c45a013d03416e0894b0454e"
---

# GitHub Stacked PRs (`gh stack`)

Use this skill for GitHub's native Stacked PRs feature. A native Stack is more
than ordinary PRs whose base branches happen to form a chain: GitHub records
the membership and provides a stack map, stack-aware checks, rebasing, and
merge behavior. Create that membership with `gh stack submit`, `gh stack link`,
or GitHub's **Create stack** UI.

As of August 30, 2026, Stacked PRs are in public preview. No repository setting
is required on supported GitHub.com repositories.

## Core Model

A stack is a linear chain of two or more PRs in one repository:

```text
(main) <- billing/schema <- billing/api <- billing/ui
```

`main` is the trunk. `billing/schema` is the bottom and merges first;
`billing/ui` is the top and merges last. Each PR targets the branch directly
below it, and the bottom PR targets the trunk.

Do not use a native Stack for unrelated or independent changes, a small change
that fits one PR, cross-repository or fork-based work, or a dependency graph
that cannot be expressed as one linear chain.

## Setup

CLI workflows require Git 2.20+, GitHub CLI 2.90+, and GitHub's official
extension:

```bash
gh extension install github/gh-stack
```

## Create A Stack

Plan layers bottom-to-top, then create and commit each concern:

```bash
gh stack init billing/schema
# Implement and commit the schema layer.

gh stack add billing/api
# Implement and commit the API layer.

gh stack add billing/ui
# Implement and commit the UI layer.

gh stack submit --auto
```

`submit --auto` pushes the branches, creates or updates their PRs, fixes their
bases, and creates the native Stack. New PRs are drafts unless `--open` is
passed. For existing branches or PRs, link them bottom-to-top:

```bash
gh stack link billing/schema billing/api billing/ui
gh stack link 41 42 43
```

See [creating stacks](references/creating-stacks.md) for when to stack, layer
planning, alternate creation flows, and more exact examples.

## Manage A Stack

Inspect and navigate:

```bash
gh stack view --short
gh stack top
gh stack down
gh stack checkout 42
```

After changing a lower layer, rebase the layers above it and push:

```bash
gh stack checkout billing/api
# Make and commit the change.
gh stack rebase --upstack
gh stack push
```

Use `gh stack sync` to reconcile local branches, GitHub state, trunk updates,
rebases, and pushes. It does not create missing PRs; use `gh stack submit` for
that. Use `gh stack modify` to interactively drop, fold, insert, rename, or
reorder layers.

Merge native stacks with `gh stack merge`, never `gh pr merge`:

```bash
gh stack merge 42 --yes --squash
```

Selecting a PR merges it and every unmerged PR below it. Direct merges are
atomic; merge queues preserve bottom-to-top order. A partial merge
automatically retargets and rebases the remaining stack.

See [managing stacks](references/managing-stacks.md) for exact commands for
editing, syncing, restructuring, merging, unstacking, and conflict recovery.

For non-interactive sessions, `--auto` avoids the submit editor, `--yes` avoids
the merge wizard, and `gh stack view --json` avoids the view TUI.

## Source

Based on GitHub's official `github/gh-stack` skill at commit
`2bd699a544a09cb5c45a013d03416e0894b0454e` and current GitHub Stacked PRs
documentation. `gh stack <command> --help` is authoritative for the installed
extension version.
