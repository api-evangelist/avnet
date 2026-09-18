---
name: avnet-iotconnect-ota-update
description: Publish firmware and schedule, monitor and if needed cancel an over-the-air update on Avnet /IOTCONNECT.
api: Avnet /IOTCONNECT Firmware API
generated: '2026-09-18'
method: generated
source: openapi/avnet-iotconnect-firmware-openapi.yml, openapi/avnet-iotconnect-event-openapi.yml, conventions/avnet-conventions.yml
operations:
  - POST /api/v2/Firmware
  - GET /api/v2/Firmware/lookup
  - GET /api/v2/firmware-upgrade/lookup/{firmwareGuid}
  - POST /api/v2/ota-update
  - GET /api/v2/ota-update
  - POST /api/v2/ota-update/device-update-history
  - PUT /api/v2/ota-update/cancel-scheduled-otaupdate
  - PUT /api/v2/Firmware/{firmwareGuid}/deprecate
note: Method + path naming; the provider publishes no operationIds.
---

# Run an OTA firmware update

All calls go to `https://firmware.iotconnect.io` with the JWT from the Authenticate API (`Authorization: Bearer <token>`).

1. **Register the firmware** — `POST /api/v2/Firmware` against a device template (upload limit raised to 1 GB in AWS release 1.1.0). Do not retry a POST blindly: there is no idempotency key, and a duplicate name returns 409.
2. **Pick the upgrade** — `GET /api/v2/Firmware/lookup` (filters: `isDraft`, `includeDeprecated`, `isEdge`, `isGateway`, `isLowBandwidth`, `isIotEdgeEnable`) and `GET /api/v2/firmware-upgrade/lookup/{firmwareGuid}` for the version GUID.
3. **Send the update** — `POST /api/v2/ota-update` with the firmware-upgrade GUID and the target devices. From Azure 1.5.1 the template can queue OTA until the device asks for it over MQTT (on-demand OTA).
4. **Monitor** — `GET /api/v2/ota-update` (sent list) and `POST /api/v2/ota-update/device-update-history`; optionally subscribe a webhook to the *OTA update acknowledgment* event via the Event API (`asyncapi/avnet-iotconnect-webhooks.yml`).
5. **Take it back** — `PUT /api/v2/ota-update/cancel-scheduled-otaupdate` cancels a scheduled update. No cancellation window is published (reversibility grade: documented, not verified). Retire a bad build with `PUT /api/v2/Firmware/{firmwareGuid}/deprecate`.
