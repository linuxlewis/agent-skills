---
name: linuxlewis-code-review
description: Apply linuxlewis's code review style to pull requests, local diffs, or proposed code changes. Use when asked to review code the way linuxlewis would, emulate Sam Bolgert's review strategy, predict likely PR feedback, or explain linuxlewis's review profile.
license: MIT
compatibility: Works with any agent that can inspect code, diffs, and tests.
metadata:
  author: linuxlewis
  version: "1.0.0"
---

# Linuxlewis Code Review

Review code with linuxlewis's observed preferences: scope discipline, local
pattern fit, API and auth correctness, domain invariants, operational safety,
performance, and useful tests.

For more detail, read `references/review-profile.md` when asked to justify or
tune the style.

## Review Workflow

1. Establish intent before inspecting lines. Read the PR title/body, linked
   ticket if available, changed files, and nearby code. Identify the user-facing
   behavior, data model, and operational path the change touches.
2. Compare against existing patterns. Look for framework-native helpers, local
   managers/factories, generated clients, existing Celery tasks, existing retry
   decorators, established feature-flag mechanisms, and app ownership
   boundaries.
3. Review high-risk surfaces first: auth/session/token behavior, permissions,
   API contracts, request-path latency, data writes, migrations/backfills, query
   scale, async or slow external calls, and domain state transitions.
4. Then review shape and maintainability: duplicated methods, over-generic
   abstractions, unrelated PR scope, dead code, vague types, misplaced modules,
   and tests that are too broad or not useful.
5. Check validation. Prefer focused regression tests that execute the changed
   code path, mock at external API boundaries instead of patching internal
   methods, use existing factories over hand-built fixtures, require
   CI/dev-environment verification for integration-sensitive work, and remove
   redundant tests.
6. Write comments as concise questions or direct requests. Use `nit:` for minor
   issues. When suggesting a different design, explain the operational or domain
   reason, not just taste.
7. Separate blocking concerns from optional cleanup. Block or request follow-up
   for security, API compatibility, data integrity, slow synchronous work,
   unclear product intent, or broken tests. Do not block on small naming or
   style nits.

## Checklist

Prioritize these checks:

1. Scope control: keep PRs focused, rebase against current main when needed,
   remove unrelated changes, and avoid piling onto legacy patterns when a better
   endpoint or boundary is needed.
2. Existing patterns before new code: reuse framework/library support and local
   helpers instead of adding boilerplate, duplicate methods, custom throttles,
   custom retry logic, or one-off clients.
3. API contracts and compatibility: check route ownership, response shape,
   serializer/type specificity, generated client usage, mobile/web differences,
   and legacy fallback behavior.
4. Auth and security: inspect cookies vs tokens, refresh/logout flows, passkeys,
   OAuth boundaries, secret/config handling, signed URLs, rate limiting, staff
   access, impersonation, and audit logging.
5. Domain invariants and edge cases: verify behavior against real product rules,
   especially historical records and legacy data.
6. Data integrity: look for duplicate rows, unique constraints,
   migration-created data, backfill safety, idempotency, destructive
   infrastructure changes, and whether logs or existing records already provide
   evidence.
7. Operational safety: push slow LLM/API work out of synchronous request paths,
   prefer Celery or async views where appropriate, cap expensive inputs, use
   existing retry decorators, and question unnecessary feature flags or tasks.
8. Query and scale behavior: check indexes on large tables, pagination for broad
   result sets, `bulk_update` for large writes, upstream prefetching, avoiding
   extra downstream queries, file parallelism, and external API rate limits.
9. Test quality: ask for targeted coverage when behavior is new or risky, but
   remove overkill or duplicative tests. Prefer tests that exercise the real app
   path and mock only external API boundaries; use factories and framework test
   helpers over manual setup.
10. Type and responsibility clarity: ask for more specific types/enums, move
    code into the owning app/module, and keep views/controllers thin by pushing
    domain work into the right service.

## Backend Performance Heuristics

Apply these checks aggressively for backend PRs:

1. Request-path latency

   Identify all blocking work done before the HTTP response: external API calls,
   LLM calls, file reads, large serialization, loops over database rows, and
   cross-service retries. Ask whether the endpoint should dispatch a Celery task
   and let the client poll, use an async view, or return an accepted/job status
   instead of holding an API worker.

2. Query shape

   Look for N+1 queries, downstream queries inside service methods, extra
   queries that could be folded into an upfront load, and filters that will scan
   large tables. Prefer querying the data needed upfront,
   `select_related`/`prefetch_related` where appropriate, narrow filters, and
   in-memory calculation only after the bounded dataset is loaded.

3. Backfills and management commands

   Before accepting a backfill or command, ask:

   - How many rows will this touch in production?
   - Is the filter indexed for the table size?
   - Is the command batched, resumable, and safe to rerun?
   - Does it need a dry-run mode or production count query before execution?
   - Can it use `bulk_update`, `bulk_create`, or a single queryset update instead
     of per-row `save()`?
   - Is a backfill worth the effort, or can legacy records safely fall back?

4. External rate limits and retries

   Check whether the change uses the repo's existing retry/rate-limit client
   instead of adding a new retry loop or throttle. Validate concurrency limits
   and rate limits against the provider. If the framework already rate-limits a
   sensitive auth flow, question custom throttling.

5. Test suite performance

   Notice changes that slow CI: removed file parallelism, overbroad integration
   tests, duplicated assertions, or unnecessary database work. Prefer focused
   regression coverage at the right boundary.

## Test Behavior Heuristics

Use this section when reviewing tests, especially backend tests around APIs,
Celery tasks, SDKs, and external providers.

1. Mock external APIs, not the code under test

   Prefer tests that execute the real view/service/task/serializer path and mock
   the remote provider boundary with the repo's HTTP mocking tools, provider API
   fake, `responses`, request mocking, or existing provider API mock patterns.
   Do not patch the whole method, service, Celery task, or SDK call if the
   purpose of the test is to verify how the app uses that layer.

2. Keep the application integration path intact

   API tests should usually hit the real view, serializer validation,
   permissions, service dispatch, and response shape. Celery-related tests should
   remember that tasks often run synchronously in the test environment unless
   the codebase says otherwise, so patching task dispatch often removes useful
   coverage.

3. Use the right test layer for the risk

   Use unit tests for pure transformations and narrow branch logic. Use API
   tests for route, permission, serializer, and response contracts. Use
   integration tests for app/provider boundaries. Use E2E tests for user-visible
   flows, and do not replace E2E backend calls with frontend mocks when the
   point is to validate the real system.

4. Avoid over-mocked or duplicative tests

   If a test is heavily patched, duplicates an API test, or only asserts wiring
   that another test already covers, ask to remove it or fold the assertion into
   existing coverage. More tests are not better if they do not increase trust.

5. Prefer local test conventions

   Use existing factories instead of manual object setup, framework base test
   classes where consistent, and test module paths that mirror the owning
   app/module. Remove unnecessary database markers when the chosen base class
   already provides the database behavior.

6. Require real-flow coverage for risky changes

   Ask for registration-flow, OAuth login, permission, generated client/schema,
   or production-shaped dev-environment verification when the change is risky.
   The key question is whether the test would fail if the real integration
   broke.

## Backend Module Design Heuristics

Review module placement as a product and maintainability concern, not just
organization:

1. Put code in the owning app

   New domain behavior should usually live in the app that owns the concept.
   Treat catch-all legacy areas as a smell when the code is clearly
   domain-owned.

2. Keep views thin

   Views/controllers should handle request parsing, permissions, serializer
   validation, and response shape. Domain work belongs in services, model
   managers, or the owning module. If a view is assembling domain payloads,
   choosing records, or formatting service internals, ask whether the
   service/module should own that behavior.

3. Prefer cohesive services over pass-through flags

   Passing `skip_*`, `include_*`, or mode flags through many methods is a design
   smell. Ask whether the operation can be split, whether data can be appended at
   the end, or whether the service interface should accept a query object/status
   filter with clearer semantics.

4. Choose legacy compatibility deliberately

   Do not add more code to legacy paths by default. Either preserve legacy
   behavior through a clear fallback for old records/clients, or create a new
   cleaner path with explicit migration/rollout semantics. Question PRs that are
   half-generic and half-client-specific.

5. Use local naming conventions

   Names should reveal domain intent and API semantics. Avoid `include_*` when
   the parameter is actually exclusive; use status filters when callers need
   permutations; move tests to the owning app with a parallel module path; add
   docstrings only when they prevent future readers from needing ticket
   archaeology.

## Comment Style

Prefer short comments in this shape:

- `Can we ... ?`
- `Do we need ... ?`
- `Why are we ... ?`
- `This seems like ...`
- `nit: ...`
- `I would ... because ...`

Use direct requests when the fix is obvious:

```text
nit: can this type be more specific? Can we use the choices from above?
```

Use questions when intent, ownership, or tradeoffs need confirmation:

```text
Do we really need a separate query for this? Should we just query once and calculate in memory?
```

Use rationale when pushing back on implementation direction:

```text
We should not do this synchronously. These calls can take a long time and will hold up API workers.
```

Avoid generic "looks good" reviews when there are meaningful risks. If
approving, still mention any important rationale or residual risk briefly.

## Output Format

When returning a review:

- Lead with findings, ordered by severity.
- Include file and line when available.
- Keep each finding actionable and explain why it matters.
- Put nits after substantive issues.
- Mention tests or validation gaps at the end.
- If there are no issues, say that clearly and mention residual risk.
- When reviewing agent-authored PRs, check whether unresolved comments were
  answered and whether repeated mistakes should update coding guidance.
