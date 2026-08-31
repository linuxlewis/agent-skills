# Creating Stacks

## When To Create A Stack

Create a stack when one piece of work naturally divides into dependent,
reviewable layers. Good signals include:

- later work depends on earlier work but the earlier layer can be reviewed now;
- different layers need different reviewers;
- one PR would be too large, but independent PRs cannot be developed or
  reviewed separately; or
- the work has a clear bottom-to-top story, such as schema, API, UI, then
  integration tests.

Keep the work in one PR when it is already easy to review. Create separate,
ordinary PRs when changes are independent. Create separate stacks for unrelated
features. Native stacks must use branches in one repository and must remain a
linear chain; forks and branching dependency graphs are unsupported.

## Plan The Layers

Put foundations at the bottom and dependents above them:

```text
(main) <- checkout/schema <- checkout/service <- checkout/ui <- checkout/tests
```

Each layer should have one clear purpose and be useful to review on its own.
Changes in a higher layer may depend on lower layers, but lower layers must not
depend on higher ones.

Prefer recognizable names such as `<topic>/<concern>` when the repository has
no stronger branch naming convention:

```text
checkout/schema
checkout/service
checkout/ui
checkout/tests
```

## Prerequisites

```bash
git --version
gh --version
gh auth status
gh extension install github/gh-stack
```

GitHub documents Git 2.20+ and GitHub CLI 2.90+ as the minimum versions. The
public preview requires no repository enablement on supported GitHub.com
repositories.

## Create Layers Incrementally

Start the bottom layer from the repository's default branch:

```bash
git switch main
git pull --ff-only
gh stack init checkout/schema

git add db/schema.sql src/models/order.ts
git commit -m "Add checkout order schema"
```

Add each new layer from the current top:

```bash
gh stack add checkout/service
git add src/services/checkout.ts
git commit -m "Add checkout service"

gh stack add checkout/ui
git add src/components/CheckoutForm.tsx
git commit -m "Add checkout form"
```

`gh stack add` carries uncommitted changes onto the new branch. Commit or stash
first when the next layer should start clean.

To use a non-default trunk:

```bash
gh stack init --base release checkout/schema
```

## Create Several Branches At Once

`init` accepts branches in bottom-to-top order and checks out the last one:

```bash
gh stack init checkout/schema checkout/service checkout/ui
```

This creates missing branches and adopts existing branches. Existing branches
must already have the intended linear ancestry.

## Create The Native Stack And PRs

Interactive submission opens an editor for PR titles, descriptions, and draft
state:

```bash
gh stack submit
```

For generated titles without the editor:

```bash
gh stack submit --auto
```

New PRs created by `--auto` are drafts. Mark new and existing PRs ready for
review with:

```bash
gh stack submit --auto --open
```

With multiple remotes, select one explicitly:

```bash
gh stack submit --auto --remote origin
```

`submit` pushes all active branches, creates missing PRs, updates existing PR
bases, and creates or updates native Stack membership. Edit generated PR
metadata afterward when needed:

```bash
gh pr edit 41 --title "Add checkout order schema" --body-file pr-schema.md
```

## Link Existing Branches Or PRs

Use `link` when branches or PRs already exist or another tool manages the Git
history. Always list arguments bottom-to-top:

```bash
# Existing branches
gh stack link checkout/schema checkout/service checkout/ui

# Existing PR numbers
gh stack link 41 42 43

# Existing PR URLs
gh stack link \
  https://github.com/OWNER/REPO/pull/41 \
  https://github.com/OWNER/REPO/pull/42

# Non-default trunk and ready-for-review PRs
gh stack link --base release --open release/schema release/api
```

Append new work to native stack number 7 without relisting its current PRs:

```bash
gh stack link 7 checkout/analytics checkout/polish
```

`link` pushes branch arguments, creates missing PRs, corrects their bases, and
creates or extends native membership. It does not create local `gh stack`
tracking. To navigate the stack locally later:

```bash
gh stack checkout 7
```

## Create A Stack In GitHub's UI

1. Create the bottom PR against the trunk.
2. Create the next PR against the bottom PR's branch.
3. Select **Create stack** when GitHub offers it.
4. Add later PRs so each targets the branch immediately below it.

GitHub may also offer to convert an eligible linear PR chain into a native
Stack. Confirm native membership by checking for the stack map and stack
number; chained PR bases alone do not prove membership.
