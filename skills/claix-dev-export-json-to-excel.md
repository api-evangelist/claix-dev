---
generated: '2026-09-19'
method: generated
name: Export JSON records to an Excel file
description: Turn one or many JSON documents into an .xlsx workbook whose columns follow a json-excel schema.
api: openapi/claix-dev-openapi.yml
operations: [createSchema, jsonToExcel]
source: >-
  Grounded in https://www.claix.dev/documentation/json-to-excel; operationIds and the four request
  schemas (JsonExcelMultipartRequest, JsonExcelArrayRequest, JsonExcelSingleObjectRequest,
  JsonExcelEnvelopeRequest) verified in openapi/claix-dev-openapi.yml.
---

# Export JSON records to an Excel file

## Auth
- `x-api-key`.

## Steps
1. **Create a `json-excel` schema** — `createSchema` with `type: json-excel` and a `schema_definition` naming the columns. Agent mode is not supported for this type.
2. **Convert** — `jsonToExcel` (`POST /api/json-excel`). The spec accepts the records as multipart (`file` + `schema_id`) or as JSON (an array, a single object, or an envelope) with `schema_id`.
3. **Save the binary** — a **200** returns `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` with a `Content-Disposition` header carrying the filename; it is the only binary response in the API. Over MCP (`claix.convert.json_to_excel`) the same file arrives as `file_base64`.

## Errors
- **400** JSON or `schema_id` invalid/missing; **422** no keys matched the schema; **502** AI failure (retry). See `errors/claix-dev-problem-types.yml`.

## Notes
- Pricing for this route is not in the public table (llms.txt says so); everything else about billing is in `plans/claix-dev-plans-pricing.yml`.
- Nothing is persisted; there is nothing to reverse.
