---
name: Provision a virtual database (VDB) with Delphix DCT
description: >-
  Find a dSource, pick a snapshot, and provision a fresh virtual database, then
  refresh or snapshot it — using the Delphix Data Control Tower Continuous Data API.
api: openapi/delphix-continuous-data-openapi.yml
operations:
  - get_dsources
  - get_snapshots
  - provision_vdb_by_snapshot
  - provision_vdb_by_timestamp
  - get_vdb_by_id
  - refresh_vdb_by_snapshot
  - snapshot_vdb
---

# Provision a virtual database (VDB) with Delphix DCT

Use the Data Control Tower Continuous Data API (`/dct/v3`) to deliver a fast,
space-efficient virtual copy of a source database.

## Authentication

Send every request with an API key in the `Authorization` header, prefixed with
`apk`:

```
Authorization: apk 1.XXXXXXXX...
```

See `authentication/delphix-authentication.yml` and `conventions/delphix-conventions.yml`.

## Steps

1. **List available dSources** — `get_dsources` (`GET /dsources`). Choose the
   `dsourceId` of the linked production source you want to virtualize. Results are
   cursor-paginated (`cursor` + `limit`, items under `items`).
2. **Pick a point in time** — `get_snapshots` (`GET /snapshots`) filtered to the
   dSource, or provision directly against a timestamp.
3. **Provision the VDB** — either:
   - `provision_vdb_by_snapshot` (`POST /vdbs/provision_by_snapshot`) with the
     chosen `snapshot_id`, target `engine_id`/`environment_id`, and VDB name; or
   - `provision_vdb_by_timestamp` (`POST /vdbs/provision_by_timestamp`) to provision
     from a wall-clock time.
   These are asynchronous — the response returns a `job` with a `jobId` (HTTP 202).
4. **Wait for the job** — poll the Jobs API until the job reaches a terminal state.
5. **Confirm the VDB** — `get_vdb_by_id` (`GET /vdbs/{vdbId}`) to read status,
   connection details, and current timeflow.
6. **Refresh or snapshot as needed** — `refresh_vdb_by_snapshot`
   (`POST /vdbs/{vdbId}/refresh_by_snapshot`) to reset the VDB to newer data, or
   `snapshot_vdb` (`POST /vdbs/{vdbId}/snapshots`) to capture a new point in time.

## Error handling

Failures return the `Error`/`ErrorResponse` JSON envelopes (see
`errors/delphix-problem-types.yml`). A `404` means the referenced dSource, snapshot,
VDB, or engine does not exist; `401` means the API key/`apk` prefix is missing or
invalid.
