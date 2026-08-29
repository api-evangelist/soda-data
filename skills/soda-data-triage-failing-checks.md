---
name: soda-triage-failing-checks
description: Find failing data quality checks in Soda Cloud, trace them to their dataset and scan, and open or update an incident.
api: Soda Cloud API v4
base_url: https://cloud.soda.io
operations:
  - GET/api/v1/checks
  - GET/api/v1/datasets
  - GET/api/v1/datasets/{datasetId}
  - GET/api/v1/datasets/byDatasetQualifiedName/{datasetQualifiedName}
  - GET/api/v1/scans/{scanId}
  - GET/api/v1/scans/{scanId}/logs
  - GET/api/v1/incidents
  - POST/api/v1/incidents
  - POST/api/v1/incidents/{incidentId}
  - GET/api/v1/incidents/{incidentId}/rcaReport
  - POST/api/v1/incidents/{incidentId}/rcaReport
generated: '2026-08-29'
method: generated
source: openapi/_original/soda-data-cloud-api-v4-openapi.yml
---

# Triage failing checks and manage incidents

## Find the failure

1. `GET/api/v1/checks` lists checks with their associated datasets and linked incidents (60 / 60).
2. Resolve the dataset either by id (`GET/api/v1/datasets/{datasetId}`) or, when you only have a
   human-readable name, by its qualified name:
   `GET/api/v1/datasets/byDatasetQualifiedName/{datasetQualifiedName}` where the DQN is
   `[datasource]/[database]/[schema]/[dataset]`. This is the only natural key in the API — use it
   instead of paging `GET/api/v1/datasets` looking for a name.
3. Read the scan behind the result: `GET/api/v1/scans/{scanId}`, and `GET/api/v1/scans/{scanId}/logs`
   for the execution detail.

## Manage the incident

4. `GET/api/v1/incidents` to check whether one already exists before creating another — incident
   creation is **not idempotent** and a retried POST produces a duplicate.
5. `POST/api/v1/incidents` to open one (10 / 60), `POST/api/v1/incidents/{incidentId}` to update it.
6. Root-cause analysis lives at `GET/api/v1/incidents/{incidentId}/rcaReport` and
   `POST/api/v1/incidents/{incidentId}/rcaReport`.

## Incremental polling

`GET/api/v1/datasets` accepts `from` (ISO 8601) so you can pull only what changed since your last
sweep, plus `page`/`size` (10–1000) and `search`. Use `from` rather than re-walking every page — the
datasets list is the most rate-limited read in the API at 30 requests / 60 seconds.

## Cautions

- **Read scoping is silent.** Soda returns only the datasets the key holds View permission on; a
  missing dataset may be a permissions gap, not an absence.
- `DELETE/api/v1/checks/{checkId}` removes a check permanently, and the check will come back on the
  next publish if it is still defined in the contract YAML. Fix the contract, not the check.
