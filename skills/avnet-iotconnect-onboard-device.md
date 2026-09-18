---
name: avnet-iotconnect-onboard-device
description: Authenticate to Avnet /IOTCONNECT, create a device template and a device, register it with the cloud broker and send it a command.
api: Avnet /IOTCONNECT Device API
generated: '2026-09-18'
method: generated
source: openapi/avnet-iotconnect-auth-openapi.yml, openapi/avnet-iotconnect-device-openapi.yml, conventions/avnet-conventions.yml
operations:
  - POST /api/v2/Auth/login
  - POST /api/v2/device-template
  - POST /api/v2/Device
  - PUT /api/v2/Device/{uniqueId}/acquire
  - PUT /api/v2/Device/{deviceGuid}/status
  - POST /api/v2/template-command/device/{deviceGuid}/send
  - GET /api/v2/Device/connection-status
  - PUT /api/v2/Device/{uniqueId}/release
note: The provider publishes no operationIds; operations are named by method + path exactly as they appear in the spec.
---

# Onboard a device on Avnet /IOTCONNECT

Hosts differ per module (`auth.iotconnect.io`, `device.iotconnect.io`); every path keeps its `/api/v2` prefix. Send `Content-Type: application/json` on every call.

1. **Log in** — `POST https://auth.iotconnect.io/api/v2/Auth/login` with the account credentials in the body and the tenant's `solution-key` header. Keep the returned JWT and send it as `Authorization: Bearer <token>` on everything below. The first-party client treats it as valid for 24 h; use `POST /api/v2/Auth/refresh-token` before it expires.
2. **Create (or reuse) a device template** — `POST https://device.iotconnect.io/api/v2/device-template` (or `/api/v2/device-template/quick`). A 409 with "name already exists" means it is already there: list with `GET /api/v2/device-template/lookup`-style lookups instead of retrying the POST — there is no idempotency key (conventions: `idempotency.coverage: none`).
3. **Add the device** — `POST /api/v2/Device` with the template GUID, the entity GUID it belongs to and a `uniqueId`. Expect 412 with an `error[]` list on validation failure.
4. **Register it with the cloud broker** — `PUT /api/v2/Device/{uniqueId}/acquire`. This is the step that creates the IoT-hub identity; it is reversible with `PUT /api/v2/Device/{uniqueId}/release` (no window is published).
5. **Activate** — `PUT /api/v2/Device/{deviceGuid}/status` with `isActive: true`; the same call with `false` is the undo.
6. **Verify** — `GET /api/v2/Device/connection-status` (or `/connection-status/bulk`) once the device SDK connects.
7. **Send a command** — `POST /api/v2/template-command/device/{deviceGuid}/send`; commands are defined on the template.

Error handling: 401 = re-login; 403 = missing role; 409 = conflict described per operation; 412 = precondition/validation with `error[{param,message}]`; 204 on lists means empty, not failure. See `errors/avnet-problem-types.yml`.
