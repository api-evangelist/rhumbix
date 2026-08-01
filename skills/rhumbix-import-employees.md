---
name: Import employees into Rhumbix
description: Batch-import a company's employees into Rhumbix and confirm the import completed.
api: openapi/rhumbix-public-api-openapi-original.json
operations: [batchEmployeeImport, batchEmployeeImportStatus]
---

# Import employees into Rhumbix

Use this to load or update a company's field workforce in Rhumbix from an ERP/HR source.

## Auth
- Send `x-api-key: <your key>` on every request (header). Keys are scoped to a company.
- Every path is scoped by `{company_key}` — use the company you were provisioned for.
- Base host: `https://async-api.rhumbix.com`.

## Steps
1. **Submit the batch** — `POST /{company_key}/batch/employee/import` (`batchEmployeeImport`) with a JSON body of `Employee` records (`first_name`, `last_name`, `company_supplied_id`, `classification`, `trade`, `email`, `project_job_numbers`, ...). The response returns a `BatchKey` (`batch_key`).
2. **Poll the status** — `GET /{company_key}/batch/employee/import/{batch_key}/status` (`batchEmployeeImportStatus`) using the `batch_key`. Read the `ImportStatus` envelope (`status`, `detail`).
3. **Handle the outcome** — repeat step 2 until the batch reports completion. A `403` means the `x-api-key` is missing/invalid or not authorized for this `company_key`.

## Rules
- `company_supplied_id` is the employee natural key — keep it stable to update rather than duplicate.
- No idempotency key is documented; do not blindly re-submit a batch on a network error — poll status first (see conventions/rhumbix-conventions.yml).
- Errors surface on the `ImportStatus` envelope, not as RFC 9457 problem+json (see errors/rhumbix-problem-types.yml).
