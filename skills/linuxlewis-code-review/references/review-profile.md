# Linuxlewis Review Profile

This skill captures linuxlewis's observed review style in a reusable, portable
form. The intent is to produce high-signal review comments, not a generic style
guide.

## Core Profile

1. Scope and ownership discipline

   Keep PRs focused, remove unrelated changes, and move code to the module or
   app that owns the behavior. Avoid extending legacy patterns when a cleaner
   boundary is available.

2. Existing patterns over one-off code

   Prefer framework-native and repo-native mechanisms before custom code:
   generated clients, factories, retry decorators, feature flag systems, service
   patterns, serializers, managers, and established provider wrappers.

3. API contract and caller-shape correctness

   Check whether endpoints, routes, serializers, generated clients, input/output
   types, and mobile/web differences match the actual callers. Push back on
   ambiguous parameters and half-generic/half-client-specific APIs.

4. Auth, session, and security details

   Review cookies vs tokens, refresh/logout flows, passkeys, OAuth boundaries,
   signed URLs, secret/config handling, staff access, impersonation, audit
   logging, and rate limits.

5. Domain invariants and legacy data

   Review the business semantics directly. Historical records, legacy clients,
   fallback behavior, locked records, state transitions, and edge cases matter as
   much as local code correctness.

6. Data integrity and migration safety

   Look for duplicate rows, missing constraints, migration-created data,
   idempotent backfills, destructive infrastructure side effects, and whether
   new stored data should instead reference existing durable records.

7. Operational safety and async boundaries

   Push slow or failure-prone work out of request paths. Prefer background jobs,
   async views, polling, input caps, existing retry/rate-limit clients, and
   simple operational models over ad hoc synchronous work.

8. Query and production-scale behavior

   Ask how many rows production has, whether filters are indexed, whether broad
   queries need pagination, whether writes should use bulk operations, and
   whether data can be loaded upfront to avoid downstream queries.

9. Test usefulness over test volume

   Optimize tests for trust. The preferred test executes the changed app code
   path and mocks only the unstable external boundary. Patching a whole service,
   task, view, SDK method, or internal helper is suspect when it hides the
   integration the PR needs to prove.

10. Type specificity and cleanup

   Ask for narrower types/enums, removal of dead fields/fallbacks, clearer names,
   and comments/docstrings that explain domain behavior rather than ticket
   history.

## Review Strategy

Review from the outside in:

1. What behavior is this PR supposed to deliver?
2. Which product, data, auth, or operational boundary does it touch?
3. Does it fit existing architecture and local patterns?
4. Does it perform correctly at production scale?
5. Do the tests prove the real integration path?
6. Are there nits worth mentioning after substantive risks are handled?

Use concise comments, usually framed as questions:

- `Can we ... ?`
- `Do we need ... ?`
- `Why are we ... ?`
- `Should we ... ?`
- `nit: ...`

For substantive issues, include the consequence: held API workers, duplicate
records, broken legacy clients, unclear ownership, weak security boundaries,
slow backfills, unindexed production queries, or tests that would still pass if
the integration broke.
