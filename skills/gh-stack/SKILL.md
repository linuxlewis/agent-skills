---
name: gh-stack
description: Use GitHub's native Stacked PRs and the official gh stack CLI extension to plan, create, link, review, rebase, modify, sync, merge, or unstack dependent pull requests. Use whenever a user mentions GitHub Stacks, native stacked PRs, a stack map, or gh stack; do not substitute a merely chained set of ordinary PRs when native stack membership is requested.
license: MIT
compatibility: Requires Git 2.20+, GitHub CLI 2.90+, and github/gh-stack for CLI workflows
metadata:
  author: linuxlewis
  version: "1.0.0"
  upstream: "github/gh-stack@2bd699a544a09cb5c45a013d03416e0894b0454e"
---

# GitHub Stacked PRs (`gh stack`)

Use this skill for GitHub's native Stacked PRs feature, not just the generic Git
pattern of one branch based on another. Native membership creates a GitHub
Stack object, the stack icon and map in each PR, stack-aware rules and CI, and
stack merge behavior. Create or link that membership with `gh stack submit`,
`gh stack link`, GitHub's **Create stack** UI, or the Stacks API.

As of August 30, 2026, Stacked PRs are in public preview and subject to change.
They require no repository setting or enablement on supported GitHub.com
repositories.

## Native Model

A stack is a strictly linear chain of two or more PRs in one repository:

```text
(main) <- auth <- api <- frontend
```

`main` is the trunk. `auth` is the bottom and merges first; `frontend` is the
top and merges last. `up` moves away from trunk, and `down` moves toward it.

The native invariants are:

- every branch is in the same repository; cross-fork stacks are unsupported;
- each PR targets the branch immediately below it, while the bottom PR targets
  the trunk;
- the branch history must remain fully linear;
- every PR is evaluated against the trunk's rules, required checks, CODEOWNERS,
  and relevant workflows, even when its direct base is another stack branch;
- a stack contains at most 100 PRs; and
- GitHub Desktop does not currently support Stacked PRs.

Read [stack design](references/stack-design.md) before deciding whether to
create a stack or choosing its layers.

## Setup And Authorization

Check prerequisites before relying on the extension:

```bash
gh --version        # GitHub docs require 2.90.0 or later
git --version       # GitHub docs require 2.20 or later
gh auth status
gh extension list
```

If `stack` is missing, install the official extension only when setup is in
scope or the user authorizes the installation:

```bash
gh extension install github/gh-stack
```

Pushing branches, creating or changing PRs, creating or dissolving a Stack,
rebasing remote branches, and merging are remote mutations. Perform them only
when the request authorizes that operation. Planning and explanation requests
stop before mutation.

With multiple remotes, pass `--remote <name>` to `push`, `submit`, `sync`,
`rebase`, or `link`, or configure `remote.pushDefault`. Never guess the remote.

## Non-Interactive Agent Use

`gh stack` changes behavior when stdout is a TTY. Agent shells may allocate a
PTY, so use explicit non-interactive forms:

| Use | Avoid bare | Reason |
|---|---|---|
| `gh stack view --json` | `gh stack view` | Bare view may open a TUI. |
| `gh stack submit --auto` | `gh stack submit` | Bare submit opens an editor. |
| `gh stack merge <target> --yes` | `gh stack merge` | Bare merge opens a wizard. |
| `gh stack init <branch>...` | `gh stack init` | Bare init prompts for names. |
| `gh stack add <branch>` | `gh stack add` | Bare add prompts for a name. |
| `gh stack checkout <target>` | `gh stack checkout` | Bare checkout opens a picker. |
| `up`, `down`, `top`, `bottom`, `trunk` | `gh stack switch` | Switch is menu-only. |

`gh stack modify` is TUI-only. Do not invoke it from an unattended agent
session. Explain the operation or use the non-interactive rebuild procedure in
[troubleshooting](references/troubleshooting.md).

## Inspect Before Editing

```bash
git status --short --branch
git remote -v
gh stack view --json
```

The JSON result contains `trunk`, `currentBranch`, and `branches[]`; every
branch includes `name`, `head`, `base`, `isCurrent`, `isMerged`, `isQueued`, and
`needsRebase`, plus PR metadata when a PR exists.

Preserve unrelated worktree changes. If the current branch is not tracked
locally but the native Stack exists on GitHub, check it out by stack number, PR
number, PR URL, or branch:

```bash
gh stack checkout 7
gh stack checkout 42
gh stack checkout https://github.com/OWNER/REPO/pull/42
gh stack checkout feature/auth
```

## Create New Work As A Native Stack

Plan bottom-to-top layers before editing, then create each layer when its
concern begins:

```bash
gh stack init billing/schema
# Implement, stage, commit, and validate the schema layer.

gh stack add billing/api
# Implement, stage, commit, and validate the API layer.

gh stack add billing/ui
# Implement, stage, commit, and validate the UI layer.

gh stack submit --auto --remote origin
gh stack view --json
```

`submit --auto` pushes every active branch, creates or updates each PR with the
correct base, and creates or updates the native Stack on GitHub. New PRs are
drafts unless `--open` is passed:

```bash
gh stack submit --auto --open --remote origin
```

Use `gh pr edit` after submission when custom PR titles or bodies are needed;
`submit --auto` generates them from commits or branch names.

## Link Existing Branches Or PRs

Use `link` when branches or PRs already exist, or another tool manages the Git
history. Arguments are bottom-to-top:

```bash
gh stack link --remote origin auth api frontend
gh stack link 41 42 43
gh stack link https://github.com/OWNER/REPO/pull/41 https://github.com/OWNER/REPO/pull/42
gh stack link 7 ui-polish                 # append to native stack #7
gh stack link --base release --open a b c
```

`link` creates or corrects PR bases and creates or extends the native Stack,
but writes no `.git/gh-stack` local tracking state. Membership updates are
additive; it never removes an existing PR. Use `checkout <stack-number>` later
if local tracking and navigation are needed.

Do not call a chain of PR bases a completed native Stack until the Stack object
exists and `gh stack` or the GitHub stack map confirms membership. For website
creation, native UI behavior, CI, and server-side rebasing, read
[native GitHub behavior](references/native-github.md).

## Apply Changes To The Owning Layer

Check out the lowest layer that owns the change, commit there, then cascade the
change upward:

```bash
gh stack checkout billing/api
git add path/to/changed-files
git commit -m "Address API review feedback"
gh stack rebase --upstack --remote origin
gh stack push --remote origin
gh stack top
```

Do not manually reproduce the old SHA bookkeeping for normal native stacks.
`gh stack rebase` handles cascading rebases and merged or squash-merged parents.

## Synchronize And Recover

Use `sync` for routine reconciliation:

```bash
gh stack sync --remote origin
gh stack sync --prune --remote origin
```

It fetches, reconciles the native Stack, fast-forwards the trunk, cascade
rebases if needed, pushes, refreshes PR state, syncs the Stack object, and
optionally prunes merged local branches. It never creates missing PRs; use
`submit --auto` for that.

On a rebase conflict:

```bash
git add path/to/resolved-files
gh stack rebase --continue
```

Abort the whole cascade with `gh stack rebase --abort`. A failed `sync` already
restores all branches; rerun `gh stack rebase` to stop at and resolve the
conflict. Read [troubleshooting](references/troubleshooting.md) for divergence,
squash merges, interrupted operations, and non-interactive restructuring.

## Merge With Native Stack Semantics

Never use `gh pr merge` to merge a native Stack. Use:

```bash
gh stack merge 42 --yes --squash   # PR #42 and every unmerged PR below it
gh stack merge 7 --yes --merge     # all unmerged PRs in native stack #7
gh stack merge 42 --yes --rebase
```

A direct stack merge is all-or-nothing. Selecting a mid-stack PR includes every
unmerged PR below it. If the trunk uses a merge queue, GitHub queues the stack
instead, preserves bottom-to-top order, and chooses the merge method.

Auto-merge and rule bypass are not currently supported. Every selected PR and
every unmerged PR below it must meet the trunk's requirements. After a partial
merge, GitHub automatically retargets and cascade-rebases the remaining stack.

## Unstacking

```bash
gh stack unstack                 # remove local and GitHub grouping
gh stack unstack 7               # target native stack #7
gh stack unstack --local         # keep the Stack on GitHub
```

Unstacking removes grouping, not branches or PRs. Merged or queued PRs cannot
be removed from their stack. Do not delete branches or close PRs unless the
user separately requests that destructive action.

## References And Sources

Read only the reference relevant to the current task:

- [stack design](references/stack-design.md) — when deciding whether to stack,
  planning layers, or naming branches;
- [command behavior](references/commands.md) — for preconditions, side effects,
  atomicity, and exact command behavior;
- [troubleshooting](references/troubleshooting.md) — for conflicts, divergence,
  squash merges, restructuring, and recovery; and
- [native GitHub behavior](references/native-github.md) — for GitHub website,
  rules, CI, merge queue, API, and preview limitations.

This skill is based on GitHub's official `github/gh-stack` skill at commit
`2bd699a544a09cb5c45a013d03416e0894b0454e` and current GitHub Stacked PRs
documentation. `gh stack <command> --help` is authoritative for the installed
extension version.
