---
name: Export workshift/timecard data from Rhumbix
description: Request a batch export of workshift and timecard data for a time window, then fetch the results.
api: openapi/rhumbix-public-api-openapi-original.json
operations: [batchWorkshiftExport, batchWorkshiftExportStatus, batchWorkshiftExportResults]
---

# Export workshift/timecard data from Rhumbix

Use this to pull captured field time (workshifts, timecard entries, cost-coded hours) out of Rhumbix into payroll or an ERP.

## Auth
- Send `x-api-key: <your key>` on every request (header), scoped to a company.
- Every path is scoped by `{company_key}`.
- Base host: `https://async-api.rhumbix.com`.

## Steps
1. **Request the export** — `POST /{company_key}/batch/workshift/export` (`batchWorkshiftExport`) with a `WorkshiftQuery` body bounding the window (`start_time`, `end_time`). The response returns a `batch_key`.
2. **Poll until ready** — `GET /{company_key}/batch/workshift/export/{batch_key}/status` (`batchWorkshiftExportStatus`). A `202` means still processing; back off and re-poll. A `200` means the export is complete.
3. **Fetch the results** — `GET /{company_key}/batch/workshift/export/{batch_key}/results` (`batchWorkshiftExportResults`). Each `Workshift` carries a `foreman` (Employee) and `timecard_entries[]`; each `TimecardEntry` has an `employee` and `work_components[]` (cost-coded regular/overtime/double-time minutes).

## Rules
- This is an asynchronous submit-poll-fetch flow — always poll status before fetching results (see conventions/rhumbix-conventions.yml).
- Map `work_components[].cost_code` to your job-costing codes; `cost_code` belongs to a `Project` via `project_job_number` (see data-model/rhumbix-data-model.yml).
- A `403` on status means the `x-api-key` is not authorized for this `company_key`/`batch_key`.
