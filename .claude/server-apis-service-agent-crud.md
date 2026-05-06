# Service Agent CRUD via HTTP

Service agents are normal Mix agents exposed as HTTP endpoints. They are the **only** way to read/write workspace DB data over HTTP — there is no direct DB query endpoint.

## STRICT RULE — In-Params

**Never call a service agent without being told its in-param contract.**

The agent server's OpenAPI spec documents only the generic `/run-agent` envelope (body schema = `{}`). There is no introspection endpoint. Two agents with the same name in different projects can have
completely different param shapes. Guessing causes silent wrong behavior (0 results, bad saves) with no obvious error.

If in-params are unknown → **ask the user. Do not probe, do not guess.**
The only exception: user explicitly says "try and figure it out."

## Calling an Agent

```
POST https://agt.remixlabs.com/v0/run-agent/{ws}/{app}/{agent}
Authorization: Bearer <JWT>
Content-Type: application/json

{ ...agent in-params... }
```

Also callable as `GET` with URL query params for some agents.

## OpenAPI Spec

The spec is at `https://agt.remixlabs.com/v1/_doc/openapi.json` (requires JWT auth).

- The Swagger UI at `/v1/swagger/` is JS-rendered — unreadable by automated fetch. Always fetch the JSON spec directly via curl.
- The spec URL is not guessable from the UI URL; find it via browser DevTools → Network tab while loading the Swagger UI.

## Wire Format Rules

### Ref / _rmx_id (critical)

All `Ref`-typed fields (`_rmx_id`, `_rmx_ids`, etc.) must be sent as a **tagged object**:

```json
{
  "_rmx_type": "{tag}case:db.ref",
  "_rmx_value": "<id string>"
}
```

Sending a bare string returns: `error in FFI execution: expected non-empty ref`

### Field values for save agents

Agents typed as `map(string)` or `array(map(string))` require **all field values to be strings**. Sending numbers causes `ERR_EXPECTED_STRING`. Cast everything to string before sending.

### Pagination

Some query agents silently return 0 results when pagination params (`first_n`, `skip_n`) are omitted — even when records exist. This is not an error; undefined `first_n` becomes `LIMIT 0`. Always
include pagination params when the agent contract requires them.

## Example Contracts (padmanabha-playground — project-specific, for illustration only)

These shapes are specific to `padmanabha-playground`. Other projects' agents with the same names will likely differ.

### cloud_query

```json
{
  "entity": "book",
  "first_n": 25,
  "skip_n": 0
}
```

Returns `{"response": [...records]}`. Missing `first_n`/`skip_n` → returns 0 results silently.

### cloud_save

```json
{
  "record": {
    "entity": "book",
    "title": "Clean Code",
    "year": "2008"
  }
}
```

Returns `{"_rmx_type": "{tag}case:ok", "_rmx_value": {"saved_object": {...}}}`.

### cloud_save_multiple

```json
{
  "records": [
    {
      "entity": "book",
      "title": "Clean Code",
      "year": "2008"
    },
    ...
  ]
}
```

Returns `{"_rmx_type": "{tag}case:ok", "_rmx_value": {"saved_objects": [...]}}`.
All field values must be strings.

### cloud_delete

```json
{
  "_rmx_id": {
    "_rmx_type": "{tag}case:db.ref",
    "_rmx_value": "<id>"
  }
}
```

Returns `{"_rmx_type": "{tag}case:ok", "_rmx_value": {"message": "Record deleted"}}`.

### cloud_delete_multiple

```json
{
  "_rmx_ids": [
    {
      "_rmx_type": "{tag}case:db.ref",
      "_rmx_value": "<id>"
    },
    ...
  ]
}
```

Returns `{"message": ""}` (silent ok — different shape from cloud_delete).

## Saved Record Metadata

Every saved record includes standard Remix fields:

- `_rmx_id` — tagged db.ref, the stable identifier
- `_rmx_rev` — tagged db.ref, the revision identifier
- `_rmx_row` — integer row number (global to the DB, not per-entity)
- `_rmx_content_hash` — content hash string
- `_rmx_created_at` / `_rmx_created_by`
- `_rmx_last_modified_at` / `_rmx_last_modified_by`