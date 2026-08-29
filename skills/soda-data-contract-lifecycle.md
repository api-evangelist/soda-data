---
name: soda-contract-lifecycle
description: Generate, publish, verify and roll back a Soda data contract — the declarative source of truth for a dataset's checks.
api: Soda Cloud API v4
base_url: https://cloud.soda.io
operations:
  - POST/api/v1/contracts/actions/createSkeleton
  - GET/api/v1/contracts/actions/createSkeleton/{operationId}
  - POST/api/v1/contracts/actions/generate
  - GET/api/v1/contracts/actions/generate/{operationId}
  - POST/api/v1/contracts
  - GET/api/v1/contracts
  - GET/api/v1/contracts/{contractId}
  - POST/api/v1/contracts/{contractId}
  - GET/api/v1/contracts/{contractId}/versions
  - POST/api/v1/contracts/{contractId}/verify
  - GET/api/v1/scans/{scanId}
  - GET/api/v1/scans/{scanId}/logs
generated: '2026-08-29'
method: generated
source: openapi/_original/soda-data-cloud-api-v4-openapi.yml
---

# Author and verify a Soda data contract

A data contract is the declarative source of truth for one dataset's checks. **Checks are not created
through the Checks API** — that surface is list and delete only. To add a check you edit contract YAML
and publish it.

## Bootstrap the YAML

Two paths, both async:

- **Skeleton** — `POST/api/v1/contracts/actions/createSkeleton` derives a contract body from the
  warehouse schema. Poll `GET/api/v1/contracts/actions/createSkeleton/{operationId}`.
- **Copilot / AI generation** — `POST/api/v1/contracts/actions/generate` for one or more datasets.
  Poll `GET/api/v1/contracts/actions/generate/{operationId}`. Both submit operations are limited to
  10 requests / 60 seconds; both poll operations to 60 / 60.

## Publish

1. `POST/api/v1/contracts` creates the contract on a dataset with an initial body. **Not
   idempotent** — on a timeout, list with `GET/api/v1/contracts` before retrying.
2. `POST/api/v1/contracts/{contractId}` publishes new YAML content. This one *is* safe to retry: it
   is addressed by contract id and overwrites rather than appends.

## Verify

3. `POST/api/v1/contracts/{contractId}/verify` triggers a verification scan (10 requests / 60 seconds).
4. Follow the scan with `GET/api/v1/scans/{scanId}` and `GET/api/v1/scans/{scanId}/logs`.

## Roll back

Every publish creates a version. This is the **only reversal path in the API** with real coverage:

5. `GET/api/v1/contracts/{contractId}/versions` returns the publish history.
6. Re-publish the prior body with `POST/api/v1/contracts/{contractId}`.

Soda does **not** state a retention window on contract versions. Do not promise a caller that an old
version will still be there — read the version list and check before you commit to a rollback plan.

## Local alternative

`sodacli contract lint <file>` validates syntax offline and `sodacli contract diff <file>` shows a
local-vs-cloud diff, both without touching Soda Cloud. Prefer them before publishing.
