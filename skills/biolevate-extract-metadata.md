---
name: Extract typed metadata from documents
description: Run a Biolevate Extraction job to pull typed metadata fields (string/int/enum/date) from one or more indexed documents, then retrieve the structured results with explanations.
api: openapi/biolevate-api-original.json
operations: [createFile, listFiles, createExtractionJob, getExtractionJob, getExtractionJobOutputs, getExtractionJobAnnotations]
---

# Extract typed metadata from documents (Biolevate / Elise)

Auth: `Authorization: Bearer <pat>`; base URL is your Elise domain. Extraction is asynchronous (submit → poll → retrieve). See `conventions/biolevate-conventions.yml`.

## Steps
1. **Have indexed files.** Use `listFiles` (GET `/api/core/files`) to find existing EliseFile UUIDs, or `createFile` (POST `/api/core/files`) to index a provider item first.
2. **Create the extraction job.** `createExtractionJob` (POST `/api/core/extraction/jobs`) with `file_ids` and a `metas[]` list — each field has a `meta` name, a `description`, and an `answer_type` (e.g. `{dataType: STRING}`, `{dataType: INT}`, or `{dataType: ENUM, enumValues: [...]}`). Send an `Idempotency-Key` header to make submission retry-safe. Capture the `jobId`.
3. **Poll.** `getExtractionJob` (GET `/api/core/extraction/jobs/{jobId}`) until `status` is `SUCCESS`/`FAILED`.
4. **Retrieve.** `getExtractionJobOutputs` (GET `.../results`) → each result carries the `meta`, its extracted `answer`, and an `explanation`. Use `getExtractionJobAnnotations` (GET `.../annotations`) for source passages.

## Notes
For tabular/entity extraction across many rows, use the Multi-Dimensional Extraction flow instead (`createMDEJob` / `getMDEJob` / `getMDEJobOutputs`). Errors follow the same map as `errors/biolevate-problem-types.yml`.
