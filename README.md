# OpenAPI Contract Check

A Camillia skill that compares OpenAPI contracts between base and branch, classifies every change as breaking, non-breaking, or ambiguous, explains consumer impact, and suggests a versioning strategy.

## What the agent gains

- A repeatable spec-to-spec comparison (not a text diff of YAML)
- A classification table covering paths, requests, responses, enums, and auth
- Consumer impact written per change: compile-time vs runtime, and from which client version
- Concrete versioning recommendations that follow the repo's existing scheme

## Install

Install from the Camillia Community Hub. After installation the skill lives in the workspace under `.workspace/extensions/openapi-contract-check/` and the agent applies it whenever a branch touches an API contract.

## Contents

- `extensions/openapi-contract-check/SKILL.md` - the procedure the agent follows
- `extensions/openapi-contract-check/change-classification-table.md` - breaking / non-breaking / ambiguous decision tables

## Validate

```
node scripts/validate.mjs
```

## License

MIT - see [LICENSE](LICENSE). Copyright 2026 Camillia contributors.
