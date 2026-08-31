# Native GitHub Stack Behavior

GitHub Stacked PRs are a native GitHub feature in public preview as of August
30, 2026. They require no repository setting or enablement on supported
GitHub.com repositories.

## Native Membership Versus A Branch Chain

A sequence of PRs with chained base branches is the required Git shape, but
native membership is a GitHub Stack object. Native membership adds the stack
icon and map, trunk-aware rules and CI, stack rebasing, and stack merge
semantics.

Create native membership with one of these paths:

- `gh stack submit` from a locally tracked stack;
- `gh stack link` for existing branches or PRs without local tracking;
- **Create stack**, **Add to stack**, or the recommendation banner on GitHub;
  or
- the Stacks REST API.

Verify membership through `gh stack view --json`, the stack number returned by
GitHub, or the stack map in the PR merge box. Do not infer native membership
solely from base branch names.

## Create Or Extend A Stack On GitHub

To create a stack entirely on the website:

1. Create the bottom PR against the intended trunk.
2. Create the next PR with its base set to the bottom PR's head branch.
3. Select **Create stack** to link the PRs natively.
4. Repeat for each additional PR, targeting the branch immediately below it.

When eligible open PRs already form a linear chain, GitHub may show a
recommendation banner. Confirming the preview converts the chain into a native
Stack.

To add a new top PR, open the stack map, choose **Add to stack**, select the new
head branch, create the PR, and confirm **Add to stack**. Website additions are
top-only; use `gh stack modify` for in-place restructuring.

## Rules And CI

GitHub evaluates every PR as if it targets the stack trunk:

- required reviews and status checks come from the trunk's protections or
  rulesets;
- CODEOWNERS are evaluated from the trunk, so a CODEOWNERS change in a lower
  layer does not affect higher layers in the same stack;
- code scanning and `pull_request` workflows targeting the trunk run for every
  PR in the stack; and
- all PRs below the selected merge point must also satisfy their requirements.

Workflow payloads expose `github.event.pull_request.stack` with the native
stack number, size, 1-based position, and base ref/SHA. A newly opened PR is not
yet stacked, so `pull_request.opened` does not contain that object. Listen for
the `pull_request` webhook action `stacked` when membership is created.

## Rebase Behavior

A native Stack must have linear ancestry. When it does not, GitHub displays
**Rebase stack** in the merge box. A server-side rebase:

1. rebases the stack on the latest trunk;
2. cascades every unmerged branch on top of the branch below it; and
3. force-pushes the rewritten branches.

Server-created rebase commits are not signed. Repositories requiring signed
commits should use `gh stack rebase` locally and then `gh stack push`.

After a partial stack merge, GitHub automatically retargets the next unmerged
PR to the trunk and cascade-rebases the remaining branches. The CLI and server
handle squash-merged parents with `rebase --onto`; ordinary manual replay is not
required.

## Merge Behavior

Selecting a PR for merge includes it and every unmerged PR below it. A middle
PR cannot merge while leaving a lower PR unmerged.

- Direct stack merge is atomic: either the whole selected contiguous group
  lands or none of it does.
- Merge queues accept the stack in order. If one PR is ejected, its descendants
  are also ejected; lower PRs already processed are unaffected.
- Merge commit creates one merge commit for the selected group.
- Squash creates one squashed commit per PR.
- Rebase replays all selected commits onto the trunk.

Native stacks require GitHub's asynchronous merge API or `gh stack merge`.
Legacy synchronous REST/GraphQL merge operations and `gh pr merge` do not
perform stack merges. Auto-merge and rule bypass are not currently supported.

## Limits And Lifecycle

- All branches must be in one repository; forks are unsupported.
- A Stack can contain at most 100 PRs.
- GitHub Desktop does not support Stacked PRs.
- A fully merged Stack is complete and cannot be extended. New unmerged
  branches submitted above it form a new Stack rooted at the trunk.
- Closing a middle PR blocks the PRs above it until the Stack is reorganized.
- Unstacking removes open, draft, and closed PRs from native membership while
  keeping their branches, PRs, and current base branches. Merged or queued PRs
  remain in the Stack.

## Authoritative Sources

- https://docs.github.com/en/pull-requests/get-started/about-stacked-prs
- https://docs.github.com/en/pull-requests/how-tos/create-pull-requests/creating-stacked-pull-requests
- https://docs.github.com/en/pull-requests/how-tos/create-pull-requests/managing-stacked-pull-requests
- https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/merging-stacked-pull-requests
- https://docs.github.com/en/pull-requests/reference/stacked-pull-requests
- https://github.com/github/gh-stack
