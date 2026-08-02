---
name: correct-or-remove-rnds-record
description: Replace a previously submitted RNDS clinical record after a correction, or remove it from the national bus when required.
api: Voa RNDS Integration API
base_url: https://integration.voa.health/v1
operations:
  - getRndsStatus
  - replaceRndsRecord
  - deleteRndsRecord
source: Grounded in openapi/voa-health-rnds-openapi.yml (real operationIds).
---

# Correct or remove an RNDS record

Fix a clinical record already on the RNDS bus, or delete it when legally/clinically necessary.

## Steps

1. **Verify a prior submission exists** (`getRndsStatus`) — `GET /ehrs/{ehr_id}/rnds/status` with `Authorization: Bearer <token>`. If `404 not_found`, there is nothing to replace or delete.
2. **Replace after a correction** (`replaceRndsRecord`) — `PUT /ehrs/{ehr_id}/rnds/replace` with body `{ practitioner_cns }`. The response links the new `rnds_record_id` to `previous_record_id`.
3. **Delete when required** (`deleteRndsRecord`) — `DELETE /ehrs/{ehr_id}/rnds` with body `{ practitioner_cns, reason }`. `reason` is mandatory.

## Rules

- **Deletion is irreversible** on the RNDS national bus — confirm intent before calling `deleteRndsRecord`.
- **Prefer replace over delete** for corrections; delete is for records that should not exist.
- **Errors** (`errors/voa-health-problem-types.yml`): `400 no_previous_submission` means no record exists yet — submit first.
- Bearer JWT required (see `authentication/voa-health-authentication.yml`); 1000 req/hour, 10 req/second per token.
