---
name: soda-onboard-datasource
description: Connect a warehouse to Soda Cloud and onboard its tables as Soda datasets, using the Soda Cloud v4 REST API.
api: Soda Cloud API v4
base_url: https://cloud.soda.io
operations:
  - POST/api/v1/datasources/actions/testConnection
  - GET/api/v1/datasources/actions/testConnection/{operationId}
  - POST/api/v1/datasources
  - POST/api/v1/datasources/{datasourceId}/discover
  - GET/api/v1/discoveredDatasets
  - POST/api/v1/datasources/{datasourceId}/onboardDatasets
  - GET/api/v1/datasources/{datasourceId}/onboardDatasets/{operationId}
  - GET/api/v1/datasets
generated: '2026-08-29'
method: generated
source: openapi/_original/soda-data-cloud-api-v4-openapi.yml
---

# Onboard a data source into Soda Cloud

Every Soda dataset lives under a datasource. Nothing else in the platform — contracts, checks,
monitors, incidents — can exist until this flow has run once.

## Before you start

- Authenticate with HTTP Basic: `Authorization: Basic base64(api_key_id:api_key_secret)`.
- Pick the right region host. `https://cloud.soda.io` (EU) and `https://cloud.us.soda.io` (US) are
  separate tenancies; a key from one does not work on the other.
- Confirm the credentials first with `GET/api/v1/test-login`. It is the tightest-limited operation in
  the API — 10 requests / 10 seconds — so call it once, not in a loop.
- Secrets belong in Soda, not in the config you post. Create them with `POST/api/v1/secrets` and
  reference them from the datasource YAML as `${secret.NAME}`.

## Steps

1. **Rehearse the connection.** `POST/api/v1/datasources/actions/testConnection` with the datasource
   YAML and a runner. It returns 202 and an operation id — this is a dry run, it creates nothing.
2. **Poll the test.** `GET/api/v1/datasources/actions/testConnection/{operationId}` until it reports a
   terminal state. Limit: 60 requests / 60 seconds, so poll on an interval, not a tight loop.
3. **Create the datasource.** `POST/api/v1/datasources` with the same YAML. Limit: 10 requests / 60
   seconds. **This create is not idempotent** — there is no Idempotency-Key on this API. If the call
   times out, do NOT blind-retry: call `GET/api/v1/datasources` and check whether it already exists.
4. **Discover tables.** `POST/api/v1/datasources/{datasourceId}/discover` triggers a discovery scan
   on demand instead of waiting for the scheduled cron. Limit: 10 requests / 60 seconds.
5. **List what was found.** `GET/api/v1/discoveredDatasets`. Discovered tables are not yet Soda
   datasets — they cannot carry contracts, checks or monitors until onboarded.
6. **Onboard the ones you want.** `POST/api/v1/datasources/{datasourceId}/onboardDatasets`, then poll
   `GET/api/v1/datasources/{datasourceId}/onboardDatasets/{operationId}`.
7. **Confirm.** `GET/api/v1/datasets?datasourceName=<name>`. Page with `page` and `size` (10–1000);
   the envelope carries `totalElements` and `totalPages`.

## Rules

- **Deletes are terminal.** `DELETE/api/v1/datasources/{datasourceId}` cascades to every dataset,
  check, scan and incident under it, and Soda documents it as permanent. Never call it to "clean up"
  a partial onboarding — reconcile with `GET/api/v1/datasources` instead.
- **Errors** are `{ "code": string, "message": string }` with an HTTP status. 403 means the key lacks
  a role and will not clear on retry; 429 means you hit the operation's bucket and there is **no
  Retry-After header**, so back off on your own schedule.
