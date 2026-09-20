---
generated: '2026-09-19'
method: generated
name: Create a schema and extract a document
description: Define the output shape once as a schema, then convert a PDF, spreadsheet, document, image or raw text into JSON that matches it.
api: openapi/claix-dev-openapi.yml
operations: [listSchemas, createSchema, pdfToJson, excelToJson, docToJson, imgToJson, txtToJson]
source: >-
  Grounded in the MCP prompt workflow.discover-and-extract and the docs at
  https://www.claix.dev/documentation/schemas/create-schema; operationIds verified in
  openapi/claix-dev-openapi.yml.
---

# Create a schema and extract a document

Claix extracts against a **schema** you own; every extraction endpoint takes a `schema_id` whose
`type` must match the endpoint (`pdf-json` for `/pdf-json`, and so on).

## Auth
- `x-api-key: <key>` (or `Authorization: Bearer <key>`); the key scopes everything to one workspace. See `authentication/claix-dev-authentication.yml`.

## Steps
1. **Reuse before you create** — `listSchemas` (`GET /api/schemas`) returns every schema on the account with `id`, `name`, `type`, `schema_definition`. Pick one whose `type` matches your input.
2. **Create one if needed** — `createSchema` (`POST /api/create-schema`, JSON): `name` (≤200 chars), `type` ∈ `excel-json | json-excel | pdf-json | doc-json | img-json | txt-json`, `schema_definition` = `{ field: { type: string|integer|number|boolean, description } }` (`description` is required). Expect **201** and keep `schema.id`.
3. **Extract** — send `multipart/form-data` with exactly `file` and `schema_id` (plus optional `space_id`) to the matching operation:
   - `pdfToJson` — `POST /api/pdf-json` (≤15 MB, text or scanned)
   - `excelToJson` — `POST /api/excel-json` (.xlsx/.csv, first sheet only)
   - `docToJson` — `POST /api/doc-json` (.docx/.txt/.md/.rtf, ≤10 MB)
   - `imgToJson` — `POST /api/img-json` (.jpeg/.png/.webp/.heic, ≤15 MB)
   - `txtToJson` — `POST /api/txt-json` (JSON body `content` + `schema_id`, ≤300,000 chars; no file)
4. **Read the result** — only **200** is success. `data` holds the records keyed by your schema fields; Excel responses also carry `mapa_columnas` (source column → field) and `total_filas_procesadas`. If the schema has `window_context` enabled the response also carries a `document_id` (see the persisted-document skill).

## Idempotency
- There is none (`conventions/claix-dev-conventions.yml`, `idempotency.coverage: none`). A retried `createSchema` makes a duplicate; a retried extraction is billed again. Retry only on 5xx/502.

## Errors
- **400** wrong field names / wrong schema `type` for the endpoint; **404** `schema_id` not on this account; **413** over the size limit; **422** nothing in the file matched the schema (blurry image, no matching columns); **502** AI provider failure — transient, back off and retry. Envelope `{error, detalle}` — see `errors/claix-dev-problem-types.yml`.

## Cost
- First 100 successful calls free, then €0.15 (PDF/Excel/Doc) or €0.10 (Image/Text) per successful call; schema operations are free. Failed calls are not billed. See `plans/claix-dev-plans-pricing.yml`.
