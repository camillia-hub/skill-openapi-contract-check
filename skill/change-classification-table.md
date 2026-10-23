# OpenAPI change classification table

Check both directions of the contract: what consumers send (requests) and what they parse (responses).

## Paths and operations

| Change | Class | Why |
|--------|-------|-----|
| Path or operation removed | Breaking | Existing calls return 404 |
| Path or operation added | Non-breaking | Old clients are unaffected |
| `operationId` renamed | Ambiguous | Breaks generated clients that use it as the method name |
| Success status code removed (200 -> 204) | Breaking | Clients branch on the status code |

## Requests

| Change | Class | Why |
|--------|-------|-----|
| New optional field or parameter | Non-breaking | Old payloads stay valid |
| New required field or parameter | Breaking | Old payloads get rejected |
| Field made required | Breaking | Old payloads get rejected |
| Field made optional | Non-breaking | Looser input is safe for senders |
| Type narrowed (string -> int, enum subset) | Breaking | Old payloads get rejected |
| Validation made stricter (`minLength` up, `pattern` added) | Breaking | Old payloads get rejected |
| Validation relaxed | Ambiguous | Clients built on the old docs may rely on the rejection |
| Default value changed | Ambiguous | Silent behavior change for clients that omit the field |

## Responses

| Change | Class | Why |
|--------|-------|-----|
| Field added | Non-breaking | Tolerant readers skip unknown fields |
| Field removed | Breaking | Deserializers fail or silently return nulls |
| Type changed | Breaking | Parse failures |
| Enum value added | Ambiguous | Breaks exhaustive switches and strict deserializers; safe for tolerant readers. Ask how consumers deserialize |
| Enum value removed | Breaking | Old payloads become unparseable |
| Field made nullable | Breaking for typed clients | Nulls appear where none were possible |

## Cross-cutting

| Change | Class | Why |
|--------|-------|-----|
| Auth requirement added or scope tightened | Breaking | Old tokens get rejected |
| Auth requirement removed | Ambiguous | A security posture change; flag for review even though clients keep working |
| Descriptions, examples, tags edited | Non-breaking | Documentation only |
| `servers` URL changed | Breaking | Clients resolve endpoints from it |

When a row says Ambiguous, the report must state which consumer behavior decides it, and ask the user when that behavior is unknown.
