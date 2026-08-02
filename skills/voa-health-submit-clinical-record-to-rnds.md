---
name: submit-clinical-record-to-rnds
description: Submit a finalized consultation's clinical record to Brazil's RNDS national health data network via the Voa integration API, then confirm acceptance.
api: Voa RNDS Integration API
base_url: https://integration.voa.health/v1
operations:
  - identify
  - submitRndsRecord
  - getRndsStatus
source: Grounded in openapi/voa-health-rnds-openapi.yml and openapi/voa-health-identify-openapi.yml (real operationIds).
---

# Submit a clinical record to RNDS

Send a finalized consultation to the RNDS national bus (FHIR R4) and confirm it was accepted.

## Prerequisites

- The establishment is RNDS-activated (active CNES, ICP-Brasil A1 certificate configured with Voa — see `lifecycle/voa-health-lifecycle.yml`).
- You hold a Voa Auth Token.
- The patient CPF and responsible practitioner CNS are known.

## Steps

1. **Get a Bearer token** (`identify`) — exchange your Auth Token for a per-consultation JWT:
   `POST https://api.voa.health/integration/identify/` with header `x-voa-token: {AUTH_TOKEN}` and body `{ consultation_id, doctor_id, patient_id, expiration }`. Read `token` from the response.
2. **Submit to RNDS** (`submitRndsRecord`) — `POST /ehrs/{ehr_id}/rnds/submit` with `Authorization: Bearer <token>` and body `{ document_type: "RAC", patient_cpf, practitioner_cns, cnes }`. Voa assembles the FHIR R4 Bundle automatically. Capture `rnds_record_id`.
3. **Confirm acceptance** (`getRndsStatus`) — `GET /ehrs/{ehr_id}/rnds/status`. Poll until `status` is `accepted` (or handle `rejected`).

## Rules

- **Async**: RNDS round trips add 2-5s latency — do not block the clinician UI; submit and poll asynchronously.
- **Rate limits**: 1000 req/hour and 10 req/second per token.
- **Errors** (`errors/voa-health-problem-types.yml`): on `400 validation_error` fix the missing field in `details[]`; on `422 rnds_rejected` read `details[].diagnostics` (e.g. patient not in national registry).
- **Idempotency**: none is provided — do not blindly retry a `submit`; check status first, and use `replaceRndsRecord` for corrections.
