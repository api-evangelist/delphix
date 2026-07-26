---
name: Mask sensitive data with a Delphix Continuous Compliance job
description: >-
  Select a masking connector and masking job, execute it to de-identify sensitive
  data, and confirm results — using the Delphix DCT Continuous Compliance API.
api: openapi/delphix-continuous-compliance-openapi.yml
operations:
  - get_connectors
  - test_connector
  - get_masking_jobs
  - get_masking_job_by_id
  - execute_masking_job
  - search_masking_jobs
---

# Mask sensitive data with a Delphix Continuous Compliance job

Use the Data Control Tower Continuous Compliance API (`/dct/v3`) to run automated
data masking so non-production environments carry protected, compliant data.

## Authentication

Send an API key in the `Authorization` header prefixed with `apk` (see
`authentication/delphix-authentication.yml`).

## Steps

1. **List connectors** — `get_connectors` (`GET /connectors`) to find the connector
   that points at the database/environment you want to mask. Use `search_connectors`
   for filtered lookups.
2. **Validate connectivity** — `test_connector`
   (`POST /connectors/{connectorId}/test`) to confirm the connector can reach its
   target before running a job.
3. **Find the masking job** — `get_masking_jobs` (`GET /masking-jobs`) or
   `search_masking_jobs` (`POST /masking-jobs/search`) to locate the `maskingJobId`.
   Inspect it with `get_masking_job_by_id` (`GET /masking-jobs/{maskingJobId}`).
4. **Execute the job** — `execute_masking_job`
   (`POST /masking-jobs/{maskingJobId}/execute`). This is asynchronous and returns a
   `job` with a `jobId` (HTTP 202).
5. **Wait for completion** — poll the Jobs API until the job reaches a terminal
   state; the masking run reports success/failure and row metrics.

## Error handling

Failures use the `Error`/`ErrorResponse` JSON envelopes (see
`errors/delphix-problem-types.yml`). Validate the connector first to avoid a masking
job failing mid-run against an unreachable target.
