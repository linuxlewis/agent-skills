---
name: high-value-tests
description: Write or review the smallest test set that gives useful confidence in a code change. Use when adding, generating, or reviewing tests, especially when agent-generated tests may duplicate coverage or add low-value cases.
license: MIT
metadata:
  author: linuxlewis
  version: "1.0.0"
---

# High-Value Tests

Protect the changed behavior with the fewest tests that give useful confidence.
Test count is not a goal.

## Choose the Coverage

1. Read the change, existing tests, and repository test guidance before writing
   tests.
2. Name the primary user path and the real risks introduced or changed by the
   work.
3. Keep one strong test for the primary user path by default. Extend an existing
   test when it can cover the behavior without making the failure unclear.
4. Add another test only when it protects a distinct risk, such as:
   - a materially different branch or failure mode;
   - permissions, security, privacy, or data integrity;
   - idempotency or a state transition;
   - an external API or persisted-data contract;
   - a boundary condition with a credible failure;
   - a regression that the primary-path test does not detect.

For each additional test, complete: `This test protects X from Y.` If the answer
is the same as another test, merge or remove the duplicate.

## Use the Right Test Layer

- Test at the smallest layer that observes the contract. Do not repeat the same
  risk at unit, API, integration, and end-to-end layers.
- Exercise the real application path. Mock external I/O boundaries, not the
  application method under test.
- Keep assertions on required boundary calls and arguments when the call is the
  observable behavior, such as queuing a task or sending an event.
- Prefer existing factories, helpers, and parameterization over repeated setup
  and near-identical test functions.
- Assert behavior and contracts. Do not test framework behavior, constants, or
  implementation details unless the change depends on them.

## Remove or Flag Low-Value Cases

When writing or editing tests, remove or combine the cases below. During a
review, report them as findings and do not edit files unless the user asks for
changes.

- repeat the same behavior with values that do not change the result;
- duplicate stronger coverage at another layer;
- only prove that internal mocks were called without verifying an observable
  contract;
- patch away the code path that they claim to test;
- add one test per field or branch when one focused or parameterized test is
  clearer;
- cannot identify a distinct failure that they would catch.

Do not remove required security, privacy, data-integrity, API-contract, or known
regression coverage to reduce test time. Follow explicit repository or user
requirements when they require broader coverage.

## Report the Result

Run the smallest relevant test command for the tests you changed. If it cannot
run, state the command and the reason instead of claiming verified coverage.

Summarize the behaviors and risks covered. If you remove or skip likely test
cases, state which ones and why their coverage is duplicate or low value.
