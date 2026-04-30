# OpenAPI HTTP API Tools — Authoring Tip

Writing OpenAPI specs for public HTTP/REST APIs and uploading them as plugin tools via "Add tool → Upload specification" in Copilot Studio. Covers spec authoring rules, the upload parameter gotcha, and common runtime errors.

> **Related skills:** Use `add-action` for the upload UI walkthrough, `edit-action` to modify action YAML after cloning, and `clone-agent` to get files locally. This tip covers what those skills don't: how to write specs that survive the upload and runtime pipeline.

## The Problem

OpenAPI specs that work fine with Swagger UI or Postman can fail silently in Copilot Studio — at upload time, at runtime, or both. The failures are non-obvious because they stem from platform-specific constraints (PowerFx type system, XML default responses, parameter registration).

## Test the API Before Writing a Spec

Call each endpoint manually first:

```powershell
Invoke-RestMethod "https://api.example.com/data?param=value&format=json"
```

| Check | Why |
|-------|-----|
| **JSON output** | Many institutional APIs (Eurostat, ECB, OECD) default to XML. If the API can return XML, add a `format` query param with a JSON default value in the spec. Without it, Copilot Studio gets XML and throws `JsonReaderException '<' is invalid`. |
| **Response shape** | If the top-level JSON response is an array `[...]` instead of an object `{...}`, PowerFx crashes with *"Expecting Record but received a Table"*. The spec must declare `type: object` regardless of actual shape. |
| **URL pattern** | SDMX-style APIs enforce strict dimension order in path params (e.g., `freq.unit.coicop.geo`). Wrong order returns 400 or empty results. Test with exact dimension order. |
| **Required params** | Which query params are mandatory? Which have API-specific defaults you shouldn't rely on? |

## Spec Rules for Copilot Studio

### Minimum working spec

```yaml
openapi: "3.0.0"
info:
  title: My API
  version: "1.0"
servers:
  - url: https://api.example.com
paths:
  /data/{param}:
    get:
      operationId: getData
      summary: Short summary
      parameters:
        - name: param
          in: path
          required: true
          schema:
            type: string
          description: Brief description
        - name: format
          in: query
          required: true
          schema:
            type: string
            default: JSON
          description: Response format
      responses:
        "200":
          description: Success
          content:
            application/json:
              schema:
                type: object
```

### Key rules

1. **Response schema: always `type: object`** — even if the API actually returns an array. The orchestrator still parses correctly; PowerFx won't crash.

2. **Force JSON with defaults** — if the API can return XML:
   ```yaml
   - name: format
     in: query
     required: true
     schema:
       type: string
       default: JSON
   ```

3. **Keep descriptions short at upload time** — long descriptions or special characters cause *"Something went wrong"* on save. Enrich descriptions later via `modelDescription` in the cloned action file (use `edit-action`).

4. **Use `default` for fixed params** — params that should always have the same value (e.g., `per_page: 500`).

5. **Response schema can be minimal** — a bare `type: object` is sufficient. You don't need to describe every field.

6. **Omit `security`/`securitySchemes` for no-auth APIs** — set auth to "None" during upload instead.

## Upload Parameter Gotcha

When uploading via "Add tool → Upload specification", in the **"Select tool parameters"** step:

- Parameters are **NOT pre-selected**
- You must click **"Add input"** for EVERY available parameter (path params + query params)
- Only explicitly added params become `AutomaticTaskInput` in the generated action YAML
- If you skip this, the orchestrator cannot pass values to those parameters at runtime

This is the single most common cause of "my tool ignores the parameters I pass." Re-upload and add all inputs.

## Common Errors

| Symptom | Cause | Fix |
|---------|-------|-----|
| `JsonReaderException '<' is invalid` | API returned XML, not JSON | Add `format` query param with JSON default in spec |
| `Expecting Record but received Table` | Response is a top-level JSON array | Change response schema to `type: object` |
| "Something went wrong" on save | Long descriptions, special characters, or complex schema | Shorten descriptions, simplify schema |
| Tool ignores parameter values | Didn't click "Add input" during upload | Re-upload, click "Add input" for all params |
| 404 at runtime | Path params don't match actual API URL | Test URL manually, fix path in spec |
| Tool missing from `actions/` after pull | Pull can't create new files | Use clone instead of pull (see `clone-agent`) |
