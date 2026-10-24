---
name: openapi-contract-check
description: Use when a branch changes an OpenAPI spec (or code that generates one) and you need to classify each contract change as breaking, non-breaking, or ambiguous with consumer impact.
---

# OpenAPI Contract Check

Compare the OpenAPI contract on the branch against the base branch and produce a consumer-facing compatibility report.

## When to use / When not to use

Use this skill when:

- A PR touches `openapi.yaml` / `openapi.json` or the DTOs, controllers, or annotations that generate the spec.
- The user asks "is this API change backward compatible?" or "do we need a new API version?".

Do not use this skill when:

- There is no spec and the API is internal-only with no consumers; a plain code review is enough.
- The API is GraphQL, gRPC, or AsyncAPI; this skill covers OpenAPI/REST only.

## Inputs

- The spec file(s): search for `openapi.yaml`, `openapi.json`, `swagger.json`, `api-spec.*`, and check CI/codegen config for the real source of truth.
- The base revision: usually `git show origin/main:<path-to-spec>`. If the spec is generated from code, generate it for both revisions into temp files.
- Known consumers, if documented: mobile apps pinned to old versions, external partners, published SDKs.

## Procedure

1. Locate the spec (see Inputs). If several exist, check each one the branch changed.
2. Produce the two versions to compare: base from git (`git show <base>:<path> > base-openapi.yaml`) and branch from the working tree. If the spec is generated, run the repo's codegen on both revisions.
3. Diff the two specs. Preferred tool: `npx openapi-diff base-openapi.yaml branch-openapi.yaml`. Without it, walk paths, operations, parameters, request bodies, responses, schemas, and enums by hand.
4. Classify every difference using [change-classification-table.md](change-classification-table.md). The short version:
   - Breaking: removed path/operation/response field; new required request field; narrowed type; removed enum value; stricter validation; removed success status code; added or tightened auth.
   - Non-breaking: new path/operation; new optional request field; new response field; new success status code; description/example-only edits.
   - Ambiguous: added enum value (breaks strict deserializers and exhaustive switches); relaxed validation; type widening; changed defaults; renamed `operationId`.
5. For each breaking or ambiguous change, write the consumer impact: which call fails, whether at compile time (generated clients) or runtime (hand-written clients, mobile apps), and from which client version.
6. Recommend versioning:
   - Only non-breaking changes -> no version bump; say so explicitly.
   - Breaking changes -> new path version (`/v2/...`), a parallel operation, or media-type versioning; pick whichever scheme the repo already uses. If the repo has none, propose path versioning and explain why in one sentence.
7. Present the report, plus a migration note per breaking change.

## Output format

```markdown
## API contract report: <spec> (<base> -> <branch>)

### Breaking (N)
- `DELETE /v1/users/{id}` removed - generated SDKs <= 2.3 fail at compile time; hand-written clients get 404 at runtime.
  Migration: use `POST /v2/users/{id}/deactivate`.

### Non-breaking (N)
- `GET /v1/orders` gains optional query param `status`; old requests unchanged.

### Ambiguous (N)
- `OrderState` enum gains `REFUNDING` - safe for tolerant readers, breaks clients with exhaustive switch statements.

### Versioning recommendation
<none required | bump to /v2 | parallel operation | ...>
```

## Guardrails

- Compare spec to spec, not YAML text: reordered keys and rewrapped descriptions are not changes.
- Never call a change non-breaking without checking both directions: what consumers send and what they parse.
- If the consumer set is unknown, state that assumption at the top of the report.
- Workspace rules and the repository's existing versioning scheme take precedence over the defaults above.
- Do not edit the spec as part of the check unless the user asked for fixes.
